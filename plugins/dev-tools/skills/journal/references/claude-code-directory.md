# Claude Code Directory Data Structures

## Overview

The `~/.claude/` directory is Claude Code's primary data store for:
- User configuration and preferences
- Session transcripts and history
- File checkpoints and backups
- Skills, plugins, and custom commands

Understanding this structure enables extracting development activity for journaling purposes.

---

## Directory Structure

```
~/.claude/
├── CLAUDE.md                 # User-level memory (loaded every session)
├── settings.json             # Global settings (permissions, model, hooks)
├── history.jsonl             # Prompt history across all sessions
├── stats-cache.json          # Aggregated usage statistics
│
├── projects/                 # Session transcripts per project
├── plans/                    # Plan mode markdown documents
├── file-history/             # File checkpoints for undo/rollback
├── todos/                    # Task lists per session
├── session-env/              # Per-session environment variables
├── shell-snapshots/          # Shell environment snapshots
├── debug/                    # Debug logs per session
│
├── commands/                 # Custom slash commands (simple skills)
├── skills/                   # Complex skills with scripts
├── plugins/                  # Plugin marketplace and installations
│
├── ide/                      # IDE integration locks
├── statsig/                  # Feature flag cache
└── telemetry/                # Usage telemetry (if enabled)
```

Additionally, `~/.claude.json` (in home directory root, **not** inside `~/.claude/`) is a system-managed state file storing OAuth session, MCP servers, per-project state, and feature flag caches. It complements `~/.claude/settings.json` which handles user-managed policies. See [`~/.claude.json` section](#claudejson-root-level) for details.

---

## Core Data Files

### `history.jsonl`

**Purpose**: Chronological log of user prompts across all sessions.

**Format**: JSON Lines (one JSON object per line)

```json
{
  "display": "Help me implement user authentication",
  "pastedContents": {},
  "timestamp": 1759076614927,
  "project": "/Users/sam/Projects/my-app"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `display` | string | The user's prompt text |
| `pastedContents` | object | Any pasted content (files, images) |
| `timestamp` | number | Unix timestamp (milliseconds) |
| `project` | string | Absolute path to project directory |

**Dev Journal Use**: Quick activity timeline - shows what was worked on and when, without full conversation context.

---

### `stats-cache.json`

> See [Analytics Documentation](https://code.claude.com/docs/en/analytics)

**Purpose**: Aggregated usage metrics computed from session data.

**Structure**:
```json
{
  "version": 1,
  "lastComputedDate": "2025-12-28",
  "dailyActivity": [
    {
      "date": "2025-12-27",
      "messageCount": 1111,
      "sessionCount": 7,
      "toolCallCount": 330
    }
  ],
  "dailyModelTokens": [
    {
      "date": "2025-12-27",
      "tokensByModel": {
        "claude-opus-4-5-20251101": 143777
      }
    }
  ],
  "modelUsage": {
    "claude-opus-4-5-20251101": {
      "inputTokens": 607362,
      "outputTokens": 2272497,
      "cacheReadInputTokens": 875053258,
      "cacheCreationInputTokens": 83393179
    }
  },
  "totalSessions": 205,
  "totalMessages": 27163,
  "longestSession": {
    "sessionId": "b9904419-fe40-4b8b-ba2d-04864690f054",
    "duration": 251635105,
    "messageCount": 290
  },
  "firstSessionDate": "2025-11-12T17:10:45.841Z",
  "hourCounts": { "7": 23, "8": 31, "9": 26 }
}
```

**Dev Journal Use**: Activity summaries, productivity metrics, model usage patterns.

---

## Session-Based Directories

### Key Concept: Session UUID

A **Session UUID** (e.g., `31f3f224-f440-41ac-9244-b27ff054116d`) is the universal identifier that links data across multiple directories:

```
Session UUID: 31f3f224-f440-41ac-9244-b27ff054116d
     │
     ├──► projects/{project-path}/31f3f224-...jsonl   (conversation)
     ├──► file-history/31f3f224-.../                  (file backups)
     ├──► todos/31f3f224-...-agent-*.json            (task lists)
     ├──► session-env/31f3f224-.../                   (environment)
     └──► debug/31f3f224-....txt                      (debug logs)
```

---

### `projects/` - Session Transcripts

**Purpose**: Complete conversation transcripts organized by project.

**Structure**:
```
projects/
├── -Users-sam-Projects-dev-journal/     # Path encoded (/ → -)
│   ├── 31f3f224-f440-41ac-9244-b27ff054116d.jsonl  # Main session
│   ├── agent-a53626b.jsonl                          # Sub-agent session
│   └── ...
└── -Users-sam-Projects-my-app/
    └── ...
```

**File Format**: JSON Lines with multiple message types

#### Message Types

**User Message**:
```json
{
  "type": "user",
  "parentUuid": null,
  "isSidechain": false,
  "isMeta": false,
  "userType": "external",
  "cwd": "/Users/sam/Projects/dev-journal",
  "sessionId": "31f3f224-f440-41ac-9244-b27ff054116d",
  "version": "2.0.75",
  "gitBranch": "main",
  "message": {
    "role": "user",
    "content": "Help me implement the batch processor"
  },
  "uuid": "e2dbdfef-3699-4d96-8027-24a09d5cd58d",
  "timestamp": "2025-12-22T21:18:34.755Z",
  "thinkingMetadata": { "level": "high", "disabled": false },
  "todos": []
}
```

**Assistant Message**:
```json
{
  "type": "assistant",
  "message": {
    "role": "assistant",
    "content": "I'll help you implement..."
  },
  "uuid": "709290a1-7998-4237-a277-f30736678903",
  "parentUuid": "e2dbdfef-3699-4d96-8027-24a09d5cd58d",
  "toolUseMessages": [...],
  "timestamp": "2025-12-22T21:19:24.929Z"
}
```

**File History Snapshot**:
```json
{
  "type": "file-history-snapshot",
  "messageId": "709290a1-7998-4237-a277-f30736678903",
  "snapshot": {
    "messageId": "e2dbdfef-3699-4d96-8027-24a09d5cd58d",
    "trackedFileBackups": {
      "README.md": {
        "backupFileName": "59e0b9c43163e850@v1",
        "version": 1,
        "backupTime": "2025-12-22T21:19:24.929Z"
      }
    },
    "timestamp": "2025-12-22T21:18:34.761Z"
  },
  "isSnapshotUpdate": true
}
```

**Queue Operation** (prompt queuing):
```json
{
  "type": "queue-operation",
  "operation": "enqueue",
  "timestamp": "2025-12-31T19:06:54.212Z",
  "sessionId": "09abfbf5-851d-44f8-8a99-0efb3d91bfd6",
  "content": "user prompt text..."
}
```

| Field | Type | Description |
|-------|------|-------------|
| `operation` | string | `"enqueue"` or `"dequeue"` |
| `content` | string | The queued prompt text |
| `sessionId` | string | Session this queue operation belongs to |

#### Key Fields Reference

| Field | Description |
|-------|-------------|
| `type` | Message type: `user`, `assistant`, `file-history-snapshot`, `queue-operation` |
| `uuid` | Unique identifier for this message |
| `parentUuid` | Links response to its prompt (message chain) |
| `sessionId` | Session identifier (links to other directories) |
| `isSidechain` | Whether this is part of a side conversation branch (always `true` for sub-agents) |
| `isMeta` | Whether this is a meta/system message (e.g., command output) |
| `userType` | User type identifier (e.g., `"external"`) |
| `cwd` | Working directory at time of message |
| `gitBranch` | Active git branch |
| `timestamp` | ISO 8601 timestamp |
| `thinkingMetadata` | Extended thinking settings: `level`, `disabled`, `triggers` (array) |
| `toolUseMessages` | Array of tool calls and results |
| `todos` | Task list state at this message |
| `slug` | Whimsical session/agent name (e.g., `"sunny-hatching-neumann"`) |
| `requestId` | API request ID (assistant messages only, e.g., `"req_011CWfFS..."`) |
| `toolUseResult` | Short summary of tool result (user messages with tool results) |
| `agentId` | Sub-agent short ID (e.g., `"ac2a8dd"`, matches `agent-{id}.jsonl` filename) |

**Dev Journal Use**: **Primary source** - contains full context of what was accomplished, reasoning, tool usage, and file modifications.

#### Message Content Structure

The `message.content` field is polymorphic depending on message type:

**User messages (regular prompt)**:
```json
"message": { "role": "user", "content": "Help me implement..." }
```

**User messages (tool results)**:
```json
"message": {
  "role": "user",
  "content": [
    { "type": "tool_result", "tool_use_id": "toolu_01...", "content": "...", "is_error": false }
  ]
}
```

**Assistant messages** (array with multiple content types):
```json
"message": {
  "role": "assistant",
  "content": [
    { "type": "text", "text": "I'll help you..." },
    { "type": "thinking", "thinking": "...", "signature": "..." },
    { "type": "tool_use", "id": "toolu_01...", "name": "Bash", "input": {...} }
  ]
}
```

| Content Type | Fields | Description |
|--------------|--------|-------------|
| `text` | `text` | Plain text response |
| `thinking` | `thinking`, `signature` | Extended thinking content (when enabled) |
| `tool_use` | `id`, `name`, `input` | Tool invocation |
| `tool_result` | `tool_use_id`, `content`, `is_error` | Tool execution result |

---

### `file-history/` - Checkpointing

> See [Checkpointing Documentation](https://code.claude.com/docs/en/checkpointing)

**Purpose**: Versioned backups of files edited during sessions for undo/rollback.

**Structure**:
```
file-history/
└── 31f3f224-f440-41ac-9244-b27ff054116d/    # Session UUID
    ├── 59e0b9c43163e850@v1                    # {fileHash}@v{version}
    ├── 59e0b9c43163e850@v2
    ├── b926f55c84aa2cef@v1
    └── ...
```

**File Naming**: `{contentHash}@v{versionNumber}`

| Component | Description |
|-----------|-------------|
| `contentHash` | 16-character hex hash of file content (e.g., `59e0b9c43163e850`) |
| `versionNumber` | Sequential version within session (1, 2, 3...) |

**Content**: Raw file content at that version

**Correlation**:
- Directory name = `sessionId` from session transcript
- `backupFileName` in `file-history-snapshot` messages points to these files

#### `backupFileName` Field Behavior

The `backupFileName` field in `trackedFileBackups` entries can be:

| Value | When it occurs | Meaning |
|-------|---------------|---------|
| `"hash@v1"` | Editing existing files | Backup file exists in `file-history/{sessionId}/` |
| `null` | **Newly created files** (common at v1) | No previous content to back up |
| `null` | Edge cases (rare, any version) | Backup creation failed or was skipped |

**Example**: When Claude creates a new file, the first snapshot has `backupFileName: null`:
```json
{
  "trackedFileBackups": {
    "new-file.md": {
      "backupFileName": null,   // No backup - file didn't exist before
      "version": 1,
      "backupTime": "2025-12-11T15:50:50.185Z"
    },
    "existing-file.md": {
      "backupFileName": "fe8c3c23062c0e71@v1",  // Backup of original content
      "version": 1,
      "backupTime": "2025-12-11T15:52:13.999Z"
    }
  }
}
```

**Dev Journal Use**: Reconstruct file changes, generate diffs, understand what was modified. Entries with `backupFileName: null` indicate file creation rather than modification.

---

### `todos/` - Task Lists

**Purpose**: Persisted todo lists created during sessions.

**Structure**:
```
todos/
├── 31f3f224-...-agent-31f3f224-....json    # {sessionId}-agent-{agentId}.json
└── ...
```

**Content**: JSON array of todo items with status

**Dev Journal Use**: Track planned vs completed work within sessions.

---

### `plans/` - Plan Mode Documents

**Purpose**: Markdown implementation plans created during plan mode.

**Structure**:
```
plans/
├── cosmic-plotting-bunny.md      # Auto-generated whimsical names
├── eager-whistling-ember.md
└── ...
```

**Content Example**:
```markdown
# Chat Exchange Details Modal

## Goal
Add a clickable link to each message that opens a modal...

## Approach
Extend sessionStorage to store per-message metadata...

## Data Structure
### New sessionStorage key: `chat-rag-message-metadata`
...
```

**Correlation**: Plans are standalone - no explicit ID linking to sessions. They're referenced by conversation context.

**Dev Journal Use**: High-level planning artifacts, architectural decisions.

---

### `debug/` - Debug Logs

**Purpose**: Debug output for troubleshooting sessions.

**Structure**:
```
debug/
├── 31f3f224-f440-41ac-9244-b27ff054116d.txt
└── ...
```

**Naming**: `{sessionId}.txt`

**Format**: Plain text with timestamped debug entries:
```
2025-12-30T00:25:27.892Z [DEBUG] [SLOW OPERATION DETECTED] execSyncWithDefaults (21.7ms): security find-generic-password
2025-12-30T00:25:27.920Z [DEBUG] Watching for changes in setting files /Users/sam/.claude...
2025-12-30T00:25:27.952Z [DEBUG] [LSP MANAGER] initializeLspServerManager() called
```

---

### `session-env/` - Environment Variables

**Purpose**: Per-session environment variable storage.

**Structure**:
```
session-env/
├── 31f3f224-f440-41ac-9244-b27ff054116d/
└── ...
```

**Note**: Directories are typically created empty. Environment data may be stored transiently or populated only under specific conditions (e.g., when using session-specific environment overrides).

---

### `shell-snapshots/` - Shell State

**Purpose**: Captures shell environment for session restoration.

**Structure**:
```
shell-snapshots/
├── snapshot-zsh-1752622750085-qza877.sh
└── ...
```

**Naming**: `snapshot-{shell}-{timestamp}-{random}.sh`

---

## Configuration Directories

### `commands/` - Simple Slash Commands

**Purpose**: User-defined slash commands (invoked with `/command-name`).

**Structure**:
```
commands/
├── brainstorm.md
├── project-brainstorm.md
└── analyze-argument.md
```

**Format**: Markdown with YAML frontmatter

```markdown
---
name: brainstorm
description: I've got an idea I want to talk through with you.
---

[Command prompt content...]
```

---

### `skills/` - Complex Skills

> See [Skills Documentation](https://code.claude.com/docs/en/skills)

**Purpose**: Skills with multiple files, scripts, and more complex logic.

**Structure**:
```
skills/
├── dev-journal -> /path/to/skill/directory    # Can be symlinks
├── create-pr/
│   └── SKILL.md
└── frontend-design/
    ├── SKILL.md
    └── scripts/
```

**SKILL.md Format**:
```markdown
---
name: dev-journal
description: Track development work, learnings, and challenges...
allowed-tools: Bash Read Write
---

[Skill instructions...]
```

---

### `plugins/` - Plugin System

> See [Plugins Documentation](https://code.claude.com/docs/en/plugins)

**Purpose**: Plugin marketplace integration and installed plugins.

**Structure**:
```
plugins/
├── cache/                           # Installed plugin files
│   └── alteredcraft-plugins/
│       └── artifact-workflow/1.0.1/
├── config.json                      # Plugin system config
├── installed_plugins.json           # Registry of installed plugins
├── known_marketplaces.json          # Marketplace sources
├── install-counts-cache.json        # Download statistics
└── marketplaces/                    # Marketplace indices
```

**installed_plugins.json Example**:
```json
{
  "version": 2,
  "plugins": {
    "artifact-workflow@alteredcraft-plugins": [
      {
        "scope": "user",
        "installPath": "/Users/sam/.claude/plugins/cache/...",
        "version": "1.0.1",
        "installedAt": "2025-12-27T16:00:23.599Z",
        "gitCommitSha": "2a2e0720ea..."
      }
    ]
  }
}
```

---

## Settings Files

> See [Settings Documentation](https://code.claude.com/docs/en/settings)

### `settings.json`

**Purpose**: Global user settings applied to all projects.

**Key Sections**:
```json
{
  "permissions": {
    "allow": ["Bash(npm run lint)", "Read(~/.zshrc)"],
    "deny": ["Read(./.env)", "Read(./secrets/**)"]
  },
  "model": "opus",
  "hooks": {
    "UserPromptSubmit": [],
    "PreToolUse": [],
    "PostToolUse": [],
    "Stop": []
  },
  "statusLine": { "type": "command", "command": "..." },
  "enabledPlugins": {
    "artifact-workflow@alteredcraft-plugins": true
  },
  "alwaysThinkingEnabled": true
}
```


---

## `~/.claude.json` (Root Level)

> See [Settings Documentation](https://code.claude.com/docs/en/settings)

**Location**: `~/.claude.json` (note: NOT inside `~/.claude/`)

**Purpose**: System-managed state file for infrastructure, authentication, and runtime state. This file is **not intended for manual editing**.

### Data Categories

| Category | Fields | Description |
|----------|--------|-------------|
| **User Preferences** | `theme`, `preferredNotifChannel`, `showExpandedTodos` | UI and notification settings |
| **OAuth Session** | `oauthAccount.accountUuid`, `emailAddress`, `organizationUuid` | Authentication state |
| **MCP Server Configs** | `mcpServers` | User-scope MCP server definitions with credentials |
| **Per-Project State** | `projects.{path}.*` | Trust dialogs, last session, costs, project-level MCP servers |
| **Feature Flags** | `cachedStatsigGates`, `cachedGrowthBookFeatures` | A/B testing and feature rollout caches |
| **Usage Tracking** | `numStartups`, `tipsHistory`, `memoryUsageCount` | Onboarding and tip display state |

### Structure

```json
{
  "numStartups": 435,
  "installMethod": "global",
  "theme": "light",
  "autoUpdates": false,
  "preferredNotifChannel": "terminal_bell",
  "hasCompletedOnboarding": true,
  "userID": "bc0cdf7f...",

  "oauthAccount": {
    "accountUuid": "...",
    "emailAddress": "...",
    "organizationUuid": "..."
  },

  "mcpServers": {
    "context7": { "command": "...", "args": [...] },
    "chrome-devtools": { ... }
  },

  "projects": {
    "/Users/sam/Projects/my-app": {
      "allowedTools": [],
      "hasTrustDialogAccepted": true,
      "mcpServers": {},
      "lastSessionId": "659badf5-015f-46d5-bfdf-97d142279740",
      "lastCost": 0.288,
      "lastAPIDuration": 86639,
      "exampleFiles": ["App.vue", "main.ts"],
      "hasCompletedProjectOnboarding": true
    }
  },

  "tipsHistory": { "shift-enter": 431, "permissions": 434 },
  "cachedStatsigGates": { "tengu_use_file_checkpoints": true },
  "cachedGrowthBookFeatures": { ... }
}
```

### Per-Project State Fields

| Field | Type | Description |
|-------|------|-------------|
| `allowedTools` | array | Project-specific tool permissions |
| `hasTrustDialogAccepted` | boolean | Whether user accepted project trust dialog |
| `mcpServers` | object | Project-scope MCP server configurations |
| `lastSessionId` | string | UUID of most recent session in this project |
| `lastCost` | number | API cost of last session (USD) |
| `lastAPIDuration` | number | API response time of last request (ms) |
| `lastDuration` | number | Total duration of last session (ms) |
| `exampleFiles` | array | Auto-detected representative files for context |
| `hasCompletedProjectOnboarding` | boolean | Project onboarding completion state |
| `dontCrawlDirectory` | boolean | Disable automatic directory indexing |

---

## Relationship: `~/.claude.json` vs `~/.claude/settings.json`

These two files serve **complementary, non-duplicative** roles:

```
~/.claude.json          →  Infrastructure & State (system-managed)
~/.claude/settings.json →  Policies & Permissions (user-managed)
~/.claude/              →  Session data, transcripts, skills, plugins
```

### Comparison

| Aspect | `~/.claude.json` | `~/.claude/settings.json` |
|--------|-----------------|------------------------|
| **Location** | `~/` (home root) | `~/.claude/` |
| **Purpose** | Runtime state, caches, auth | Security rules, hooks, model |
| **Manual editing** | Not recommended | Intended for editing |
| **What it tracks** | *What has happened* | *What should happen* |
| **MCP servers** | User-scope definitions | N/A |
| **Permissions** | N/A | `allow`, `deny`, `ask` rules |
| **Per-project** | Trust, sessions, costs | Use `.claude/settings.json` in projects |
| **Git tracking** | Never commit | Project-level can be committed |

### Why Two Files?

1. **Separation of Concerns**
   - `~/.claude.json` = ephemeral/computed state (sessions, caches, OAuth tokens)
   - `settings.json` = intentional policy decisions (permissions, hooks, model selection)

2. **Different Lifecycles**
   - `~/.claude.json` changes constantly (every startup increments `numStartups`)
   - `settings.json` changes only when you modify configuration

3. **Security Boundary**
   - `~/.claude.json` contains OAuth tokens and MCP server credentials — never commit
   - `settings.json` (and project-level `.claude/settings.json`) is safe to version control

4. **Historical Design**
   - `~/.claude.json` predates the `~/.claude/` directory structure
   - It remains the "lightweight preferences" file while `~/.claude/` handles heavier data

---

## ID Correlation Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Session UUID (Primary Key)                       │
│            e.g., 31f3f224-f440-41ac-9244-b27ff054116d               │
└─────────────────────────────────────────────────────────────────────┘
         │
         ├──► projects/{project-path}/{sessionId}.jsonl
         │         └── type: user/assistant/file-history-snapshot/queue-operation
         │         └── uuid/parentUuid: message chain
         │         └── messageId: references file-history snapshots
         │
         ├──► projects/{project-path}/agent-{shortId}.jsonl
         │         └── Sub-agent transcripts (spawned by Task tool)
         │         └── shortId is truncated hash (e.g., agent-a980ab1.jsonl)
         │
         ├──► file-history/{sessionId}/
         │         └── {fileHash}@v{version}
         │         └── Referenced by snapshot.trackedFileBackups
         │
         ├──► todos/{sessionId}-agent-{agentId}.json
         │         └── agentId may equal sessionId (main) or differ (sub-agent)
         │
         ├──► session-env/{sessionId}/
         │
         └──► debug/{sessionId}.txt

┌─────────────────────────────────────────────────────────────────────┐
│                    Message UUID Chain                               │
│                  (within session transcript)                        │
└─────────────────────────────────────────────────────────────────────┘
         │
         user message (uuid: abc123, parentUuid: null)
              │
              └──► assistant message (uuid: def456, parentUuid: abc123)
                        │
                        └──► file-history-snapshot (messageId: def456)

┌─────────────────────────────────────────────────────────────────────┐
│                    Sub-Agent Relationship                           │
│                  (Task tool spawns sub-agents)                      │
└─────────────────────────────────────────────────────────────────────┘
         │
         Main session: 31f3f224-f440-41ac-9244-b27ff054116d.jsonl
              │
              └──► Sub-agent: agent-a980ab1.jsonl
              │         └── Independent transcript
              │         └── Own file-history, todos possible
              │         └── Messages have: agentId, slug, isSidechain: true
              │
              └──► Sub-agent: agent-acdf3e5.jsonl

Sub-agent messages inherit the parent `sessionId` but add:
- `agentId`: Short ID matching the filename (e.g., `"a980ab1"`)
- `slug`: Whimsical name (e.g., `"sunny-hatching-neumann"`)
- `isSidechain: true`: Always set for sub-agent messages
```

---

## Dev Journal Integration Strategy

### Data Sources by Use Case

| Need | Primary Source | Secondary Source |
|------|----------------|------------------|
| "What did I work on today?" | `history.jsonl` | `projects/*.jsonl` |
| "What files did I change?" | `file-history/` | Session transcripts |
| "What problems did I solve?" | `projects/*.jsonl` | - |
| "How much did I use Claude?" | `stats-cache.json` | - |
| "What was my plan?" | `plans/*.md` | Session transcripts |

### Recommended Extraction Flow

```
1. history.jsonl
   └── Get prompts for date range
   └── Extract project paths and timestamps

2. projects/{path}/*.jsonl
   └── Load sessions for those projects
   └── Extract assistant reasoning, tool usage
   └── Identify file modifications

3. file-history/{sessionId}/
   └── Reconstruct actual file changes
   └── Generate diffs if needed

4. stats-cache.json
   └── Add activity metrics
   └── Session durations, message counts
```

---

## References

- [Memory Documentation](https://code.claude.com/docs/en/memory)
- [Settings Documentation](https://code.claude.com/docs/en/settings)
- [Checkpointing Documentation](https://code.claude.com/docs/en/checkpointing)
- [Skills Documentation](https://code.claude.com/docs/en/skills)
- [Plugins Documentation](https://code.claude.com/docs/en/plugins)
- [Analytics Documentation](https://code.claude.com/docs/en/analytics)
- [Full Documentation Index](https://code.claude.com/docs/llms.txt)

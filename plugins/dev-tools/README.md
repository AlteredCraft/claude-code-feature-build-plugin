# Dev Tools Plugin

Developer tools plugin with skills for journaling and productivity.

## Installation

```bash
claude plugin install alteredcraft-plugins/dev-tools
```

## Skills

### journal

A development journaling skill that helps developers reflect on and document their work using Claude Code activity data.

**Usage:**

```
Skill(dev-tools:journal)
```

When triggered, the skill will:

1. Ask for a time period (if not specified)
2. Gather activity data from Claude Code directories
3. Synthesize a narrative journal entry
4. Offer to refine until you're satisfied

**Features:**

- **User-centric narratives**: Captures what you accomplished, not AI actions
- **Selective highlighting**: Gauges significance to highlight meaningful work
- **Multi-day summaries**: Provides highlights and day-by-day breakdowns
- **Iterative refinement**: Always offers to revise before finalizing

**Data Sources:**

- `~/.claude/history.jsonl` - Activity timeline
- `~/.claude/projects/` - Session transcripts
- `~/.claude/file-history/` - File modifications
- `~/.claude/todos/` - Task completion status
- `~/.claude/stats-cache.json` - Usage metrics

See `skills/journal/references/claude-code-directory.md` for detailed documentation of these data structures.

**Example Output:**

```markdown
## Dev Journal: January 3, 2026

You had a focused day on the Claude Explorer API. The highlight was implementing the activity summary endpoint—something you'd been planning for a while.

**Claude Explorer** - Implemented the activity summary endpoint
- Designed the response schema for cross-project aggregation
- Added date range filtering with proper timezone handling
- ✅ Add /activity/summary endpoint
- ✅ Add /activity endpoint for detailed timeline

*Also accomplished: Fixed a null pointer in the todos parser*
```

## License

MIT

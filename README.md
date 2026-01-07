# AlteredCraft Claude Code Plugins

A collection of plugins for [Claude Code](https://claude.com/claude-code).

## Installation

```bash
# Add the marketplace
/plugin marketplace add AlteredCraft/claude-code-plugins

# Install a plugin
/plugin install <plugin-name>@alteredcraft-plugins
```

For more details, see the [Claude Code Plugins documentation](https://code.claude.com/docs/en/plugins).

## Available Plugins

### [artifact-workflow](./plugins/artifact-workflow/)

An artifact-driven development workflow inspired by Google Antigravity. Adds a `/artifact-workflow:build` command that guides you through interrogation, planning, and execution phases with persistent artifacts for task lists, implementation plans, and walkthroughs.

```bash
/plugin install artifact-workflow@alteredcraft-plugins
```

**❗ Restart claude**
```bash
/artifact-workflow:build --help
```

### [dev-consultant](./plugins/dev-consultant/)

A structured product consultation workflow that helps refine project ideas into fully-formed design specifications through strategic questioning and iterative refinement.

```bash
/plugin install dev-consultant@alteredcraft-plugins
```

**❗ Restart claude**
```bash
/dev-consultant:brainstorm <your project idea>
```

### [dev-tools](./plugins/dev-tools/)

Developer tools with skills for journaling and productivity. Includes a `journal` skill that synthesizes Claude Code activity data into meaningful journal entries.

```bash
/plugin install dev-tools@alteredcraft-plugins
```

**❗ Restart claude**
```
Skill(dev-tools:journal)
```

## License

MIT

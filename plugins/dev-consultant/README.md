# Dev Consultant

A structured product consultation workflow for Claude Code that helps refine project ideas into fully-formed design specifications.

## What It Does

This plugin adds a `/dev-consultant:brainstorm` command that guides you through a three-phase consultation process:

1. **Idea Refinement** — Strategic questioning to understand purpose, scope, requirements, constraints, and success criteria
2. **Design Presentation** — Section-by-section design specification with iterative feedback
3. **Final Specification** — Comprehensive design document with justifications and research links

## Why Use It

Ideas are messy. Jumping straight to implementation often means building the wrong thing. This workflow forces structured thinking before coding begins:

- Surfaces hidden assumptions through focused questions
- Builds shared understanding incrementally
- Captures design rationale alongside decisions
- Produces a reference spec you can return to

## Installation

### From GitHub

```bash
# Add the marketplace
/plugin marketplace add AlteredCraft/claude-code-plugins

# Install the plugin
/plugin install dev-consultant@alteredcraft-plugins
```

### Manual Installation

Clone this repository and add it as a local marketplace:

```bash
git clone https://github.com/AlteredCraft/claude-code-plugins.git
cd claude-code-plugins

# In Claude Code:
/plugin marketplace add ./
/plugin install dev-consultant@alteredcraft-plugins
```

## Usage

### Start a consultation

```bash
/dev-consultant:brainstorm
```

Or with an initial idea:

```bash
/dev-consultant:brainstorm Build a CLI tool for managing dotfiles across machines
```

### The process

1. Claude asks focused questions one at a time, offering suggestions
2. Once requirements are clear, Claude presents the design in sections
3. You approve or refine each section before moving on
4. Final output is a comprehensive design specification

## License

MIT

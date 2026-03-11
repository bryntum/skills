# Bryntum Skills for Claude Code

This repository contains Claude Code skills for working with [Bryntum](https://bryntum.com) components — Scheduler, Scheduler Pro, Gantt, Calendar, Grid, and TaskBoard.

## Skills

### `bryntum`

Guides Claude through integrating Bryntum components into web apps. Covers:

- Package installation (trial and licensed)
- CSS setup for Bryntum 7+ (plain CSS, no SASS/SCSS)
- Framework-specific patterns (Vanilla JS, React, Angular, Vue)
- Component sizing
- Bryntum docs lookup via MCP

## Installation

Copy the `bryntum/` directory into your `.claude/skills/` folder:

```bash
cp -r bryntum ~/.claude/skills/
```

Or clone and copy in one step:

```bash
git clone https://github.com/bryntum/bryntum-skills.git
cp -r bryntum-skills/bryntum ~/.claude/skills/
```

## Requirements

- [Claude Code](https://claude.ai/code)
- The [Bryntum MCP server](https://github.com/bryntum/bryntum-mcp) for docs lookup (optional but recommended)

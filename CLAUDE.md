# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin marketplace — a collection of reusable skills, commands, and agents distributed via the Claude plugin system. There is no build process, test suite, or compiled code. Everything is documentation-driven markdown.

## Repository Structure

- `.claude-plugin/marketplace.json` — Marketplace definition (top-level, points to `./plugin`)
- `plugin/.claude-plugin/plugin.json` — Plugin metadata (registers skills, commands, and agents)
- `plugin/skills/<skill-name>/SKILL.md` — Skill definition (what Claude executes when the skill is invoked)
- `plugin/skills/<skill-name>/CREDITS.md` — Attribution for the skill's origin
- `plugin/commands/<command-name>.md` — Command definition (single markdown file, user-invoked via `/command-name`)
- `plugin/agents/<agent-name>.md` — Agent definition (markdown with YAML frontmatter, used as subagent types)

## Adding Content

### Skills
1. Create a directory under `plugin/skills/<skill-name>/`
2. Write `SKILL.md` — the full prompt/instruction set
3. Add `CREDITS.md` if based on existing work
4. Update the Skills table in `README.md`

### Commands
1. Create a markdown file at `plugin/commands/<command-name>.md`
2. Update the Commands table in `README.md`

### Agents
1. Create a markdown file at `plugin/agents/<agent-name>.md` with YAML frontmatter (`name`, `description`)
2. Update the Agents table in `README.md`

## Skill Design Conventions

Skills in this repo follow a phase-based workflow structure:

- **Context Discovery** comes first (read existing files, understand the codebase)
- **Structured information gathering** uses multiple rounds of targeted questions before acting
- **Propose alternatives** before committing to an approach (typically 2–3 options)
- **Delegation model**: skills may spawn subagents and define a reporting contract for what they return

See `plugin/skills/feature-design-assistant/SKILL.md` for a full example of a complex multi-phase skill, and `plugin/skills/make-plan/SKILL.md` for a simpler orchestration skill.

## Distribution

The plugin is distributed as a GitHub-hosted marketplace. Users install it by referencing `simenmorch/claude` in their `~/.claude/plugins/known_marketplaces.json`. No publishing step is required — pushing to `main` is sufficient.

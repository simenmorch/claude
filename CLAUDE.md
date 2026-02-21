# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin marketplace — a collection of reusable skills (slash commands) distributed via the Claude plugin system. There is no build process, test suite, or compiled code. Everything is documentation-driven markdown.

## Repository Structure

- `.claude-plugin/marketplace.json` — Marketplace definition (top-level, points to `./plugin`)
- `plugin/.claude-plugin/plugin.json` — Plugin metadata for the installable skill set
- `plugin/skills/<skill-name>/SKILL.md` — The skill definition (this is what Claude executes)
- `plugin/skills/<skill-name>/CREDITS.md` — Attribution for the skill's origin

## Adding a New Skill

1. Create a directory under `plugin/skills/<skill-name>/`
2. Write `SKILL.md` — this is the full prompt/instruction set that Claude will receive when the skill is invoked
3. Add `CREDITS.md` if the skill is based on existing work
4. Update the table in `README.md`

## Skill Design Conventions

Skills in this repo follow a phase-based workflow structure:

- **Context Discovery** comes first (read existing files, understand the codebase)
- **Structured information gathering** uses multiple rounds of targeted questions before acting
- **Propose alternatives** before committing to an approach (typically 2–3 options)
- **Delegation model**: skills may spawn subagents and define a reporting contract for what they return

See `plugin/skills/feature-design-assistant/SKILL.md` for a full example of a complex multi-phase skill, and `plugin/skills/make-plan/SKILL.md` for a simpler orchestration skill.

## Distribution

The plugin is distributed as a GitHub-hosted marketplace. Users install it by referencing `simenmorch/claude` in their `~/.claude/plugins/known_marketplaces.json`. No publishing step is required — pushing to `main` is sufficient.

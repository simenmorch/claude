# claude-code

A personal collection of reusable Claude Code skills, slash commands, and AI workflow tools.

## Installation

Add this repo as a marketplace in `~/.claude/plugins/known_marketplaces.json`:

```json
{
  "simenmorch": {
    "source": {
      "source": "github",
      "repo": "simenmorch/claude"
    },
    "installLocation": "~/.claude/plugins/marketplaces/simenmorch"
  }
}
```

## Skills

| Skill | Description |
|-------|-------------|
| [feature-design-assistant](plugin/skills/feature-design-assistant/) | Turn ideas into fully formed designs and specs through natural collaborative dialogue |
| [make-plan](plugin/skills/make-plan/) | Create a detailed, phased implementation plan with documentation discovery |

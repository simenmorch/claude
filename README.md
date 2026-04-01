# claude-code

A personal collection of reusable Claude Code skills, commands, agents, and AI workflow tools.

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
| [frontend-design](plugin/skills/frontend-design/) | Create distinctive, production-grade frontend interfaces with high design quality |
| [debugger](plugin/skills/debugger/) | Systematic debugging and root cause analysis for identifying and fixing software issues |
| [grill-me](plugin/skills/grill-me/) | Interview the user relentlessly about a plan or design until reaching shared understanding |

## Commands

| Command | Description |
|---------|-------------|
| [deep-review](plugin/commands/deep-review.md) | Multi-agent code review with confidence filtering — spawns specialized reviewers in parallel, scores findings with Haiku, and surfaces only high-signal issues |

## Agents

Specialized review agents used by the `deep-review` command. Can also be invoked individually via `@agent-simenmorch-skills:reviewer-*`.

| Agent | Focus |
|-------|-------|
| [reviewer-security](plugin/agents/reviewer-security.md) | Security vulnerabilities, authorization gaps, debug artifacts |
| [reviewer-logic](plugin/agents/reviewer-logic.md) | Business logic correctness, null safety, race conditions, fail-fast |
| [reviewer-testing](plugin/agents/reviewer-testing.md) | Test coverage, behavior-driven testing, isolation |
| [reviewer-commits](plugin/agents/reviewer-commits.md) | Commit atomicity, ordering, message quality |
| [reviewer-git-history](plugin/agents/reviewer-git-history.md) | Git blame analysis, reverted decisions, historical context |
| [reviewer-code-comments](plugin/agents/reviewer-code-comments.md) | Comment contract compliance, orphaned comments, broken references |
| [reviewer-design](plugin/agents/reviewer-design.md) | UI/UX low-hanging fruit (conditional — only when UI files changed) |

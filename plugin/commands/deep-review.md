# Deep Review — Multi-Agent Code Review

Spawn specialized review agents in parallel, then filter findings by confidence to surface only high-signal issues.

## Task

$ARGUMENTS

## Instructions

### 1. Determine Scope

Parse `$ARGUMENTS` to determine the review scope. Three modes:

- **Single commit**: `$ARGUMENTS` is a commit hash (e.g., `abc123`)
  - Diff command: `git diff <commit>^..<commit>`
  - Commit log: `git show --stat <commit>`
- **Branch vs custom base**: `$ARGUMENTS` is a branch name (e.g., `feat/branch-a`)
  - Diff command: `git diff <branch>...HEAD`
  - Commit log: `git log <branch>..HEAD --oneline`
  - Use case: reviewing branch B against branch A when A is reviewed independently
- **Branch vs main** (default): `$ARGUMENTS` is empty
  - Diff command: `git diff main...HEAD`
  - Commit log: `git log main..HEAD --oneline`

**How to distinguish commit hash from branch name**: Run `git cat-file -t <arg>` — if it returns `commit`, check if `git branch --list <arg>` also matches. If it's a known branch name, treat it as a base branch. If it looks like a short/full SHA (hex-only, 7-40 chars), treat it as a commit.

### 2. Gather Context

Run the appropriate git diff commands from step 1 to get the changed file list:

```bash
git diff [scope] --name-only
git diff [scope] --stat
```

Store:
- The diff scope command (agents will run it themselves to examine the full diff)
- The changed file list
- The original `$ARGUMENTS` (users may include extra context like focus areas)

### 3. Select and Spawn Review Agents

#### Always spawn (core agents):

These agents provide unique value that a standard code review does not cover:

| # | `subagent_type`          | Why it's always-on |
|---|--------------------------|-----|
| 1 | `reviewer-security`      | Dedicated security lens + debug artifact detection |
| 2 | `reviewer-logic`         | Fail-fast philosophy, null safety, race conditions |
| 3 | `reviewer-testing`       | Coverage gaps, behavior vs implementation testing |
| 4 | `reviewer-commits`       | Commit atomicity, ordering, message quality |
| 5 | `reviewer-git-history`   | Blame-checking for reverted decisions and churn |
| 6 | `reviewer-code-comments` | Comment contract compliance, orphaned comments |

#### Conditionally spawn:

Scan the changed file list and spawn these agents only when relevant files are present:

| `subagent_type`     | Spawn when changed files include |
|---------------------|----------------------------------|
| `reviewer-design`   | Templates, components, views, stylesheets, or frontend files (e.g., `*.html`, `*.jsx`, `*.tsx`, `*.vue`, `*.svelte`, `*.blade.php`, `*.css`, `*.scss`, files with `component`, `page`, `view`, `widget`, `layout` in the path) |

#### Spawning rules

- Spawn all selected agents **in a single message** (parallel execution)
- Each agent receives in its task prompt:
  - The diff scope command to run (so the agent can examine the diff itself)
  - The list of changed files
  - The original `$ARGUMENTS` (so agents understand any user-specified focus areas)

### Error Recovery

- **Agent fails or times out**: Report which agent(s) failed and present results from the agents that succeeded
- **Empty diff**: Inform the user that no changes were found for the given scope

### 4. Confidence Filtering

After all agents return, collect every individual finding (Critical, Warning, or Suggestion) from all agents into a
flat list. Then spawn **parallel Haiku agents** — one per agent that reported findings — to score each finding.

Each Haiku scoring agent receives:
- The original diff scope command (so it can verify against the actual code)
- The list of changed files
- The agent's findings to score
- The list of CLAUDE.md files in the repository (root + any in modified directories)

**Scoring rubric** (give this to each Haiku agent verbatim):

Score each finding on a scale from 0-100 indicating confidence that it is a real, actionable issue:

- **0**: False positive. Does not stand up to scrutiny, or is a pre-existing issue not introduced by this diff.
- **25**: Might be real, but likely a false positive or stylistic nitpick not explicitly required by project conventions.
- **50**: Moderately confident. Real issue but minor — a nitpick or edge case unlikely to cause problems in practice.
- **75**: Highly confident. Verified real issue that will impact functionality, or directly violates a documented convention. The existing approach in the diff is insufficient.
- **100**: Certain. Confirmed real issue that will cause bugs or security problems in practice. Evidence directly confirms this.

**Examples of findings that should score LOW (0-25):**
- Pre-existing issues not introduced by the diff
- Issues a linter, type checker, or CI would catch (formatting, imports, type errors)
- General code quality opinions not backed by project conventions
- Pedantic nitpicks a senior engineer wouldn't flag
- Intentional changes in functionality related to the PR's purpose
- Issues on lines the diff did not modify

**Examples of findings that should score HIGH (75-100):**
- Direct violations of CLAUDE.md or documented project conventions
- Reverting a deliberate past bug fix without acknowledgment
- SQL injection or XSS vulnerabilities
- Missing authorization checks
- Broken cross-references or violated code contracts

### 5. Apply Filter

- **Keep** all findings scored **>= 60**
- **Drop** all findings scored **< 60**
- If an agent's findings are ALL dropped, omit that agent's section entirely from the output
- Preserve the original severity labels (Critical/Warning/Suggestion) — the confidence score only determines inclusion, not severity

### 6. Present Results

Present only the surviving findings, grouped by agent area:

```
## [Area] Review — [VERDICT]

[Filtered findings only — with confidence score shown inline]

- [severity] [issue] → `file:line` — [explanation] (confidence: XX)
```

If ALL findings from ALL agents were filtered out:
```
## Review Complete — APPROVED

All findings were below the confidence threshold. No high-signal issues detected.
```

### 7. Summary Table

Aggregate verdicts into a final table (only include agents that ran and had surviving findings, plus a row for any agent that was fully filtered):

```
## Summary

| Area | Verdict | Critical | Warnings | Suggestions | Filtered Out |
|------|---------|----------|----------|-------------|--------------|
| Security & Hygiene | ... | ... | ... | ... | ... |
| Logic & Robustness | ... | ... | ... | ... | ... |
| Testing | ... | ... | ... | ... | ... |
| Commit History | ... | ... | ... | ... | ... |
| Git History | ... | ... | ... | ... | ... |
| Code Comments | ... | ... | ... | ... | ... |
| Design/UI/UX | ... | ... | ... | ... | ... |

**Overall: [APPROVED | SUGGESTIONS | NEEDS WORK]**
```

Note: Only include rows for agents that were actually spawned. The Design/UI/UX row only appears when UI files were in the diff.

**Verdict rules:**
- Any **Critical** with confidence >= 60 → overall **NEEDS WORK**
- Any **Warning** with confidence >= 60, no Critical → overall **SUGGESTIONS**
- Only **Suggestions** or everything filtered → overall **APPROVED**

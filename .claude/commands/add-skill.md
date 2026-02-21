---
description: Import a skill from a GitHub URL and add it to this repository
argument-hint: <github-url>
---

# Add Skill

Import a skill from the GitHub URL provided in `$ARGUMENTS` and add it to this repository under `plugin/skills/`.

## Phase 1: Parse the URL

Parse `$ARGUMENTS` to extract:
- GitHub owner and repository name
- The path within the repo (if any)
- The skill folder name: use the last meaningful path segment, or the repo name if pointing to a repo root

URL patterns to handle:
- `https://github.com/<owner>/<repo>` → skill name = `<repo>`
- `https://github.com/<owner>/<repo>/tree/<branch>/<path>` → skill name = last segment of `<path>`
- `https://github.com/<owner>/<repo>/blob/<branch>/<path>/SKILL.md` → skill name = parent folder of `SKILL.md`

Normalize the skill name to lowercase kebab-case.

## Phase 2: Fetch SKILL.md Content

Construct a raw content URL by:
- Replacing `github.com` with `raw.githubusercontent.com`
- Removing `/blob/` or `/tree/` (replace with `/`)

If the URL points to a folder or repo root, try fetching `SKILL.md` from these locations in order:
1. `https://raw.githubusercontent.com/<owner>/<repo>/main/SKILL.md`
2. `https://raw.githubusercontent.com/<owner>/<repo>/master/SKILL.md`
3. `https://raw.githubusercontent.com/<owner>/<repo>/main/<path>/SKILL.md`

Use WebFetch to retrieve the content. If not found at any location, report the failure and stop.

## Phase 3: Verify License

Check for a LICENSE file in this order of preference (most specific to least specific):
1. Next to the SKILL.md file: `https://raw.githubusercontent.com/<owner>/<repo>/main/<skill-path>/LICENSE`
2. Repo root: `https://raw.githubusercontent.com/<owner>/<repo>/main/LICENSE`
3. Repo root with `.md` extension: `https://raw.githubusercontent.com/<owner>/<repo>/main/LICENSE.md`

(Try both `main` and `master` for each.)

From the LICENSE file content, identify the license type (e.g. MIT, Apache-2.0, GPL-3.0, AGPL-3.0).

**If no LICENSE file is found anywhere:** proceed and use `unknown license` in CREDITS.md.

## Phase 4: Check for Conflicts

Check whether `plugin/skills/<skill-name>/` already exists using the Glob tool.
If it does, inform the user that the skill already exists and stop without creating any files.

## Phase 5: Create Skill Files

Create two files:

**`plugin/skills/<skill-name>/SKILL.md`**
Write the raw fetched content exactly as retrieved.

**`plugin/skills/<skill-name>/CREDITS.md`**
Write attribution in this exact format (matching existing CREDITS.md files in the repo):
```
Based on [<repo-name>](<original-github-url>) by [<owner>](https://github.com/<owner>), licensed under <license>.
```

## Phase 6: Update README.md

Read `README.md`, then append a new row to the Skills table:
```
| [<skill-name>](plugin/skills/<skill-name>/) | <description> |
```

For `<description>`: use the `description` field from the SKILL.md frontmatter if present, otherwise use the first non-heading paragraph of the SKILL.md content.

## Phase 7: Report

Output a summary:
- Skill name and folder created
- Files written
- Source URL and license attribution

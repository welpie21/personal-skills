---
name: weekly-recap
description: >-
  Summarise the user's commits and pull requests from the past week across
  ~/Documents/Github. Uses git log and GitHub CLI. Use when the user asks for a
  weekly summary, what they worked on last week, or a recap of recent changes
  across their projects.
---

# Weekly Summary

Produce a recap of the **user's own work** across git repos under `~/Documents/Github`.
Use commit history as the source of truth; enrich with GitHub CLI where available.

## Output format

Respond with **only** a markdown list — no intro, no outro, no tables:

```markdown
- **project-name** — One-sentence summary of what changed.
```

Rules:

- Include a project **only if the user has commits** in the date range.
- One bullet per project; keep each summary to one short sentence.
- Use the directory name as the project name (e.g. `surrealdb.js`, not the remote URL).
- Mention merged PRs or open branches only when they add context; do not list every commit.

## Date range

Default: **past 7 calendar days** ending today.

If the user says "last week", use the **previous Monday–Sunday** window.

Compute dates explicitly before querying:

```bash
# Last 7 days (default)
SINCE=$(date -v-7d +%Y-%m-%d)   # macOS
UNTIL=$(date +%Y-%m-%d)

# Previous calendar week (Mon–Sun)
# Adjust SINCE/UNTIL accordingly
```

## Step 1 — Identify the user

Read git identity from global config:

```bash
git config --global user.name
git config --global user.email
```

Filter commits with `--author` matching **name or email**. Also match short forms
(e.g. `Julian` when name is `Julian Mills`).

## Step 2 — Discover repos

Scan `~/Documents/Github` for git repositories (one level deep):

```bash
for dir in ~/Documents/Github/*/; do
  [ -d "$dir/.git" ] && echo "$dir"
done
```

Skip directories that are not git repos. Do not recurse into subdirectories.

## Step 3 — Collect commits per repo

For each repo, fetch the user's commits in the date range:

```bash
git -C "$repo" log \
  --since="$SINCE" --until="$UNTIL" \
  --author="$AUTHOR" \
  --format="%h %ad %s" --date=short
```

If zero commits, skip the repo.

For repos with commits, optionally inspect scope:

```bash
git -C "$repo" log --since="$SINCE" --until="$UNTIL" \
  --author="$AUTHOR" --stat --oneline
```

Note the current branch and whether work is merged or still on a feature branch.

## Step 4 — Enrich with GitHub CLI

If `gh auth status` succeeds, look up PRs for repos with activity.

Resolve the remote owner/repo:

```bash
git -C "$repo" remote get-url origin
```

Search PRs authored by the user in the date range:

```bash
gh search prs \
  --author=@me \
  --created=">$SINCE" \
  --repo OWNER/REPO \
  --json number,title,state,mergedAt \
  --limit 20
```

Also check PRs updated in the range (for ongoing work):

```bash
gh pr list --repo OWNER/REPO --author @me --state all \
  --json number,title,state,updatedAt,headRefName \
  --limit 20
```

If `gh` is unavailable or unauthenticated, rely on git log only — do not fail, and warn the user that `gh` was not available.

## Step 5 — Synthesise

Group all evidence (commits, PR titles, branch names, file areas) into no more than three sentences per project. Focus on **what shipped or progressed**, not commit counts.

Good: "Extracted SQON into its own package with publish workflow and SDK integration."
Bad: "14 commits on feat/sqon."

## Step 6 — Respond

Output **only** the bullet list. No headings, no "you also had no activity in…"
section for quiet repos.

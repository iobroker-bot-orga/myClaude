# Prompt: Add a "removed from stable repository" history entry

## Required inputs (fill these in before using this prompt)

- **Adapter repository URL:** `https://github.com/<org>/<repo>` (e.g. `https://github.com/Newan/ioBroker.easee`)

---

## Task

Create a pull request against `ioBroker/ioBroker.docs` (base branch `master`) that adds a
"removed from stable repository" entry for the given adapter to
`docs/en/history/history.md`.

---

## Step 1 – Resolve adapter details

```bash
gh repo view <org>/<repo> --json description,defaultBranchRef \
  --jq '{desc: .description, branch: .defaultBranchRef.name}'
```

Note the **default branch** (usually `main` or `master`) — it is needed for the raw icon URL.

Strip `ioBroker.` from the repo name to get the short **adapter name** used in the heading
(e.g. `ioBroker.easee` → `easee`).

---

## Step 2 – Find the icon filename

```bash
gh api repos/<org>/<repo>/contents/admin --jq '.[].name'
```

The icon is typically `<adaptername>.png` inside the `admin/` directory. Use whatever
`.png` or `.svg` file is there that matches the adapter name.

---

## Step 3 – Write a one-sentence description

Fetch the adapter's README and take the first meaningful sentence that describes what the
adapter does. Use **past tense** — the adapter is no longer active.

```bash
# via WebFetch:
# https://raw.githubusercontent.com/<org>/<repo>/<branch>/README.md
```

The `description` field from `gh repo view` in Step 1 is a useful fallback.

Do **not** include version numbers in the description.

---

## Step 4 – Write the history entry

Insert the new entry **just before** the closing comment block at the very end of
`docs/en/history/history.md`:

```
<!--  ######################## keep these lines at end of file as a reminder ######################## -->
```

### Entry format

```markdown
## <adaptername> (D.M.YYYY) – removed from stable repository
https://github.com/<org>/<repo>

<img src="https://raw.githubusercontent.com/<org>/<repo>/<branch>/admin/<icon-filename>" width="100" height="100" />

<One sentence (past tense) describing what the adapter did.> The adapter is no longer maintained and is no longer functional — it has been removed from the ioBroker stable repository. Users should look for an alternative adapter or community fork.
```

### Rules

- **Heading:** short adapter name only — no `ioBroker.` prefix.
- **Date:** use today's date in `D.M.YYYY` format (day and month without leading zeros,
  four-digit year), e.g. `14.10.2026` — never ISO format.
- **Label:** exactly `– removed from stable repository` (en-dash, not hyphen).
- **Link:** the full GitHub URL of the adapter repo, on its own line, no label.
- **Icon:** always `https://raw.githubusercontent.com/…` — never `github.com`. Use the
  default branch name resolved in Step 1.
- **Description:** one sentence in **past tense** describing what the adapter did, followed
  by the fixed removal notice. No version numbers, no bullet points.

---

## Step 5 – Create the branch and commit

```bash
git checkout master
git pull

git checkout -b <adaptername>-removed-whats-new

git add docs/en/history/history.md
git commit -m "news: add <adaptername> removed from stable repository entry

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Step 6 – Push to the fork and open the PR

The working copy's `origin` remote points to `ioBroker/ioBroker.docs` (no push access).
Push to the user's fork instead:

```bash
# Add the fork remote if it is not present yet
git remote add fork https://github.com/mcm1957/ioBroker.docs.git

git push -u fork <adaptername>-removed-whats-new
```

Write the PR body to a temp file, then create the PR:

```bash
cat > /tmp/pr_body.md <<'EOF'
Adds a "What's new" entry for ioBroker.<adaptername> to `docs/en/history/history.md`
noting that the adapter has been removed from the ioBroker stable repository. The adapter
is no longer maintained and is no longer functional.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF

gh pr create \
  --repo ioBroker/ioBroker.docs \
  --head mcm1957:<adaptername>-removed-whats-new \
  --base master \
  --title "news: add <adaptername> removed from stable repository entry" \
  --body-file /tmp/pr_body.md
```

---

## Reference example

PR: https://github.com/ioBroker/ioBroker.docs/pull/748
Adapter: `Newan/ioBroker.easee`

# Prompt: Add a "new adapter at beta (latest) repository" history entry

## Required inputs (fill these in before using this prompt)

- **Adapter repository:** `<org>/<repo>` (e.g. `BenAhrdt/ioBroker.device-monitoring`)

---

## Task

Create a pull request against `ioBroker/ioBroker.docs` (base branch `master`) that adds a
"new adapter at beta (latest) repository" entry for the given adapter to
`docs/en/history/history.md`.

---

## Step 1 – Resolve adapter details

```bash
gh repo view <org>/<repo> --json description,defaultBranchRef
```

Note the **default branch** (usually `main` or `master`) — it is needed for the raw icon URL.

Strip `ioBroker.` from the repo name to get the short **adapter name** used in the heading
(e.g. `ioBroker.device-monitoring` → `device-monitoring`).

---

## Step 2 – Find the icon filename

```bash
gh api repos/<org>/<repo>/contents/admin --jq '.[].name'
```

The icon is typically `<adaptername>.png` inside the `admin/` directory. Use whatever
`.png` or `.svg` file is there. If several images are listed, pick the one that matches
the adapter name.

---

## Step 3 – Write a one-sentence description

Fetch the adapter's README and take the first meaningful sentence that describes what the
adapter does. Installation steps, badges and headings are not appropriate.

```bash
# via WebFetch:
# https://raw.githubusercontent.com/<org>/<repo>/<branch>/README.md
```

Alternatively, the `description` field from `gh repo view` in Step 1 is often good enough
for a short adapter.

Do **not** include version numbers in the description.

---

## Step 4 – Confirm the adapter is in beta (latest) but not stable

```bash
# Check beta/latest list (sources-dist.json):
# https://raw.githubusercontent.com/ioBroker/ioBroker.repositories/master/sources-dist.json
# Check stable list (sources-dist-stable.json):
# https://raw.githubusercontent.com/ioBroker/ioBroker.repositories/master/sources-dist-stable.json
```

Use WebFetch on both URLs and search for the adapter name. If it appears in
`sources-dist-stable.json` the correct label is `– new adapter at stable repository`
instead (see `CLAUDE-newsUpdateStableRelease.md` for that flow).

---

## Step 5 – Write the history entry

Insert the new entry **just before** the closing comment block at the very end of
`docs/en/history/history.md`:

```
<!--  ######################## keep these lines at end of file as a reminder ######################## -->
```

### Entry format

```markdown
## <adaptername> (D.M.YYYY) – new adapter at beta (latest) repository
https://github.com/<org>/<repo>

<img src="https://raw.githubusercontent.com/<org>/<repo>/<branch>/admin/<icon-filename>" width="100" height="100" />

<One sentence describing what the adapter does.>
```

### Rules

- **Heading:** short adapter name only — no `ioBroker.` prefix.
- **Date:** use today's date in `D.M.YYYY` format (day and month without leading zeros,
  four-digit year), e.g. `11.10.2026` — never ISO format.
- **Label:** exactly `– new adapter at beta (latest) repository` (en-dash, not hyphen).
- **Link:** the full GitHub URL of the adapter repo, on its own line, no label.
- **Icon:** always `https://raw.githubusercontent.com/…` — never `github.com`. Use the
  default branch name from Step 1.
- **Description:** one sentence, plain prose, no version numbers, no bullet points.
  Present tense, third person (e.g. "Watches … and generates …").

---

## Step 6 – Create the branch and commit

```bash
git checkout master
git pull

git checkout -b <adaptername>-whats-new

git add docs/en/history/history.md
git commit -m "news: add <adaptername> new adapter at beta (latest) entry

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Step 7 – Push to the fork and open the PR

The working copy's `origin` remote points to `ioBroker/ioBroker.docs` (no push access).
Push to the user's fork instead:

```bash
# Add the fork remote if it is not present yet
git remote add fork https://github.com/mcm1957/ioBroker.docs.git

git push -u fork <adaptername>-whats-new
```

Write the PR body to a temp file, then create the PR:

```bash
cat > /tmp/pr_body.md <<'EOF'
Adds a "What's new" entry for ioBroker.<adaptername> to `docs/en/history/history.md` as a
new adapter at the beta (latest) repository. <One sentence about what the adapter does.>
It is listed in sources-dist.json but not yet in sources-dist-stable.json.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF

gh pr create \
  --repo ioBroker/ioBroker.docs \
  --head mcm1957:<adaptername>-whats-new \
  --base master \
  --title "news: add <adaptername> new adapter at beta (latest) entry" \
  --body-file /tmp/pr_body.md
```

---

## Reference example

PR: https://github.com/ioBroker/ioBroker.docs/pull/719  
Adapter: `BenAhrdt/ioBroker.device-monitoring`

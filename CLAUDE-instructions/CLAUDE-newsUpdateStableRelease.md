# Prompt: Add a "What's new" history entry for an ioBroker adapter

## Required inputs (fill these in before using this prompt)

- **Adapter name:** `<adaptername>` (e.g. `shelly`)
- **Version range:** from `<from-version>` up to `<to-version>` (e.g. `11.0.0` to `12.0.0`)

---

## Task

Create a pull request against `ioBroker/ioBroker.docs` (base branch `master`) that adds a
"What's new" entry for ioBroker.`<adaptername>` versions `<from-version>` to `<to-version>`
to `docs/en/history/history.md`.

---

## Step 1 – Gather release information

Fetch the GitHub release notes for every stable release of the adapter between
`<from-version>` and `<to-version>` (inclusive of the final version, exclusive of the
starting version):

```bash
# List all release tags to find the range
gh api repos/<org>/ioBroker.<adaptername>/releases?per_page=50 --jq '.[].tag_name'

# Read the body of the final release
gh release view v<to-version> --repo <org>/ioBroker.<adaptername> \
  --json body,publishedAt --jq '{date: .publishedAt, body: .body}'
```

Find the **release date** of `<to-version>` from `publishedAt`.

Also read the adapter's `io-package.json` (or the repo's README) to get a one-line
description of what the adapter does — use it in the opening sentence of the entry.

---

## Step 2 – Identify the adapter's GitHub owner or org

```bash
gh repo view iobroker-community-adapters/ioBroker.<adaptername> 2>/dev/null \
  || gh repo view ioBroker/ioBroker.<adaptername>
```

Use whichever succeeds; that org is `<org>` for all subsequent commands.

---

## Step 3 – Write the history entry

Insert the new entry **just before** the closing comment block at the very end of
`docs/en/history/history.md`:

```
<!--  ######################## keep these lines at end of file as a reminder ######################## -->
```

### Entry format

```markdown
## <adaptername> (D.M.YYYY) – new stable release
https://github.com/<org>/ioBroker.<adaptername>

<img src="https://raw.githubusercontent.com/<org>/ioBroker.<adaptername>/master/admin/<adaptername>.png" width="100" height="100" />

<One sentence saying what the adapter does and which version range this entry covers.>

**<Theme 1 – e.g. a major new subsystem or UI area>**
- <User-facing description of what changed, written for a non-developer reader.>
- <Another change in the same theme.>

**<Theme 2>**
- …

**Other improvements and fixes**
- <Notable fix, described from the user's perspective — what was wrong and what is better now.>
- …
```

### Date format

Use `D.M.YYYY` (day without leading zero, month without leading zero, four-digit year),
matching the style used throughout the file — e.g. `20.9.2026` not `2026-09-20`.

### Logo URL

Always use `https://raw.githubusercontent.com/…` (never `github.com`).
The image path is usually `master/admin/<adaptername>.png`; check the repo if the branch
or filename differs.

### Writing style rules

- Write **for users**, not developers. Describe what the user can do or see, not how the
  code works.
- Group related changes under a bold heading (e.g. `**Device Manager**`,
  `**New devices**`). Use as many or as few groups as make sense — do not force a fixed
  set.
- **Omit** purely technical items: dependency updates, Node.js / js-controller /
  admin version requirement bumps, TypeScript migrations, CI changes, test additions.
- **Omit** alpha/beta/dev release notes; base the entry only on stable release notes.
- Keep bullet points concise but complete — one idea per bullet, written as a full clause.
- Prefer em-dashes (` — `) over parenthetical asides where possible.
- If a fix corrects something that was clearly wrong from a user's point of view
  (wrong values, failed authentication, lost commands), include it. Pure internal
  clean-ups (log spam reduction, refactors) can be omitted unless they noticeably
  affect reliability.

---

## Step 4 – Create the branch and commit

```bash
# Make sure you are on master and it is up to date
git checkout master
git pull

git checkout -b <adaptername>-v<to-version>-whats-new

git add docs/en/history/history.md
git commit -m "news: add What's new entry for ioBroker.<adaptername> v<to-version>

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Step 5 – Push to the fork and open the PR

The working copy's `origin` remote points to `ioBroker/ioBroker.docs` (no push access).
Push to the user's fork instead:

```bash
# Add the fork remote if it is not present yet
git remote add fork https://github.com/mcm1957/ioBroker.docs.git

git push -u fork <adaptername>-v<to-version>-whats-new
```

Create the PR against `ioBroker/ioBroker.docs` base `master`:

```bash
gh pr create \
  --repo ioBroker/ioBroker.docs \
  --head mcm1957:<adaptername>-v<to-version>-whats-new \
  --base master \
  --title "news: add <adaptername> v<to-version> What's new entry" \
  --body-file <path-to-body-file>
```

Write the PR body to a temp file first (here-strings in PowerShell can be fragile with
`--body`). The body should read like this:

```
Adds a "What's new" entry for ioBroker.<adaptername> to `docs/en/history/history.md`,
summarising the changes from v<from-version> to v<to-version> (released <date>):
<two or three sentences naming the headline topics covered — same level of detail as
the opening paragraph of the entry itself>.

Technical details such as dependency updates and minimum Node.js/js-controller
requirements are intentionally omitted.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

---

## Step 6 – If `gh pr edit` fails with a GraphQL Projects error

Use the REST API directly:

```bash
BODY=$(cat <path-to-body-file>)
gh api --method PATCH repos/ioBroker/ioBroker.docs/pulls/<pr-number> \
  -f title="news: add <adaptername> v<to-version> What's new entry" \
  -f body="$BODY" \
  --jq '{title, html_url}'
```

---

## Reference example

PR: https://github.com/ioBroker/ioBroker.docs/pull/717  
Adapter: `shelly`, versions `11.0.0` → `12.0.0`

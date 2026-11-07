# Report Format — ioBroker Adapter Review

## Filename Convention

Use this naming pattern for the report file:

```
AdapterReview_<adaptername>_<yyyyMMdd>_<hhmmss>.md
```

| Placeholder     | Format                                              | Example         |
|-----------------|-----------------------------------------------------|-----------------|
| `<adaptername>` | Short adapter name, no `ioBroker.` prefix           | `shoppingroute` |
| `<yyyyMMdd>`    | Today's date — 4-digit year, 2-digit month and day  | `20261107`      |
| `<hhmmss>`      | Current local time — 2-digit hour, minute, second   | `143022`        |

Full example: `AdapterReview_shoppingroute_20261107_143022.md`

---

## Delivery

After all three passes are complete, deliver the report in two ways:

### 1. Artifact (rendered HTML)

Publish the report as a Claude artifact so the user can view it rendered, bookmark the URL, and
reference it in future review sessions (future sessions can read it back via the Artifact tool
without the user re-uploading anything).

Build a self-contained HTML page that:
- Renders the full markdown report using `marked.js` from cdnjs
- Sets the page `<title>` to `AdapterReview — <adaptername> <yyyyMMdd>`
- Uses a clean, readable typography (system-ui font stack, max-width ~860px, comfortable padding)
- Supports both light and dark mode via `@media (prefers-color-scheme: dark)`
- Styles GitHub-flavoured checkboxes so `- [x]` and `- [ ]` render as actual tick boxes

After publishing, give the user the artifact link in one sentence so they can share or bookmark it.

### 2. Downloadable `.md` file

Write the report markdown to a local file using the filename convention above, then send it via
`SendUserFile` so the user gets a one-click download with the correct filename preset.

**Do not** auto-save the report to any path on the user's computer.

---

## Report Header

Always start the report with this exact header block (no section heading above it):

```markdown
**Report generated:** YYYY-MM-DD HH:MM:SS
**Adapter:** ioBroker.<adaptername>
**Version reviewed:** <version>
**Repository:** <url> (tag <tag>)
**Previous review:** <filename of previous review, or "none">
**Checklist version:** ioBroker Adapter Checklist V3.5
**Skill version:** 1.3.0
**Reviewer:** Claude Sonnet 4.6 / automated review
```

---

## Section Order

Use exactly these sections in this order:

### 1. Status of Previous Review Findings

Describe which findings from the previous review have been fixed, which remain open, and which are
no longer applicable. If there is no previous review, state that explicitly.

### 2. Full Re-Review Results (Checklist V3.5)

#### ✅ PASS items

List all passed checks from all three passes. A brief label per item is sufficient — no need to
explain why something passed. A passed check is always listed here, never in Observations.

#### 🔴 ACTIONABLE FINDINGS

List all failed checks from all three passes. Use a level-4 heading (`####`) for each finding.
Do not number findings. Include a short description of why the check failed and suggest a fix
where applicable.

#### ⚠️ OBSERVATIONS *(optional)*

Additional remarks that are noteworthy but not outright failures. Do not classify observations as
blocking or non-blocking — this is done manually. Do not number observations.

---

## Formatting Rules

**Checklist items:**
- Passed: `- [x] <check label>`
- Needs action: `- [ ] <check label>`

**Problem consolidation:** If the same problem appears in many places (e.g., 25 log messages in
German), write one finding and include up to 5 representative references (file + line number). Do
not list every occurrence.

**What NOT to include:**
- No Summary section
- No summary table
- No Overall Assessment section
- No merge / no-merge recommendation

**Borderline items:** If a potential issue turns out to be acceptable after a second look, do not
list it as a finding. Add it to **Observations** instead.

**Section markers:** ✅, 🔴, and ⚠️ are used only on the section headings, not on individual items.

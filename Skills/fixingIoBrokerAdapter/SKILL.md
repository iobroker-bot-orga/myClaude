---
name: fixingIoBrokerAdapter
description: Use this skill to fix, resolve, or implement a change for an ioBroker adapter. Triggers on prompts like "fix issue <URL>", "fix <GitHub issue link>", "resolve issue", "implement fix for ioBroker", or any bug report or change request for an ioBroker adapter repository. Covers branch creation, code fix, changelog, unit tests, PR creation, and issue comment.
version: 1.3.0
---

# Fixing an ioBroker Adapter Issue

> **Skill version:** 1.3.0

## 1. Understand the Task

- If a GitHub **issue URL or number** is provided:
  - Read the issue title and body carefully.
  - Read **all comments** in order — later comments may refine, clarify, or override the original description.
  - Note any log excerpts, stack traces, or reproduction steps attached to the issue.
- If **no issue is provided**, treat the user's prompt as the full specification.
- **Any instruction given directly in the prompt takes higher priority** than information found inside a GitHub issue.
- If anything is unclear or ambiguous, **ask before proceeding** — do not guess.

## 2. Analyse the Problem

- Read the relevant source files to understand the current behaviour.
- Identify the root cause. Consider edge cases and related code paths.
- Keep the analysis focused; avoid re-reading files you have already understood.
- If the fix could be done in multiple ways, briefly note the trade-offs but **do not implement anything until you have a clear plan**.

## 3. Plan the Fix

- Determine the **minimal code change** that resolves the issue. Do not clean up, optimise, or refactor anything beyond what is necessary.
- If you spot an opportunity to simplify or make the code more robust, **note it as a suggestion** but do not implement it unless the user explicitly confirms.
- Decide whether a unit test is appropriate:
  - **Add a unit test** when the fix addresses non-trivial logic (e.g. a calculation, state machine, data transformation, or conditional branch).
  - **Skip a unit test** for trivial changes (e.g. log-message wording, simple value conversion, cosmetic adjustments).

## 4. Prepare the Branch

- **Never commit directly to `main` or `master`.**
- Determine the default branch name (`main` or `master`) by inspecting the repository.
- Create a new branch based on the current HEAD of that default branch.
  - Suggested naming: `fix/<short-description>` or `fix/issue-<number>` when an issue number is available.
- If you are asked to make additional corrections **after a PR has already been merged**, create a **new branch and new PR** rather than re-opening the old one.

## 5. Implement the Fix

- Make only the changes required to solve the problem.
- Do not reformat, reorganise, or touch unrelated code.
- If a unit test is appropriate (see step 3), add it in the same commit or as a follow-up commit on the same branch.

### Coding Standards

- **Language**: All code, comments, variable names, and identifiers must be written in **English**.
- **Log messages**: All log output (debug, info, warn, error) must be in **English**.
- **i18n / translations**: When adding or changing text in i18n files or i18n objects (e.g. `i18n/`, `admin/i18n/`, inline translation objects), provide translations for **all languages supported by ioBroker**. The standard set is:
  `en`, `de`, `ru`, `pt`, `nl`, `fr`, `it`, `es`, `pl`, `uk`, `zh-cn`.
  - Write the English text first, then add the remaining languages.
  - If a translation cannot be provided for a language, use the English text as a fallback rather than leaving the key empty.

## 6. Update the Changelog

- Open `README.md` and locate the changelog section.
- Find the `### **WORK_IN_PROGRESS**` heading.
  - If it does not exist, **create it at the top of the changelog** (above all existing entries).
- Add exactly **one line** under that heading in this format:
  ```
  - (Author) <user-focused description of what changed>
  ```
  - Replace `Author` with the GitHub username of whoever is credited (the issue reporter, the fixer, or as instructed).
  - Keep the description short and user-focused (what the user sees or experiences, not internal implementation details).
- **Do not** place the entry inside a comment block or a `WORK_IN_PROGRESS` template comment — it must appear as a real list item directly under the heading.

## 7. Commit and Push

- Write a clear, concise commit message that describes the change (e.g. `fix: correct calculation of X in Y`).
- Push the branch to the remote.

## 8. Create the Pull Request

The PR description must contain all three of the following sections:

```
## Problem
<Concise description of the bug or requirement as observed by the user.>

## Analysis
<What was found in the code and why it caused the problem.>

## Solution
<What was changed and why this resolves the problem.>
```

- **Do not** include any text that would automatically close the issue when the PR is merged (e.g. "Closes #123", "Fixes #123", "Resolves #123"). Issues must always be closed manually.
- If an issue was provided, **link the PR to the issue** (e.g. "Related to #123") without using a closing keyword.

## 9. Comment on the Issue (if an issue was provided)

Add a comment to the GitHub issue containing:
- A brief summary of the root cause.
- A brief description of the fix.
- A link to the newly created PR.

## Quick-Reference Rules

| Rule | Requirement |
|------|-------------|
| Branch target | Always branch off `main`/`master` HEAD |
| Direct commits to main/master | **Never** |
| Additional corrections after PR merged | New branch + new PR |
| Code changes | Minimal — no cleanup/optimisation unless instructed |
| Suggestions | Welcome, but not implemented without explicit confirmation |
| Uncertainty | Always ask, never guess |
| Changelog location | Under `### **WORK_IN_PROGRESS**` in `README.md` |
| Closing keywords in PR | **Forbidden** |
| Issue closure | Manual only |
| Unit tests | Add for non-trivial logic; skip for trivial changes |
| Code language | English only |
| Log messages | English only |
| i18n entries | All ioBroker languages (`en de ru pt nl fr it es pl uk zh-cn`) |

---

## Skill Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.3.0 | 2026-11-08 | Improved description to reliably trigger on "fix issue <URL>" prompts |
| 1.2.0 | 2026-11-08 | Added version field and skill changelog |
| 1.1.0 | 2026-11-08 | Added coding standards (English-only code/logs, full i18n language set) |
| 1.0.0 | 2026-11-08 | Initial version |

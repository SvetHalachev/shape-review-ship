---
name: audit
description: Review code that already exists - a feature, a package, a set of files - with the same two passes as the review loop, and write the proven findings into a GitHub issue by severity, ready to fix through the flow in small PRs. Use when asked to audit, review or check an existing feature or area rather than a PR.
argument-hint: "[feature, folder or files]"
---

# Audit an existing feature

Review first, for code that's already there. The result is an issue the user can read, approve and work through, not a PR.

## 1. Agree what's being audited

Name it in a line: the feature or area, the files or folders it lives in, and what it promises (from its docs, types, tests and the issues that built it). If the user named only a feature, find its files first and confirm the list in one line.

Large areas get split into audits of their own (one per package or concern), each its own issue, so every list stays short enough to fix.

## 2. Run both passes at once

Send both in the same message. Neither sees the other's results or your reasoning.

**Pass 1, independent reviewer.** A fresh general-purpose agent, with this prompt filled in:

```
Audit <feature> in <repo path>: <files or folders>.
<How to run the project's checks and tests, from its CLAUDE.md.>

What it promises: <its docs, types, the issues that built it>.

Find real defects: inputs it doesn't handle, promises it doesn't keep,
data it can lose or corrupt, errors that throw instead of being reported,
security problems, tests that pass without proving their claim, and work
that costs far more than it needs to.
For each suspected defect, write a small repro test or script and run it.
Put repro files only in <a scratch folder outside the repo>; don't edit
tracked files, commit, push, post to GitHub, or run slow end-to-end suites.

Report confirmed defects with repro output, ranked High, Medium or Low,
then briefly what you checked that was fine. Skip style preferences.
```

**Pass 2, `/code-review`.** At high effort, locally, on the same files:

```
/code-review high <files or folders>
```

Never `ultra` (cloud, billed), and never `--comment`. If `code-review` isn't available, run a second fresh agent with a reading brief: correctness bugs, simpler or existing ways to do the same thing, needless cost.

Don't edit the working tree while they run.

## 3. Check and sort the findings

- Keep only what's proven by a repro or by reading the code. Re-check anything only one pass reported, and anything that seems surprising.
- Drop style preferences and anything the project has already decided. Say in a line what was dropped and why.
- Rank each finding: **High** loses data, breaks pages or leaks something; **Medium** gives wrong results in real use; **Low** is rare input, wording or cost.
- Group findings that share a cause, so one fix closes them.

## 4. Write the issue

One issue per audit, titled `Audit: <area>`, with:

- one line on what was audited and when;
- the findings as checkboxes under **High**, **Medium** and **Low**, each saying what goes wrong and how to see it, in a sentence or two;
- what was checked and found fine, briefly, so it isn't audited again.

Use the project's labels and writing rules. Open it in the user's browser and give the link.

## 5. Fix it through the flow

Ask which findings to fix first. Then hand over to `shape` (or `flow`): it writes a plan of small PRs, often one per severity, shapes the first, and the usual loop runs from there. Tick each finding on the issue as its PR merges.

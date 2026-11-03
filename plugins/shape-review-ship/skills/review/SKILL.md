---
name: review
description: Review a draft PR in a loop until it's clean - two independent passes (a fresh reviewer that proves defects with repros, and /code-review), triage, fixes with failing tests first, and re-review of only what changed. Use after `build` opens a draft PR, or when asked to review, re-review or "run the review loop" on a PR.
---

# Review loop

The PR stays a draft through every round, so CI doesn't run. The loop ends when a round finds nothing worth fixing, or when it's time to step back (below).

## 1. Run both passes at once

Send both in the same message so they run in parallel. Neither sees the other's results or your reasoning.

**Pass 1, independent reviewer.** Spawn a fresh general-purpose agent with only the shape and the diff. Fill in this prompt:

```
Review PR #<n> in <repo path> (branch <branch>, against <base>).
<How to run the project's checks and tests, from its CLAUDE.md.>

The agreed behaviour is the shape in issue #<n> (gh issue view <n>).
Read the diff with: git diff <base>...HEAD

[Re-review rounds only]
These commits changed: <one line per commit>.
Settled, not up for review: <decisions made in earlier rounds>.

Look for real defects: <the risky areas of this change - inputs, edge cases,
other write paths, interactions with existing code, tests that pass without
proving their claim>.
For each suspected defect, write a small repro test or script and run it.
Put repro files only in <a scratch folder outside the repo>; don't edit
tracked files, commit, push, post to GitHub, or run slow end-to-end suites.

Report confirmed defects with repro output, ranked by severity,
then briefly what you checked that was fine. Skip style preferences.
```

**Pass 2, `/code-review`.** Invoke the `code-review` skill at high effort on the PR, or on the commit range in a re-review round, with the settled decisions in brackets:

```
/code-review high <PR number>
/code-review high <from>..<to> (<settled decisions>; report only defects)
```

Always `high`, which runs locally. Never `ultra`: that runs a billed, multi-agent review in the cloud, and only the user starts it. Don't pass `--comment` either: findings come back to this session, not as PR comments.

If `code-review` isn't available (older Claude Code), run a second fresh agent instead, with the same shape and diff and a different brief: read the diff for correctness bugs, simpler or existing ways to do the same thing, and needless cost. Its findings don't need repros.

**While they run, don't edit the working tree.** The reviewer reads and runs it. Wait for both before changing code.

## 2. Triage every finding

| Finding | Do |
|---|---|
| Proven defect in this PR | Failing test first, watch it fail for the right reason, fix, gated commit. |
| A test that passes without proving its claim | Fix the test until a mutation of the code would fail it. Check by breaking the code once. |
| Real problem outside the shape | Add it to the issue it belongs to, with what was found. Don't widen the PR. |
| Bug in code another person or team owns | Open an issue with their label, the repro and the API to use. Don't fix their code. |
| Low value, or rests on a premise that doesn't hold | Reject with a one-line reason the user can check ("nothing has launched, so no stored data predates this"). |
| Design question | Decide it, record it as settled, and say so in the next round's prompt. Only defects count after that. |

When both passes report the same thing, that's strong evidence. When they disagree, check it yourself.

Before fixing, look for an existing mechanism that already covers the case. A second check that duplicates an existing one is a finding against you.

## 3. Fix

- One commit per fix, each with its test, gated on the project's checks (see `build`).
- Push when the round's fixes are in.
- Keep the shape in the issue and the PR body true: if a fix changes what was agreed, update both and name the change in the demo.

## 4. Re-review only what changed

- Any commit that changes logic gets both passes again, scoped to the new commits (`git diff <last reviewed>..HEAD`).
- A test-only commit that closes a finding doesn't need another round.
- Tell both passes what's settled. Without that, rounds start arguing about decisions instead of finding defects.

## 5. Step back when the fixes can't converge

Some problems can't be fixed by patching, because the approach, the tool or the scope is wrong. More rounds only move the bug. Check for these signs before every round:

- **New cases in the same area two rounds in a row,** each fix opening the next.
- **Fixes add special cases:** another `if` for one more input, option or combination, rather than a rule that covers them.
- **A fix undoes or re-adds an earlier one,** or two findings pull in opposite directions.
- **You're working around a tool:** a library's locking, caching, compression or timing that keeps surprising you, and each fix adds more code around it.
- **The input space can't be listed:** every option combination, every character, every engine. The reviewers will always find one more.
- **The tests get elaborate** just to set up the case, or a test keeps passing for the wrong reason.
- **A check duplicates one that already exists,** or is in a place that can't see everything it needs.

When you see one, **stop patching** and offer the user a change of approach, with your recommendation:

| Change | When |
|---|---|
| **Narrow the scope or the API** | Fewer options, fewer inputs, so each one can be right. A small API that's always correct beats a wide one that's mostly correct. |
| **Replace the tool** | The tool fights the use case. Name the replacement and what it costs. |
| **Use an existing mechanism** | Something already handles the case; move the logic there. |
| **Move it to its own issue** | It's a real problem, but bigger than this PR. Write down what was found. |
| **Accept and document a limit** | The remaining cases are rare and the cost of handling them is high. Say exactly what isn't handled. |

Ask before continuing. Once the user decides, update the shape in the issue and the PR body, record the decision as settled for the next round, and say in the demo what changed and why.

Also stop and suggest finishing when a round only finds misuse the types already forbid, or inputs no real caller sends. Say which findings you'd leave and why.

## 6. Done

When a round comes back clean, or only with findings you've rejected with reasons, summarise for the user:
- what each round found and fixed;
- what was rejected and why;
- anything moved to other issues.

Then hand over to `ship`.

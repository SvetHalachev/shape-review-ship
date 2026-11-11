---
name: review
description: Review a draft PR in a loop until it's clean - two independent passes in the first round (a fresh reviewer that proves defects with repros, and /code-review), then the reviewer alone for at most two more rounds (with /code-review only when a fix restructures code), triage, fixes with failing tests first, and an early step back when fixes don't converge. Use after `build` opens a draft PR, or when asked to review, re-review or "run the review loop" on a PR.
---

# Review loop

The PR stays a draft through every round, so CI doesn't run. The loop ends when a round finds nothing worth fixing, or when it's time to step back (below).

Reviews are the costliest part of the flow. Spend them where they find defects: both passes on the first round, then only as much as each fix needs.

## 0. Before every round: check your own work

Most wasted rounds come from slips a reviewer shouldn't have to find. Before sending anything to review:

- **The commit happened.** `git log -1` shows it and `git status` is clean. If the gated command stopped, nothing after it in the same command should have run; check, and don't send reviewers at an old commit.
- **The shape still holds.** Compare the diff with the approved shape. If a fix changed what it promised (a new restriction, a dropped behaviour, a stricter API), stop and ask the user before reviewing. Never rewrite the shape as "decided in review" on your own.
- **The new tests prove something.** Break the code each test covers, once, and see it fail. Put it back.
- **Comments sit on what they describe,** and nothing inserted between a comment and its code.
- **The PR body is true** for what's committed now.

## 1. Round 1: both passes, at once

Send both in the same message so they run in parallel. Neither sees the other's results or your reasoning.

**Pass 1, independent reviewer.** Spawn a fresh general-purpose agent with only the shape and the diff. Fill in this prompt:

```
Review PR #<n> in <repo path> (branch <branch>, against <base>).
<How to run the project's checks and tests, from its CLAUDE.md.>

The agreed behaviour is the shape in issue #<n> (gh issue view <n>).
Read the diff with: git diff <base>...HEAD

Look for real defects: <the risky areas of this change - inputs, edge cases,
other write paths, interactions with existing code, tests that pass without
proving their claim>.
For each suspected defect, write a small repro and run it. Stop once the
risky areas are covered; report at most the 5 most severe defects.

Rules for your repros:
- Put files only in <a scratch folder outside the repo>.
- Don't edit tracked files, commit, push, post to GitHub, or run slow
  end-to-end suites.
- Don't check out other branches in the real checkout, and don't copy files
  into tracked folders.
- Never run the package manager inside a copy of the repo: a copy whose
  dependencies link to the real ones can prune them. Run copies with the
  language's own runtime only.
- Drop any scratch database you create.

Report confirmed defects with repro output, ranked by severity, then in a few
lines what you checked that was fine. No style points, no cleanups.
```

**Pass 2, `/code-review`.** Invoke the `code-review` skill at high effort on the PR:

```
/code-review high <PR number> --max-findings default
```

`--max-findings` stays set once given, so round 1 resets it to the default. Always `high`, which runs locally. Never `ultra`: that runs a billed, multi-agent review in the cloud, and only the user starts it. Don't pass `--comment`: findings come back to this session, not as PR comments.

If `code-review` isn't available (older Claude Code), run a second fresh agent instead, with the same shape and diff and a different brief: read the diff for correctness bugs, simpler or existing ways to do the same thing, and needless cost. Its findings don't need repros.

**While they run, don't edit the working tree.** The reviewer reads and runs it. Wait for both before changing code.

The two passes find different things: the reviewer finds what only running shows (crashes, timings, broken inputs); `/code-review` finds design problems and simpler ways. Both are worth it once.

## 2. Triage every finding

| Finding | Do |
|---|---|
| Proven defect in this PR | Failing test first, watch it fail for the right reason, fix, gated commit. |
| A test that passes without proving its claim | Fix the test until a mutation of the code would fail it. Check by breaking the code once. |
| Real problem outside the shape | Add it to the issue it belongs to, with what was found. Don't widen the PR. |
| Bug in code another person or team owns | Open an issue with their label, the repro and the API to use. Don't fix their code. |
| Low value, a cleanup, or rests on a premise that doesn't hold | Reject with a one-line reason the user can check ("nothing has launched, so no stored data predates this"). |
| One case of a pattern (the same kind of gap as an earlier finding, or one you can see more of) | Write the rule that covers the pattern, then fix with it. A fix for one instance tends to open the next finding. |
| Design question inside the approved shape | Decide it, record it as settled, and say so in the next round's prompt. Only defects count after that. |
| A fix that would change the approved shape | Stop and ask the user, with a recommendation, before building it. |

When both passes report the same thing, that's strong evidence. When they disagree, check it yourself.

Before fixing, look for an existing mechanism that already covers the case. A second check that duplicates an existing one is a finding against you.

## 3. Fix

- One commit per fix, each with its test, gated on the project's checks (see `build`).
- Push when the round's fixes are in.
- Keep the shape in the issue and the PR body true. If a fix changes what was agreed, the user decides first (see the table), then update both and name the change in the demo.

## 4. Later rounds: one pass, sized to the fix

After round 1, review only the new commits (`git diff <last reviewed>..HEAD`), and only as much as each needs:

| The new commits | Review |
|---|---|
| Only tests, comments, names, or a revert to the base's behaviour | None. Say why in the summary. |
| A fix to what an earlier round reviewed | The independent reviewer alone, scoped to the new commits, with repros. It may run on a smaller model (`model: "sonnet"`) when it only checks earlier fixes. |
| A fix that restructures code, or a new approach | The independent reviewer, and `/code-review high <from>..<to> (<settled decisions>; report only defects) --max-findings 5`. |

After round 1 the two passes mostly find the same defects, and `/code-review`'s extra findings are cleanups; the reviewer's repros are the better use of a round.

At most two rounds after round 1. After round 3, small fixes ship without another round; anything structural goes to the user as a decision.

Tell every pass what's settled. Without that, rounds argue about decisions instead of finding defects.

## 5. Step back early when fixes don't converge

Some problems can't be fixed by patching, because the approach, the tool or the scope is wrong. More rounds only move the bug. **Check these signs after round 2, and stop as soon as one shows:**

- **New cases in the same area as the round before,** each fix opening the next.
- **Fixes add special cases:** another `if` for one more input, option or combination, rather than a rule that covers them.
- **A fix undoes or re-adds an earlier one,** or two findings pull in opposite directions.
- **You're working around a tool:** a library's or database's locking, caching, compression, parsing or ordering that keeps surprising you.
- **The input space can't be listed:** every option combination, every character, every number, every engine. The reviewers will always find one more.
- **The tests get elaborate** just to set up the case, or a test keeps passing for the wrong reason.
- **A check duplicates one that already exists,** or is in a place that can't see everything it needs.

When you see one, **stop patching** and offer the user a change of approach, with your recommendation:

| Change | When |
|---|---|
| **Narrow the scope or the API** | Fewer options, fewer inputs, so each one can be right. A small API that's always correct beats a wide one that's mostly correct. |
| **Contain instead of enumerate** | The input space can't be listed: let the tool that knows the rules refuse it (a savepoint, a parser's own error), and handle that refusal, instead of predicting every case. |
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

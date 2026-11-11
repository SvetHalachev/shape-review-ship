# shape-review-ship

Claude Code skills that take an issue from an agreed shape to a merged PR. Run the whole thing with `flow`, or each step on its own:

| Skill | Does | Waits for |
|---|---|---|
| `flow` | Runs the four below in order, picking up wherever the work is. Loops the reviews until a round is clean, and stops to ask after three rounds or as soon as fixes keep opening new cases | your two approvals |
| `audit` | Reviews code that already exists with the same two passes, and writes the proven findings into an issue by severity, ready to fix through the flow | you to pick what to fix first |

The four steps:

| Skill | Does | Waits for |
|---|---|---|
| `shape` | Writes what the change looks like, what's in and out of scope, the tests, and, for security work, who it guards against, into the GitHub issue, and opens it for you | you to say "shape approved" |
| `build` | Builds it in small commits, failing test first, each commit gated on the project's checks, and opens a draft PR | nothing; hands over to `review` |
| `review` | Reviews the PR in a loop: round 1 runs a fresh reviewer that proves defects with repros and `/code-review` in parallel; later rounds run the reviewer alone (`/code-review` too only when a fix restructures code), at most two of them; a finding that's one case of a pattern is fixed by its rule; fixes come with failing tests first; steps back from round 2 when fixes keep creating new edge cases | a clean round |
| `ship` | Marks the PR ready so CI runs once, gives you the demo with any differences from the shape, and rebase-merges | you to say "approve" |

The skills work on any project. They read its check commands and rules from its own `CLAUDE.md`.

## How it runs

```mermaid
flowchart TD
    existing([Existing feature]) --> audit[audit: both passes on the code,<br/>findings into an issue by severity]
    audit --> issue
    issue([Issue]) --> shape[shape: write the shape into the issue]
    shape --> approve1{You: shape approved?}
    approve1 -- changes --> shape
    approve1 -- yes --> build[build: failing test first,<br/>commits gated on the checks]
    build --> draft[Draft PR, no CI]
    draft --> passes

    subgraph loop [review loop]
        passes[Round 1: two passes in parallel,<br/>fresh reviewer with repros + /code-review high.<br/>Later rounds: the reviewer alone, at most two]
        passes --> triage{Triage each finding}
        triage -- proven defect --> fix[Failing test, fix, gated commit]
        triage -- out of scope --> elsewhere[Its own issue]
        triage -- not real --> reject[Reject with a reason]
        fix --> converging{Converging?}
        converging -- yes --> passes
    end

    converging -- "no: same area again, special cases,<br/>inputs that can't be listed, fighting a tool,<br/>3 rounds" --> stepback{You: change approach?<br/>narrow, contain, replace the tool,<br/>use what exists, split, accept a limit}
    stepback --> shape
    triage -- clean round --> ready[ship: ready, CI runs once]
    ready --> demo[Demo, with any change from the shape]
    demo --> approve2{You: approve?}
    approve2 -- yes --> merge([Rebase-merge, tick the issue])
    approve2 -- changes --> fix
```

`flow` runs all of it. Each box with "You" is where it stops and waits.

## Requirements

- **Claude Code.** Both reviewers come with it: the independent reviewer is its built-in general-purpose subagent, and the second pass is its `/code-review` skill. If your version has no `/code-review`, `review` runs a second fresh reviewer instead. Nothing else to install.
- **The GitHub CLI (`gh`), signed in** (`gh auth login`). The skills use it to read and edit issues, and to open, ready and merge PRs.
- **A `CLAUDE.md` in your project** that says how to run its checks.

## Install

In Claude Code:

```
/plugin marketplace add SvetHalachev/shape-review-ship
/plugin install shape-review-ship@shape-review-ship
```

Or from a terminal:

```sh
claude plugin marketplace add SvetHalachev/shape-review-ship
claude plugin install shape-review-ship@shape-review-ship
```

## Use

Ask in plain words ("shape issue 59", "run the review loop on PR 88"), or call a skill directly:

```
/shape-review-ship:flow 59      # the whole thing, from wherever issue 59 is
/shape-review-ship:audit packages/core/src/fields   # review first, for code that exists
/shape-review-ship:shape 59
/shape-review-ship:build
/shape-review-ship:review 88
/shape-review-ship:ship 88
```

Give your project a `CLAUDE.md` that says how to run its checks (lint, types, tests, end-to-end), so `build` can gate commits on them and `review` can tell the reviewer how to run things.

`review` uses the `code-review` skill that comes with Claude Code.

## Updates

Every commit to this repo is a new version. Claude Code doesn't update plugins from GitHub on its own unless you turn it on:

- **Automatic:** run `/plugin`, open **Marketplaces**, pick `shape-review-ship`, and choose **Enable auto-update**. New versions arrive in the background; run `/reload-plugins` or restart to use them.
- **By hand:**
  ```sh
  claude plugin marketplace update shape-review-ship
  claude plugin update shape-review-ship@shape-review-ship
  ```
  then `/reload-plugins` in a running session.

## The flow, in short

1. **Shape** it in the issue. Nothing is built before it's approved.
2. **Build** it test-first, with every commit gated on the checks.
3. **Open a draft PR**, so CI doesn't run during review.
4. **Check your own work**, then **review** it twice in parallel, and triage: fix what's proven, reject what isn't with a reason, and move what's out of scope to its own issue.
5. **Re-review** only the new commits with the reviewer alone, telling it what's settled; `/code-review` joins only when a fix restructures code. Tests-only commits need none. After round 3, small fixes ship without another round.
6. **Step back** as soon as fixes keep opening new cases (checked from round 2): change the shape instead of patching again.
7. **Ship**: ready, CI once, demo, approve, merge.

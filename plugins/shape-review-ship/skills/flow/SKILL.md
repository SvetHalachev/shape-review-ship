---
name: flow
description: Run the whole flow for an issue - shape it, build it, loop the reviews until a round comes back clean, ship it - picking up wherever the work already is, and stopping only for the shape and merge approvals. Use when asked to "run the flow", "take issue N all the way", or to carry on with an issue or PR in progress.
argument-hint: "[issue or PR number]"
---

# Full flow

Runs `shape`, `build`, `review` and `ship` in order, by invoking each skill. Between the two approvals it keeps going without being asked.

## 1. Find where the work is

Given an issue or PR number (or the current branch), work out the stage and start there:

| What you find | Start at |
|---|---|
| An existing feature or area to check, with no issue yet | `audit`, then ask which findings to fix first |
| An audit issue with open findings | `shape`, for the next PR on its plan |
| An issue with no `Shape` section | `shape` |
| A shape the user hasn't approved in this conversation or on the issue | ask for approval, as `shape` does |
| An approved shape, no branch or PR | `build` |
| A draft PR, not yet reviewed, or with commits after the last review | `review` |
| A draft PR whose last round came back clean | `ship` |
| A ready PR | `ship`, from the demo |

Say in one line what you found and where you're starting.

## 2. Run it

1. **`shape`**, then **stop** until the user says "shape approved" (or asks for changes, which you make and show again).
2. **`build`** through to the draft PR.
3. **`review`**: both passes on round 1, then one pass sized to each fix, until a round comes back clean or with only findings rejected with reasons. Don't ask between rounds; fix, push and re-review on your own.
4. **`ship`**: ready, CI, demo, then **stop** until the user says "approve". Then merge.

While it runs, post a one-line update at each stage change ("PR #88 open, review round 1 running"), not a report each time.

## 3. When to stop and ask

Stop the loop and ask the user, with a recommendation, when:

- **The fixes aren't converging,** checked from round 2: any sign in `review`'s step-back list, such as new cases in the same area as the round before, fixes that add special cases, inputs that can't be listed, or work-arounds for a tool. Propose a change of approach (narrow it, contain instead of enumerate, replace the tool, use what exists, move it to its own issue, or accept a documented limit), not another round.
- **Three review rounds** have run without a clean one, even if each looks small. Summarise what each round found, and say whether the findings are converging.
- A fix would **change the approved shape** in any way the user would notice: a new restriction, a dropped behaviour, a stricter API, a different default. Ask before building it, and never rewrite the shape as decided on your own.
- Something needs a **force-push**, touches code another person or team owns, or would spend paid resources (a cloud review, extra CI runs).
- CI fails for a reason that isn't in the change.

Otherwise, keep going.

## 4. Finish

After the merge, tick the item on the issue and say what's next on it, or offer the next issue.

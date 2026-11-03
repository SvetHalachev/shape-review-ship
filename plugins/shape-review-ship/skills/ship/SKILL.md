---
name: ship
description: Ship a reviewed PR - mark it ready so CI runs once, give the demo with any differences from the shape, open it for approval, and rebase-merge once approved. Use after the review loop comes back clean, or when asked to ship, demo or merge a PR.
---

# Ship

## 1. Mark it ready

Only when every review round is done:

```sh
gh pr ready <n>
gh pr checks <n> --watch
```

CI runs once. If it fails, fix with a failing test first and push. A logic fix goes back through `review` for the new commits.

## 2. Demo

Tell the user, briefly:

- **What it does,** with the example code or commands from the shape, as they work now.
- **How to try it:** the exact commands, and what they'll see.
- **Every difference from the approved shape,** and why. If there are none, say so.
- **What review found:** the real defects fixed, what was rejected and why, and anything moved to other issues.
- **Anything in the process worth knowing,** such as a force-push the user approved or a slip in a commit message.

Open the PR in the user's browser (`gh pr view <n> --web`) and put the link in the message.

## 3. Merge on approval

Wait for "approve". Then:

```sh
gh pr merge <n> --rebase --delete-branch
git checkout <default branch> && git pull --ff-only
```

Use the project's merge style if it names one. Tick the item on the issue with the PR number, and note where anything left over lives. Then offer the next issue, with `shape`.

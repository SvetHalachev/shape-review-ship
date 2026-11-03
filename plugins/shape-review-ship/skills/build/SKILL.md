---
name: build
description: Build an approved shape in small commits - failing test first, every commit gated on the project's checks - and open it as a draft PR. Use after a shape is approved, or when asked to build or implement a shaped issue.
---

# Build

Only after the user has approved the shape.

## 1. Branch

From an up-to-date default branch: `<type>/<short-slug>`, using the commit types below. Follow the project's own branch rule if it has one.

## 2. Failing test first, for every behaviour

1. Write the test for one behaviour from the shape.
2. Run it and watch it fail **for the right reason**: a missing check or a wrong value, not a typo or a setup error. Fix the test if it fails for anything else.
3. Make it pass with the smallest change that does.
4. Repeat for the next behaviour.

A test that would pass without the change proves nothing. When unsure, break the code once and check that the test fails.

## 3. Gate every commit on the checks

Find the project's checks in CLAUDE.md, package.json, a Makefile or CI config (lint, types, unit tests, and end-to-end tests when the change can reach them). Run them and commit in **one** command, starting with `set -e`, so a red run can't be committed:

```sh
set -e
<the project's full check command>
<end-to-end tests, when relevant>
git add <files> && git commit -m "<type>(<scope>): <summary>"
```

- One logical change per commit, files named explicitly. Never `git add -A`.
- Conventional Commits: `feat`, `fix`, `perf`, `refactor`, `test`, `chore`, `ci`, `docs`. Use the type of that commit's change.
- A failed check stops the commit. Read the failure, fix it, and run the same command again.

## 4. Open a draft PR

```sh
git push -u origin <branch>
gh pr create --draft --title "<what it does>" --body "<body>"
```

The body is a few sentences: what it does, any decision that isn't obvious, what was tested and what wasn't, and the issue it's part of.

It stays a draft through review, so CI doesn't run on every round. Then run `review`.

## Rules while building

- No force-push. If a rebase is needed, ask first and use `--force-with-lease=<branch>:<old sha>`.
- Don't edit code another person or team owns. Open an issue for them instead.
- New ideas go into issues, not into this PR.
- Follow the project's rules for commits, comments and what may appear in the repo.

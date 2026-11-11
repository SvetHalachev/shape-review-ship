---
name: shape
description: Shape an issue before any code - write what the change looks like to the person using it, what's in and out of scope, and the tests that will prove it, into the GitHub issue, then open it for approval. Use when starting work on an issue, or when asked to shape, plan or scope one.
---

# Shape

Nothing is built until the shape is approved. The point is to check "is this what I imagined" cheaply, before code exists.

## 1. Read before writing

- The issue, its comments and anything it links to.
- The project's CLAUDE.md, contributing notes and existing patterns in the code the change touches.
- Check the issue's premise against the code. If it's already done, already covered by another mechanism, or rests on something untrue, say so instead of shaping it.

## 2. Try the risky choice first

When the shape rests on how a tool behaves (a query plan, an index, a database's limits, a parser, a library's ordering or locking), check it with a small experiment before writing the shape: an `EXPLAIN` on realistic data, a timing, a few edge inputs through the real tool. Five minutes here is cheaper than a review round finding it after the build. Put what you measured in the shape.

## 3. Split if it's big

An issue with many items gets lettered PRs (A, B, C), grouped by severity or by area, each small enough to review in one sitting. Write the plan as a short "Plan" section, then shape only the first.

## 4. Write the shape into the issue

Add a section `## Shape` (or `## Shape A`) to the issue body with:

- **What changes, as the person using it sees it.** Real example code or commands, real messages, before and after. No internal design unless it's the point.
- **Decisions the reader should check.** A default value, a limit, a name, a behaviour change. Make each one visible so it can be challenged.
- **Who it guards against,** for a change that touches security: who could misuse it, what they can already do (a database admin, another site on the same server, a signed-in user), and what's out of scope. Reviews then check a stated model instead of discovering one, a round at a time.
- **Out of scope.** What this deliberately doesn't do, and where that lives instead.
- **Tests.** The cases that will prove it, including the failure cases.

Keep it short. Use the project's writing rules for issues.

## 5. Ask for approval

- Open the issue in the user's browser (`gh issue view <n> --web`) and put the link in the message.
- Summarise the shape in a few lines and name the decisions you'd like checked.
- Wait for "shape approved", or for changes. Don't start building on silence.

If the user raises a new idea while shaping, add it to the right issue or open a new one. Don't fold it into this shape unless they ask.

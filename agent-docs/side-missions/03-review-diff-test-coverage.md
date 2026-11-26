---
kind: improvement    # bug | feature | improvement | idea
status: open         # open | planned | done | rejected
created: 2026-11-26
updated: 2026-11-26
plans: []
---

# review-diff: check that new logic has tests

## Summary
Neither of `review-diff`'s two reviewers checks whether new logic in the diff comes with tests. The Standards reviewer looks at written standards, consistency, code smells and size; the Plan reviewer looks at acceptance criteria and scope. Untested new code passes both.

## Why it matters
`implement-change` builds test-first, but code written outside a plan (by hand, or a quick fix) has no such guard, and `review-diff` is the one place that sees all of it before a merge.

## What we know
- `skills/review-diff/SKILL.md`, step 4, defines the two reviewers; Decision 7 of plan `01-own-skill-workflow` lists what each checks, and tests are not on either list.
- Reviewers are read-and-search only (they don't run tests), decided while building stage 04 of that plan, so a check would look for test files and test functions that cover the new code, not measure coverage.

## Options considered
- Add "new logic without a test" to the Standards reviewer: smallest change; still open.
- A third reviewer for tests: more separation, more cost per review; still open.

## Open questions
- What counts as "new logic" that needs a test (a new branch, a new function, a new script)?

## Resume here
Read step 4 of `skills/review-diff/SKILL.md`, then draft one line for the Standards reviewer that flags new functions or branches without a test touching them, and dry-run it on a repo with one tested and one untested new function.

## History
- 2026-11-26: created, found while building stage 04 of `01-own-skill-workflow` (review question Q2).

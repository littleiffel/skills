---
status: parked
created: 2026-11-24
updated: 2026-11-24
plans: []
---

# Parallel stage workers

## Idea
Let `implement-change` fan out stages that don't block each other to Orca worker agents (the `orchestration` skill), each working in its own worktree and branch, and then collect their results.

## Why it matters
A plan with independent stages could finish in one go, not one session per stage. Each stage's `blocked-by` list already shows which stages can run in parallel.

## What we know
- `plan-change` keeps stage statuses only in the stage files, so branches working on different stages in parallel don't conflict on the overview (`skills/plan-change/SKILL.md`, "Stages").
- The `orchestration` skill (`~/.agents/skills/orchestration`) already runs Orca workers with task graphs and worktrees.
- Version 1 of `implement-change` runs one stage per run and makes one commit per stage. Until this concept is built, you can run parallel stages by starting sessions in separate worktrees yourself.

## Options considered
- Orca workers through the `orchestration` skill: still open.
- Running sessions in separate worktrees by hand: the stopgap for now.

## Open questions
- How does a worker claim a stage, so that two workers never pick the same one?
- In what order are the stage branches merged, and who resolves conflicts?
- Does each worker run its own reviewer, or does the supervisor review them all?

## Resume here
Once `implement-change` version 1 has been used on a real plan, read the `orchestration` skill and sketch how a plan's `blocked-by` graph would map onto its task graph.

## History
- 2026-11-24: created, split off while designing `implement-change` (grilling Q6).

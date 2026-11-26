---
kind: improvement    # bug | feature | improvement | idea
status: open         # open | planned | done | rejected
created: 2026-11-25
updated: 2026-11-25
plans: []
---

# Move concept files in other repos to side missions

## Summary
Other repos still keep concept files in `agent-docs/concepts/`, linked from their plans. Once plan `01-own-skill-workflow` is merged and the skill links are switched, plan-change and side-mission no longer read that folder, so move those files to `agent-docs/side-missions/` in each repo.

## Why it matters
Until they're moved, planning in those repos won't find the earlier analysis in their concepts, and new side missions would start numbering from `01` next to the old folder.

## What we know
Found on 2026-11-25:
- `~/code/weo/airflow-dags`: `agent-docs/concepts/01-run-reports.md` (status `planned`), linked from 4 files in `agent-docs/plans/01-lst-result-report/`.
- Its worktree `~/orca/workspaces/airflow-dags/add-qualtiy-metrics-to-hyperview-sharpening-pipe`: `01-run-reports.md` and `02-hyperview-pydms-quality-single-dag.md`, linked from plans `01-lst-result-report` and `02-hyperview-quality-report`.
- Its worktree `~/orca/workspaces/airflow-dags/tubeworm` also has an `agent-docs/concepts/` folder.
- `~/orca/workspaces/geo-operations/flatworm`: `01-gdal-vsigs-sandbox-auth.md` (status `parked`), linked from 2 files in `agent-docs/plans/01-agent-sandbox/`.

Steps per repo: `git mv` each file to `agent-docs/side-missions/`, map `exploring` and `parked` to `open` (`planned` and `rejected` stay), add `kind`, rename `## Idea` to `## Summary`, add a History line, and change `../../concepts/` to `../../side-missions/` in the plans that link them. Worktrees pick the move up when they merge the branch that holds it.

## Options considered
- Teach the skills to also read old `agent-docs/concepts/` folders: rejected while planning `01-own-skill-workflow`, because it adds permanent text for a one-off move.

## Open questions
- Nothing yet.

## Resume here
After `01-own-skill-workflow` is merged and the links are switched, run `grep -rl 'concepts/' agent-docs/` in each repo above, then do the steps in "What we know", one commit per repo.

## History
- 2026-11-25: created, split off while planning `01-own-skill-workflow`.

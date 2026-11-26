---
status: done         # todo | in-progress | done | cancelled
blocked-by: []
---

# 01 – Side missions replace concepts

Part of [Own skill workflow](00-overview.md).

## What to build
Replace the `concept` skill with a `side-mission` skill, move the existing concept over, and point every other skill and doc at side missions.

**The skill.** `git mv skills/concept skills/side-mission`, then rewrite `skills/side-mission/SKILL.md`:
- `name: side-mission`. The description covers capturing, resuming, listing and closing side missions, with trigger phrases such as "note this bug for later", "park this idea", "add a side mission", "resume side mission 01-parallel-stage-workers", "what side missions do we have", and says plan-change, implement-change and review-diff use it to park findings. Not for planning committed work: that is plan-change.
- Intro: a side mission is work found along the way (a bug, a feature, an improvement, an idea) that the current context doesn't need to solve. The glossary holds terms, ADRs hold settled decisions, side missions hold what is not yet settled or not yet done.
- Files: `agent-docs/side-missions/NN-slug.md`, next free number after the highest. Always refer to a side mission by its full `NN-slug`: two worktrees can pick the same number, and different slugs keep them apart. Never renumber or delete a file.
- Frontmatter: `kind: bug | feature | improvement | idea`, `status: open | planned | done | rejected`, `created`, `updated`, `plans: []`.
- Statuses: `open` (nothing scheduled, or analysis ongoing; replaces exploring and parked), `planned` (all of it is covered by plans), `done` (solved directly without a plan; the commit goes in History), `rejected` (reason in History).
- Template: the concept template with `## Idea` renamed to `## Summary`. "Summary" and "Resume here" must be filled; for a small bug, "Resume here" is the reproduction or fix step.
- Capture, resume, list and close work as they do in `concept` today, with the new statuses and `kind`. List shows `NN-slug`, kind, status, updated and the summary in one line, `open` first. When another skill invokes it to park something, it writes the file without asking, uses the `kind` the caller passes (or infers one if none is passed), and adds a History line saying where it came from. Closing as `rejected` asks for the reason when the user hasn't given one.
- If the analysis shows the side mission is ready to build, suggest `/plan-change from side mission NN-slug`.

**Move concept 01.** `git mv agent-docs/concepts/01-parallel-stage-workers.md agent-docs/side-missions/01-parallel-stage-workers.md`; set `kind: idea`, `status: open`, rename `## Idea` to `## Summary`, replace "Until this concept is built" with "Until this side mission is built", add to its Options that Orca workers through the `orchestration` skill conflict with ADR 0003 (no calls to other skill packs), so that option needs the ADR reopened or an own worker skill, and add the History line `2026-11-25: moved from concept 01`. `agent-docs/concepts/` is then gone.

**Write side mission 02.** `agent-docs/side-missions/02-move-concepts-in-other-repos.md`, `kind: improvement`, `status: open`: after this plan is merged and the links are switched, move `agent-docs/concepts/*` to `agent-docs/side-missions/` in `~/code/weo/airflow-dags` (1 concept, linked from 4 plan files; its worktree `add-qualtiy-metrics-to-hyperview-sharpening-pipe` has 2, `tubeworm` has the folder too) and `~/orca/workspaces/geo-operations/flatworm` (1 concept, linked from 2 plan files): `git mv`, map `exploring`/`parked` to `open`, add `kind`, rename `## Idea` to `## Summary`, fix `../../concepts/` links in plans. Resume here: start with `grep -rl 'concepts/' agent-docs/` in each repo. History: split off while planning `01-own-skill-workflow`.

**plan-change** (`skills/plan-change/SKILL.md`): replace every concept mention (lines 3, 27, 34, 62, 64, 70–72, 103, 105, 122, 126 at the time of planning) with side missions:
- description: out-of-scope findings are parked as side missions in `agent-docs/side-missions/`;
- step 1: list open side missions; planning "from side mission NN-slug" seeds the grilling the way a concept did;
- step 2: parking offers a side mission, with a recommended `kind` that is passed on to `side-mission`;
- step 3: invoke `side-mission` for each parked item and for the built-from side mission (`plans:` list, History, `planned` only if fully covered);
- step 6 and templates: `Built from [side mission NN-slug](../../side-missions/NN-slug.md)` and `Out (non-goals)` links.

**implement-change** (`skills/implement-change/SKILL.md`): step 4 "Ideas outside the stage" parks a side mission via `side-mission`; step 7's summary and commit list side missions instead of concepts.

**Docs.**
- `GLOSSARY.md`: replace the **Concept** entry with **Side mission** (work found along the way that the current context doesn't need to solve: a bug, feature, improvement or idea), `_Avoid_: concept, backlog item, ticket`.
- `docs/adr/0001-one-stage-per-run.md`: the concept 01 link and text point at `../../agent-docs/side-missions/01-parallel-stage-workers.md`.
- `CLAUDE.md`: under "Issue tracker", add one line saying that tracker is for Matt's skills only, and this repo's own skills keep plans and side missions in files (`docs/adr/0002-side-missions-are-files.md`).
- New `docs/adr/0002-side-missions-are-files.md`: side missions (like plans) are files, not issues. Considered: issues in the repo's tracker (Orca Tasks visibility, collision-free numbers), rejected because every repo would need a tracker configured; one GitHub issue per plan stage, rejected because files and issues would drift. Consequences: Orca Tasks stays empty; numbers may repeat across worktrees, so references use the full `NN-slug`.

## Acceptance criteria
- [x] `skills/concept/` no longer exists; `skills/side-mission/SKILL.md` exists with `name: side-mission` and the statuses, kinds, file location and template above.
- [x] `agent-docs/concepts/` no longer exists; `agent-docs/side-missions/01-parallel-stage-workers.md` and `02-move-concepts-in-other-repos.md` exist with valid frontmatter and filled Summary and Resume here.
- [x] `grep -rni concept skills/ docs/adr/ GLOSSARY.md agent-docs/side-missions/` only matches the glossary's _Avoid_ list, ADR 0002, side mission `02-move-concepts-in-other-repos` (which is about moving concept files), and History lines about the move.
- [x] `CLAUDE.md` has the line about the issue tracker being for Matt's skills only.
- [x] ADR 0002 exists, and the link in ADR 0001 resolves to an existing file.
- [x] Dry run (capture): from a scripted conversation that mentions a bug, the skill writes `NN-slug.md` with the next free number, `kind: bug`, `status: open`, Summary and Resume here filled, without asking questions when invoked by another skill.
- [x] Dry run (list and close): listing shows `open` first; closing as `done` records the commit in History; closing as `rejected` without a reason makes the skill ask for one, then records it.
- [x] The independence check and description-length check from the overview pass.

## How to verify
- Run the `grep` and `ls` checks above.
- Dry run, following the overview's protocol, in a throwaway repo with one commit and an existing `agent-docs/side-missions/01-foo.md`. User brief: capture a bug as if invoked by plan-change with `kind: bug`, capture an idea directly, list, close the bug as done with commit `abc1234`, then ask to reject the idea without giving a reason (the reason, given only when asked, is "superseded"). Inspect the resulting files.
- Read plan-change and implement-change once through for leftover concept wording.

## Notes
- Decisions 2, 3, 12 and 13.
- Don't touch the links in `~/.claude/skills/`: they point at the main checkout, and the installed `concept` must keep working there until this branch is merged.
- From `agent-docs/plans/NN-slug/00-overview.md`, `../../side-missions/` resolves to `agent-docs/side-missions/`, the same depth as the old concepts link.

## Deviations
- Overview's dry-run protocol says `git init -b main` → used `git init` then `git symbolic-ref HEAD refs/heads/main` (why: git 2.25.1 on this machine has no `-b`; stage 04's criterion names the same command).
- `skills/side-mission/SKILL.md` got four clarifications beyond "What to build": it offers a commit unless the calling skill commits the file itself; it says where the History origin comes from (the caller); closing as `done` replaces "Resume here"; every close sets `updated:`. "Resume" also allows fixing a small one directly and closing it as `done` (why: gaps the dry run found; the `done` status needs a path to reach it).
- `skills/plan-change/SKILL.md` line 62 now names "a bug, feature, improvement or idea" instead of "an idea", and the example in step 1 reads "plan from side mission 04-s1-fusion" (why: reviewer, to match the side-mission skill).
- Plan said to write side mission `02-drop-matt-skills` and number the concept move `03` → no `02-drop-matt-skills`; the concept move is `02-move-concepts-in-other-repos` (why: the user removes Matt's skills by hand; re-plan logged in the overview's review log).
- Side mission 01 links `docs/adr/0003-no-dependencies-on-other-skill-packs.md`, which stage 02 creates (forward link; the file name matches stage 02's spec).

## Result
Verified 2026-11-26.
- `ls skills/ agent-docs/`: `skills/concept/` and `agent-docs/concepts/` are gone; `skills/side-mission/SKILL.md` and side missions `01-parallel-stage-workers` and `02-move-concepts-in-other-repos` exist, each with `kind`, `status: open`, a filled Summary and Resume here.
- `grep -rni concept skills/ docs/adr/ GLOSSARY.md agent-docs/side-missions/`: matches only the glossary's _Avoid_ line, side mission 01's History line about the move, and side mission `02-move-concepts-in-other-repos` (8 lines). ADR 0002 has none.
- `grep -rn '## Idea\|status: *\(exploring\|parked\)' skills/ agent-docs/side-missions/`: only the migration steps in `02-move-concepts-in-other-repos` (extra check from the review).
- Independence check: no matches. Description lengths: implement-change 721, plan-change 976, side-mission 795 (limit 1024).
- `CLAUDE.md` line 7 has the issue-tracker line; ADR 0002 exists; ADR 0001's link resolves to `agent-docs/side-missions/01-parallel-stage-workers.md`.
- Dry run (fresh subagent, throwaway repo in the session scratchpad, `dry-01`): capture invoked as plan-change with `kind: bug` wrote `02-csv-export-drops-last-row.md` (`kind: bug`, `status: open`, Summary and Resume here filled) and asked no questions; a direct capture wrote `03-satellite-tile-cache.md` (`kind: idea`); the list showed a table with `NN-slug`, kind, status, updated, summary, `open` first (all three were open, so the order check is weak); closing the bug as done added "done, fixed by the user in commit abc1234" to History; rejecting without a reason made it ask "Why are we rejecting 03-satellite-tile-cache? …" and record "superseded by the new tile server". Files read: only the worktree's `skills/side-mission/SKILL.md` and the throwaway repo, nothing under `~/.agents/` or the main checkout.
- Not dry-run tested: the four clarifications in Deviations, added after the dry run.
- Review: 7 findings (fixed: Result written; commit-offer wording so side missions parked by review-diff still get an offer; deviations logged; old-format grep added as evidence; plan-change step 1 example and line 62 wording; raised with the user: none; dropped: none. Noted for later stages: the ADR 0003 forward link, and plan-change's description has 48 characters left before the limit).

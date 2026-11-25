---
status: todo         # todo | in-progress | done | cancelled
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

**Write side mission 02.** `agent-docs/side-missions/02-drop-matt-skills.md`, `kind: improvement`, `status: open`: uninstall Matt's skills (`~/.agents/skills/` and their links in `~/.claude/skills/`), then remove `docs/agents/` and the "Agent skills" section of `CLAUDE.md`. This also ends the trigger overlaps: "grill" with his `grilling`, glossary work with his `domain-modeling`, reviews with his `code-review`. Resume here: list which of his skills are still used and whether this repo's skills cover them. History: split off while planning `01-own-skill-workflow`.

**Write side mission 03.** `agent-docs/side-missions/03-move-concepts-in-other-repos.md`, `kind: improvement`, `status: open`: after this plan is merged and the links are switched, move `agent-docs/concepts/*` to `agent-docs/side-missions/` in `~/code/weo/airflow-dags` (1 concept, linked from 4 plan files; its worktree `add-qualtiy-metrics-to-hyperview-sharpening-pipe` has 2, `tubeworm` has the folder too) and `~/orca/workspaces/geo-operations/flatworm` (1 concept, linked from 2 plan files): `git mv`, map `exploring`/`parked` to `open`, add `kind`, rename `## Idea` to `## Summary`, fix `../../concepts/` links in plans. Resume here: start with `grep -rl 'concepts/' agent-docs/` in each repo. History: split off while planning `01-own-skill-workflow`.

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
- [ ] `skills/concept/` no longer exists; `skills/side-mission/SKILL.md` exists with `name: side-mission` and the statuses, kinds, file location and template above.
- [ ] `agent-docs/concepts/` no longer exists; `agent-docs/side-missions/01-parallel-stage-workers.md`, `02-drop-matt-skills.md` and `03-move-concepts-in-other-repos.md` exist with valid frontmatter and filled Summary and Resume here.
- [ ] `grep -rni concept skills/ docs/adr/ GLOSSARY.md agent-docs/side-missions/` only matches the glossary's _Avoid_ list, ADR 0002, side mission 03 (which is about moving concept files), and History lines about the move.
- [ ] `CLAUDE.md` has the line about the issue tracker being for Matt's skills only.
- [ ] ADR 0002 exists, and the link in ADR 0001 resolves to an existing file.
- [ ] Dry run (capture): from a scripted conversation that mentions a bug, the skill writes `NN-slug.md` with the next free number, `kind: bug`, `status: open`, Summary and Resume here filled, without asking questions when invoked by another skill.
- [ ] Dry run (list and close): listing shows `open` first; closing as `done` records the commit in History; closing as `rejected` without a reason makes the skill ask for one, then records it.
- [ ] The independence check and description-length check from the overview pass.

## How to verify
- Run the `grep` and `ls` checks above.
- Dry run, following the overview's protocol, in a throwaway repo with one commit and an existing `agent-docs/side-missions/01-foo.md`. User brief: capture a bug as if invoked by plan-change with `kind: bug`, capture an idea directly, list, close the bug as done with commit `abc1234`, then ask to reject the idea without giving a reason (the reason, given only when asked, is "superseded"). Inspect the resulting files.
- Read plan-change and implement-change once through for leftover concept wording.

## Notes
- Decisions 2, 3, 12 and 13.
- Don't touch the links in `~/.claude/skills/`: they point at the main checkout, and the installed `concept` must keep working there until this branch is merged.
- From `agent-docs/plans/NN-slug/00-overview.md`, `../../side-missions/` resolves to `agent-docs/side-missions/`, the same depth as the old concepts link.

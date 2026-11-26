---
change: Own skill workflow
status: done         # draft | todo | in-progress | done | cancelled
created: 2026-11-25
---

# Own skill workflow

## Goal
Make this repo's skills independent of Matt Pocock's skill pack while taking over its best ideas. Off-topic findings become side missions instead of concepts, plan-change keeps `GLOSSARY.md` and ADRs up to date while it grills (the way grill-with-docs does), and a new `review-diff` skill reviews a worktree's changes for code standards and plan fit. plan-change also gains two planning rules: test seams and wide refactors.

## Scope
**In:**
- `side-mission` skill replacing `concept`, and the move of concept 01
- `domain-docs` skill owning `GLOSSARY.md` and `docs/adr/`
- plan-change: calls `domain-docs` while grilling, test seams, wide refactors, side missions
- implement-change: side missions, glossary read-only with reporting, suggests `review-diff` after the last stage
- `review-diff` skill
- `GLOSSARY.md`, ADR 0001 link fix, ADR 0002, ADR 0003
- One line in `CLAUDE.md`: its issue tracker is for Matt's skills only

**Out (non-goals):**
- Uninstalling Matt's skills and removing `docs/agents/` and the "Agent skills" section of `CLAUDE.md`: done by hand, outside this plan and without a side mission.
- Plan stages as GitHub issues (Orca Tasks): rejected, recorded in ADR 0002.
- A `tdd` skill: `implement-change` already builds test-first.
- Multi-context glossaries (`GLOSSARY-MAP.md`).
- Changing `implement-change`'s reviewer.
- Moving concept files in other repos (`~/code/weo/airflow-dags` and its worktrees, `geo-operations/flatworm`): parked as side mission [02-move-concepts-in-other-repos](../../side-missions/02-move-concepts-in-other-repos.md), written in stage 01.

## Decisions
1. No data impact: only skill instructions and docs change. (why: nothing produces data here)
2. Side missions replace concepts. One `side-mission` skill, renamed from `concept`, files in `agent-docs/side-missions/NN-slug.md`, always referred to by full `NN-slug`. Frontmatter `kind: bug | feature | improvement | idea` and `status: open | planned | done | rejected`; `open` replaces exploring and parked, `done` means solved directly without a plan (commit in History). Template is the concept template with `## Idea` renamed `## Summary`. Capture, resume, list, close. (why: "concept" read like an ADR; a side mission says the current context doesn't need to solve it)
3. Everything stays in files, no issue tracker, for side missions as well as plans and stages. Orca Tasks stays empty. Recorded as ADR 0002. (why: an issue tracker would make every repo depend on tracker setup)
4. Skills in this repo never call or read skills from Matt's pack or any other pack; borrowed ideas are rewritten in this repo's own words. Recorded as ADR 0003. (why: the user wants to manage and eventually own the whole workflow)
5. New `domain-docs` skill owns the root `GLOSSARY.md` and `docs/adr/NNNN-slug.md`: challenges terms against the glossary, sharpens vague ones, updates the glossary as soon as a term settles, offers an ADR only when a decision is hard to reverse, surprising without context, and the result of a real trade-off, and creates files lazily. Single root glossary only. plan-change calls it while grilling; side-mission calls it when an analysis settles a term. A plan decision that gets an ADR links it from the overview. (why: plan-change should behave like grill-with-docs without depending on it)
6. `implement-change` only reads the glossary and ADRs, uses glossary terms in code, tests and commit messages, and reports missing or conflicting terms in its hand-off instead of editing `GLOSSARY.md`. Its reviewer stays as it is. (why: changing the language is a planning decision)
7. New `review-diff [ref]` skill: ref defaults to `main`; reviews the working tree against the merge-base with the ref, including uncommitted and untracked files; two reviewer subagents in parallel, Standards (documented standards, consistency with surrounding code, own code-smell list, small and readable) and Plan (stage criteria and scope); report only, never edits; small cleanups inside touched code are suggestions, bigger ones are offered as side missions. `implement-change` suggests `/review-diff main` after a plan's last stage. (why: style and smell review across a whole branch, without slowing each stage)
8. plan-change asks for test seams before writing stages (existing seams first, as high as possible, as few as possible) and records them on a `**Test seams:**` line under Quality. Wide refactors are cut expand → migrate in batches → contract; batches that can't keep tests passing alone merge into one stage. (why: borrowed from Matt's to-spec and to-tickets, without his integration branch)
9. plan-change's description keeps "grill me on this". (why: the overlap with Matt's auto-starting `grilling` lasts only until his skills are removed by hand)
10. No `tdd` skill. (why: implement-change already writes the failing test first)
11. Each stage is verified by a dry run plus text searches (see Quality). (why: skills are prose; running them is the only end-to-end check)
12. Concept 01 is moved to side mission 01, `skills/concept` is deleted, the glossary's "Concept" becomes "Side mission", and ADR 0001's link is fixed. The skill links in `~/.claude/skills/` are switched by hand after merge to `main`. Concept files in other repos are moved later (side mission 02-move-concepts-in-other-repos). (why: the links point at the main checkout; the move is a one-off, so the skills don't learn to read old concept folders)
13. `CLAUDE.md` gets one line saying its GitHub issue tracker is for Matt's skills only, and this repo's skills keep plans and side missions in files (ADR 0002). (why: every session here loads it; side mission 02 removes the section later)

## Quality
Affects data: no data involved. Only skill instructions and docs change.

**Test seams:** each skill's `SKILL.md`, exercised by a dry run, plus text searches over `skills/` for what a dry run can't cover. A dry run works like this:
- A fresh subagent follows the skill from its path in this worktree, in a throwaway git repo (`git init -b main`) under the session's scratchpad directory.
- It never uses the Skill tool: the new skills aren't installed until merge, and the installed ones (old `concept`, Matt's skills) would run instead. When a skill says "invoke X", it reads `<worktree>/skills/X/SKILL.md` and follows that.
- It gets a user brief: the facts of the toy scenario, plus "accept recommendations unless they contradict these facts; when a question isn't covered, answer as a careful user would". It never gets the stage's acceptance criteria, so it can't echo them back.
- It reports every file it read and every file it wrote, in order. The implementer checks that no read path is under `~/.agents/` or `/home/thierry/claude-general-env/`, then judges the produced files against the criteria.

**Independence check (every stage):** this search finds nothing. Keep explanations of names (why `review-diff` isn't called something else) in plan notes, not in `SKILL.md` files.

```sh
grep -rnE '`/?(grilling|grill-with-docs|grill-me|domain-modeling|code-review|to-spec|to-tickets|setup-matt-pocock-skills|tdd|implement|implement-spec|codebase-design|diagnosing-bugs|improve-codebase-architecture|orchestration|writing-for-agents|wayfinder|triage|to-questionnaire|prototype|research|wizard|wait-what|retro|pr)`|\.agents/skills' skills/
```

**Description length (every stage):** each `description:` in `skills/*/SKILL.md` stays at or below 1024 characters: `for f in skills/*/SKILL.md; do sed -n 's/^description: //p' $f | wc -c; done`.

## Risks
- Matt's `grilling`, `domain-modeling` and `code-review` start automatically and overlap with plan-change, `domain-docs` and `review-diff` while installed. (mitigation: they write the same formats, so harm is small; typing the skill name avoids a wrong pick; the overlap ends when Matt's skills are removed by hand)
- The links in `~/.claude/skills/` point at `/home/thierry/claude-general-env/skills/` (the main checkout), so the new skills aren't active until merge, and removing the `concept` link early breaks plan-change on `main`. (mitigation: switch links only after merge; whichever stage finishes the plan lists these commands in its hand-off:)
  ```sh
  ln -s /home/thierry/claude-general-env/skills/{side-mission,domain-docs,review-diff} ~/.claude/skills/
  rm ~/.claude/skills/concept
  ```
- While this plan runs, the installed `implement-change` comes from `main` and parks ideas through `concept` into `agent-docs/concepts/`. (mitigation: from stage 02 on, park ideas by following `<worktree>/skills/side-mission/SKILL.md` instead, and log that as a deviation; never recreate `agent-docs/concepts/`)
- Side-mission numbers can repeat across worktrees. (mitigation: always refer by full `NN-slug`; never renumber)
- A scripted dry run misses problems that only show in a real conversation. (mitigation: the first real use after merge is the final test; findings become side missions)
- Stages 01, 02, 03 and 04 all edit `plan-change` or `implement-change`. (mitigation: blockers order them. After 01, stage 04 can run in parallel with 02 or 03. 02 and 04 both add a line to `implement-change`'s step 9, so expect a small merge there; 03 and 04 touch different files)

## Stages
Statuses live only in each stage file's frontmatter. They are not repeated here, so branches working on different stages in parallel don't conflict on this file.

| # | Stage | Blocked by |
|---|-------|------------|
| 01 | [Side missions replace concepts](01-side-missions.md) | – |
| 02 | [domain-docs skill](02-domain-docs.md) | 01 |
| 03 | [plan-change: glossary, test seams, wide refactors](03-plan-change-rules.md) | 01, 02 |
| 04 | [review-diff skill](04-review-diff.md) | 01 |

## Review log
- 2026-11-25 skeptic: 16 findings (applied: dry-run protocol with worktree skill paths, user brief instead of script and ordered list of reads and writes; link commands written into Risks and the last stage's notes; parking note while the installed implement-change is the old one; review-diff writes only side-mission files after a yes; "this concept" wording in the moved file; description-length check; stage 02 reports new terms instead of adding them; wider independence check; parallelism note; write-order check in stage 03; review-diff error paths and plan detection; named steps and "last stage" defined; `kind` passed by the caller; rejected-without-reason test; scratchpad instead of `mktemp -d`. Dropped: subagents may not nest, because stage 04 already allows one subagent to play both reviewers)
- 2026-11-25 engineer: 10 findings plus 3 notes (applied: same dry-run, link-command, wording, parallelism, independence-check and step-3 findings as above; plan-change's "confirm before you write any files" becomes "any plan files"; `git init -b main`; side mission 01 notes its conflict with ADR 0003; no backticked `code-review` in `SKILL.md`. Not adopted: fast-forwarding `main` after stage 01, because Decision 12 switches the links after the merge, and the parking note covers the meantime. Commit the plan before stage 01: covered at hand-off)
- 2026-11-25 user: concept files in other repos are parked as side mission 03 (move after merge); `CLAUDE.md` gets one line in stage 01. Plan approved.
- 2026-11-26 user: removing Matt's skills is done by hand, not tracked as a side mission. Side mission `02-drop-matt-skills` was dropped before stage 01's commit, and `03-move-concepts-in-other-repos` became `02-move-concepts-in-other-repos`. Stage 01's file list and criteria were updated to match (small re-plan; reviewers not rerun).
- 2026-11-26 user: stage 03 keeps "a smaller rename stays one ordinary stage" (fits Decision 8) and accepts the noted limit of the wide-refactor evidence; stage 04's reviewers stay read-and-search only; parked side mission `03-review-diff-test-coverage` and extended `01-parallel-stage-workers` with the missing review suggestion for parallel worktrees. Plan done.

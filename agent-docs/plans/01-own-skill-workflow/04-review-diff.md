---
status: done         # todo | in-progress | done | cancelled
blocked-by: [01]
---

# 04 – review-diff skill

Part of [Own skill workflow](00-overview.md).

## What to build
A `review-diff` skill that reviews everything in the current worktree that differs from a ref, on two separate axes: code standards and fit with the plan.

**`skills/review-diff/SKILL.md`**:
- Description: review the worktree's changes against `main` or a given branch, tag or SHA, for code standards and plan fit; trigger phrases such as "review my changes", "review this branch", "review since <sha>", "review the worktree against main". Report only.
- **1. Pin the ref.** `/review-diff [ref]`, default `main`. Check it with `git rev-parse --verify`; compute `base=$(git merge-base <ref> HEAD)`. The diff is `git diff $base` (working tree, so uncommitted changes count) plus untracked files from `git ls-files --others --exclude-standard`; commits are `git log --oneline $base..HEAD`. A ref that doesn't exist, no merge-base (unrelated histories) or an empty diff stops here with a clear message, before any subagent starts.
- **2. Find the standards.** `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `CODING_STANDARDS.md` and linter or formatter config, plus `GLOSSARY.md` and `docs/adr/` if present.
- **3. Find the plan.** Stage files under `agent-docs/plans/` touched by the diff, commit messages of the form `<plan folder> stage NN:`, or a plan path the user passes. If none, the Plan reviewer is skipped and the report says so.
- **4. Two reviewer subagents in parallel**, fresh context each, told to report and not edit. Each finding has: severity (blocker / major / minor), `file:line`, the problem, a suggested fix.
  - **Standards:** breaches of a documented standard (cite file and rule; any severity); consistency with the surrounding code (naming, structure, error handling, reusing existing helpers instead of adding new ones); the code-smell list in `references/code-smells.md`, labelled "possible <smell>", never a blocker, and overridden by a documented repo standard; small and readable (code that isn't needed, flexibility nobody asked for, anything that could be simpler); glossary terms used as defined. Skip what tooling already enforces. Cleanups are suggested only inside code the diff touches; anything bigger is labelled "side mission".
  - **Plan:** for each stage in the diff: acceptance criteria missing or only partly met, work outside the stage's scope, criteria that look met but are implemented wrongly, deviations not logged. Quote the stage line for each finding.
- **5. Report.** `## Standards` and `## Plan`, kept separate and not re-ranked, then one summary line per axis with its worst finding. Offer to park each "side mission" finding with the `side-mission` skill, as one Q/➡️ round with a recommendation each. The only files it ever writes are side-mission files, and only after a yes; it never edits code or plan files.

**`skills/review-diff/references/code-smells.md`**: this repo's own list, one line each, what it is → how to fix: mysterious name, duplicated code, feature envy, data clumps, primitive obsession, repeated switches, shotgun surgery, divergent change, speculative generality, message chains, middle man, refused bequest, plus dead or unused code.

**`skills/implement-change/SKILL.md`**, step 9: when the plan is now finished (every stage `done` or `cancelled`), suggest `/review-diff main`.

## Acceptance criteria
- [x] `skills/review-diff/SKILL.md` and `references/code-smells.md` exist with the steps above.
- [x] Dry run (main case): in a throwaway repo (`git init -b main`) with `main` and a branch whose commits (message `01-x stage 01: y`) add a plan stage file `agent-docs/plans/01-x/01-y.md` with two criteria, one not met, and the same helper function in two files, plus an uncommitted change and an untracked file with a name inconsistent with its neighbours: `## Standards` reports the duplication as possible duplicated code and the naming inconsistency; `## Plan` finds the stage and reports the unmet criterion; the uncommitted and untracked files are reviewed; the user brief declines every parking offer, and `git status --porcelain` and file hashes are identical before and after.
- [x] Dry run (errors): a ref that doesn't exist and an empty diff each stop before any subagent starts, with a clear message.
- [x] Dry run (no plan): a diff without plan files or stage commits skips the Plan reviewer and says so.
- [x] implement-change's step 9 suggests `/review-diff main` once every stage is `done` or `cancelled`.
- [x] The independence check and description-length check from the overview pass.

## How to verify
- The three dry runs above, following the overview's protocol (the subagent may launch the two reviewer subagents itself, or play both roles in turn if it can't).
- Run the independence check.

## Notes
- Decisions 4 and 7.
- Matt's `code-review` diffs `<ref>...HEAD`, committed work only; this skill includes the working tree on purpose.
- The name `code-review` is taken twice (Matt's skill and Claude Code's built-in `/code-review`), hence `review-diff`.
- Write the smell list in this repo's words; don't copy from `~/.agents/skills/code-review/` (ADR 0003). Don't explain the name choice in `SKILL.md`: a backticked `code-review` there fails the independence check.
- If this is the last stage to finish (every other stage `done` or `cancelled`), list the link commands from the overview's Risks in the hand-off.
- Stage 02 also edits implement-change (steps 3/4 and 9). If 02 and 04 run in parallel, expect a small merge in step 9.

## Deviations
- Overview's dry-run protocol says `git init -b main` → used `git init` then `git checkout -b main` on the unborn branch (why: git 2.25.1 has no `-b`, and this session's sandbox refused `git symbolic-ref`, which stages 01 and 02 used).
- Step 3 keeps only stages that are `in-progress` or `done` (why: a diff that adds a whole plan would otherwise get every `todo` stage's criteria reported as unmet).
- `skills/review-diff/SKILL.md` got clarifications beyond "What to build", from gaps the first dry runs found: the step 1 commands run one at a time with a `<base>` placeholder, and an empty merge-base is checked before `git diff` (an empty `$base` silently diffs against the index); only the Standards reviewer launches when no stage is found; the report starts with one line naming branch, ref, merge-base, commits, uncommitted and untracked files; dropped findings go after the summary lines; the Plan reviewer may quote an overview line (why: each run had to guess there).
- After the review: reviewers read and search but don't run code or tests; the _Avoid_ check applies per term; the empty-diff message no longer claims the worktree "matches" the ref; the parking step says `side-mission` then offers a commit (why: reviewer findings 2–5). After the second round, step 1 also lists `git status --short --branch` for the header line (why: runs A2 and C2 both added exactly this command; not rerun).
- The protocol's read-path check ("nothing under `/home/thierry/claude-general-env/`") → "nothing under it except this worktree" (why: this worktree lives at `/home/thierry/claude-general-env/.claude/worktrees/agent-a786300e9f5a6cbc1`).

## Result
Verified 2026-11-26.
- `skills/review-diff/SKILL.md` has steps 1–5 as specified (pin the ref with the three stops, standards, plan detection, two parallel reviewers with the Standards and Plan checks, report with separate axes and summary lines, parking offer as one Q/➡️ round); `skills/review-diff/references/code-smells.md` lists the 13 smells, one line each, what it is → how to fix, in this repo's words (nothing under `~/.agents/` was read).
- `skills/implement-change/SKILL.md` step 9 has a new bullet after stage 02's: once every stage is `done` or `cancelled`, suggest `/review-diff main`.
- Independence check: no matches. Description lengths: domain-docs 756, implement-change 721, plan-change 976, review-diff 612, side-mission 795 (limit 1024). `grep -rn "code-review\|code review" skills/review-diff/`: no matches.
- Dry runs: a fresh subagent per run, repos built by the implementer in the session scratchpad. First round on the text before the clarifications; second round (A2, B2, C2) on the final text except the `git status --short --branch` line. Files read in all runs: only this worktree's `skills/review-diff/SKILL.md` and `references/code-smells.md`, `skills/side-mission/SKILL.md` in run C, and the throwaway repos.
  - A, main case (`dry-04a`): `main` with `read_scene.py`, `write_scene.py` and a `CLAUDE.md` with two coding rules; branch `feature/scene-ids` with commit `01-x stage 01: y` adding `agent-docs/plans/01-x/00-overview.md` and `01-y.md` (`status: done`, both criteria ticked, the second, "creating `folder` and its parents", not met) and an identical `_scene_id` helper in both modules; uncommitted `overwrite` parameter in `write_scene.py`; untracked `sceneClipper.py` next to snake_case modules. For A2 a second commit added a `todo` stage `02-z.md`. Brief: "types `/review-diff` with no arguments … declines every parking offer". Both reviewers ran as parallel subagents in A and A2. A2's report: `## Standards` "possible duplicated code. `_scene_id` is a word-for-word copy of the one in `read_scene.py:4`" and "the camelCase module name breaks the pattern of the other modules, which are snake_case", plus the uncommitted `overwrite` as possible speculative generality; `## Plan` / `### 01-x stage 01`: blocker "`write_scene` never creates the output folder" quoting the criterion, the uncommitted `overwrite` and the untracked `sceneClipper.py` as outside the stage; stage 02 (`todo`) left out of the Plan brief. One parking offer (A: two), declined. Before and after both A and A2: `git status --porcelain`, HEAD, `main` and the sha256 of every file outside `.git` identical; no stash entry.
  - B, errors (`dry-04b`, branch `feature/empty` at `main`, plus an orphan branch `other`): B2 `/review-diff nosuchbranch` → "There is no branch, tag or commit called `nosuchbranch`. Local branches: `feature/empty` (current), `main`, `other`."; `/review-diff` → "Nothing to review: this worktree has no changes that `main` doesn't have (merge-base `914af34`)."; `/review-diff other` → "`other` and HEAD share no history, so there is no point to diff from." (stopped right after `git merge-base` exited 1). No subagent in any run of B or B2.
  - C, no plan (`dry-04c`: `main` holds a finished plan `01-scene-reader`; branch `feature/clip` adds `clip_scene.py` in commit "Add scene clipping"; brief: "added … by hand, without a plan. They type `/review-diff main`"): only the Standards reviewer launched; `## Plan`: "Skipped: no `in-progress` or `done` plan stage found in the diff, its commits or the arguments." and "Plan: skipped". In C the user accepted one parking offer, so `side-mission` wrote and committed `agent-docs/side-missions/01-scene-dataclass.md`, the only write. C2 ran on that branch, declined parking, and wrote nothing.
- Not covered by a dry run: finding a stage only through its commit message, and the `[plan path]` argument.
- Review: 7 findings (fixed: final text dry-run again as A2, B2, C2; reviewers no longer run code; _Avoid_ check per term; empty-diff wording; side-mission commit offer named; a `todo` stage added to A2 for the status filter; raised with the user: whether reviewers may ever run tests (answer: keep them read-and-search only, revisit after first real use); parked: no reviewer checks whether new logic has tests, as side mission `03-review-diff-test-coverage`, and the step 9 suggestion missing for parallel worktrees, added to side mission `01-parallel-stage-workers`; dropped: step 9 suggestion never firing when the last stages run in parallel worktrees, because a checkout can't see other worktrees' statuses (ADR 0001) and the stage spec asks for exactly this condition).

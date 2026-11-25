---
status: todo         # todo | in-progress | done | cancelled
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
- [ ] `skills/review-diff/SKILL.md` and `references/code-smells.md` exist with the steps above.
- [ ] Dry run (main case): in a throwaway repo (`git init -b main`) with `main` and a branch whose commits (message `01-x stage 01: y`) add a plan stage file `agent-docs/plans/01-x/01-y.md` with two criteria, one not met, and the same helper function in two files, plus an uncommitted change and an untracked file with a name inconsistent with its neighbours: `## Standards` reports the duplication as possible duplicated code and the naming inconsistency; `## Plan` finds the stage and reports the unmet criterion; the uncommitted and untracked files are reviewed; the user brief declines every parking offer, and `git status --porcelain` and file hashes are identical before and after.
- [ ] Dry run (errors): a ref that doesn't exist and an empty diff each stop before any subagent starts, with a clear message.
- [ ] Dry run (no plan): a diff without plan files or stage commits skips the Plan reviewer and says so.
- [ ] implement-change's step 9 suggests `/review-diff main` once every stage is `done` or `cancelled`.
- [ ] The independence check and description-length check from the overview pass.

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

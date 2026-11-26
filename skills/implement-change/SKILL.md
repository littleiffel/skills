---
name: implement-change
description: Implement one stage of an approved plan written by plan-change (agent-docs/plans/NN-slug/). Picks the named stage or the next eligible one, builds it test-first, verifies it against its acceptance criteria and quality checks, has a reviewer subagent check the diff, then sets the stage to done and makes one commit for it once the user says yes. Use whenever the user wants to implement, build, continue or resume a planned change, with phrases like "implement stage 02", "build the next stage of 03-cloud-mask", "continue the plan" or "implement agent-docs/plans/03-cloud-mask-v2". Only for changes that already have a plan in agent-docs/plans/. To plan new work, or when the plan itself has to change, use plan-change.
---

# Implement a change

Build one stage of an approved plan per run, and end with one commit that holds the stage's code, its ticked acceptance criteria and its `status: done`. One stage per run gives the user a checkpoint after every stage, and one commit per stage lets any stage be reviewed or reverted on its own (see `docs/adr/0001-one-stage-per-run.md` in the skills repo).

The plan was written so that implementing it needs no further questions to the user. Its decisions are settled: follow them, don't reopen them. Ask the user only where this skill says to.

Plans live in `agent-docs/plans/NN-slug/`: `00-overview.md` plus numbered stage files. The `plan-change` skill owns their format.

## Process

### 1. Load the plan and pick the stage

The user names a plan (by number, slug or folder name) and optionally a stage: `/implement-change 03-cloud-mask-v2`, or `/implement-change 03-cloud-mask-v2 stage 02`. If the plan is unclear, list `agent-docs/plans/` and ask which one.

Read `00-overview.md` and check its `status`:
- `draft`: stop. The plan hasn't been reviewed and approved yet. Tell the user to finish it with `/plan-change`.
- `cancelled` or `done`: stop and say so.
- `todo` or `in-progress`: continue.

Read the frontmatter of every stage file, then pick the stage:
- **A stage is named.** If it is `done` or `cancelled`, say so and stop. If any of its blockers isn't `done`, stop and list the blockers with their statuses. A `cancelled` blocker counts as not done: the stage may depend on work that will never exist, so point the user to a re-plan with `/plan-change`.
- **No stage is named.** Take the first match from this list:
  1. A stage that is `in-progress` in this checkout: resume it.
  2. The lowest-numbered `todo` stage whose blockers are all `done`.
  3. Nothing is eligible: list each remaining stage with whatever blocks it, then stop.

Whenever this step stops, also name the stages that are eligible now, with the command to start one.

### 2. Check the working tree

Run `git status --porcelain`. The stage's commit must contain only that stage's work:
- **Starting a new stage:** the tree must be clean, and untracked files count. If it isn't clean, list the files, ask the user to commit them or move them out of the way, and end the run; the user starts it again once the tree is clean. Never stash, reset or discard changes yourself.
- **Resuming an `in-progress` stage:** uncommitted changes from the earlier run are expected. Read the diff and the stage file's `## Deviations` and `## Result` sections to see where that run stopped, and continue from there instead of starting over. Files that clearly have nothing to do with the stage get the same treatment as above.

### 3. Start the stage

- Set the stage's `status: in-progress`. If the overview is still `todo`, set it to `in-progress` as well. Apart from this and step 7, don't touch the overview's status: statuses live in the stage files, so parallel branches don't conflict on the overview.
- Read the overview (Decisions, Quality, Risks), the stage file, and the files and modules the stage names. Also read the `## Result` of the stages this one is blocked by: an earlier run may have flagged a problem for this stage, such as reference data that looks wrong. If it is still unresolved, raise it before building. If the repo has a `GLOSSARY.md` or `docs/adr/`, read the parts that touch this stage. For a broad sweep of the codebase, use an Explore subagent so file dumps stay out of your context.

The `in-progress` status stays local to this checkout until the stage's commit lands. When the user runs sessions in parallel in other worktrees, each session names its own stage.

### 4. Build it, test first

For logic and data processing: for each acceptance criterion, write the test that proves it, or find an existing one. Run it and watch it fail for the right reason, on its assertion rather than an import or syntax error. Then write the code until it passes. A test that has never failed on its assertion may not test anything. When resuming code that an earlier run already wrote, check this by running the tests against a deliberately broken version, without changing the working files. For a docs, config or CI stage, where a failing test means nothing, follow the stage's "How to verify" instead.

Stay inside the stage. While you build:

- **Deviations.** If the plan's details don't hold (a file it names doesn't exist, an approach doesn't fit the code), take the smallest path that keeps the plan's decisions, acceptance criteria and quality thresholds intact, and log it in the stage file under `## Deviations`. If no such path exists, because the work would break a decision, a criterion or a threshold, go to step 8 and explain the conflict there, with your recommended fix.
- **Glossary terms.** Name things in code, tests and the commit message with the terms in `GLOSSARY.md`. Don't edit `GLOSSARY.md` or write ADRs: changing the project's language or its settled decisions is a planning decision. Note any term that is missing from it or conflicts with it, for the hand-off.
- **Ideas outside the stage.** When a bug, feature, improvement or idea comes up that isn't part of this stage, ask once, with your recommendation and a recommended `kind`, whether to park it as a side mission. If the user agrees, invoke the `side-mission` skill to write it. It never goes into this stage's diff.

### 5. Verify

- Run every check in the stage's "How to verify", plus the existing tests around the code you touched, so regressions show up.
- Run the quality checks that the stage's "How to verify" names, on the test set, and compare each metric with its threshold in the overview. Run only those: a check meant for a later stage may need code that doesn't exist yet.
- Tick an acceptance criterion `[x]` only when you have evidence for it. Never edit a criterion's text.
- Write the evidence into `## Result` (template below), so a later reader can see why the stage is `done`.

If a check still fails after a real attempt, go to step 8. Never lower a threshold, edit reference data, or weaken, skip or delete a test to reach `done`: changing what counts as done is a re-plan decision, and the user makes it. If the reference data itself looks wrong, go to step 8 and show the evidence: fixing reference data is the user's job, outside this skill.

### 6. Review

Launch one reviewer subagent with a fresh context. Give it the repo root, the plan folder path, the stage file path, and how to see the work (`git diff HEAD` plus untracked files). Don't share your own opinion of the work, so its read stays independent. Tell it to report findings and not edit any files. Ask it to check:

- whether the diff meets each acceptance criterion and the overview decisions that apply to this stage;
- whether the tests really prove the criteria, or would still pass with broken code;
- edge cases and error paths the stage calls for that were missed;
- changes outside the stage's scope, and deviations that weren't logged.

It doesn't do a general style review. Each finding comes back with: severity (blocker / major / minor), file, problem, and a suggested fix.

If you can't launch subagents, do the review yourself, re-reading the diff with that lens.

Then handle the findings:
- Check each one against the code and drop the wrong ones, saying briefly why.
- Fix the obvious ones directly and list them.
- Put the rest to the user as one round of questions, each with your recommended fix:

  ```
  ❓ **Q1 – <title>**: <finding and options>

  ➡️ <your recommended fix>
  ```

- After any fix, run step 5 again and update `## Result`. Don't run the reviewer again: one reviewer per run.
- If a `blocker` finding is still unresolved, the stage can't be `done`. Go to step 8.

Add one line about the review outcome to `## Result`.

### 7. Commit

- Show the user a short summary: the stage, the files changed, the key verification results, deviations, the review outcome, any side missions parked, and the proposed commit message `<plan folder> stage NN: <stage title>`, for example `03-cloud-mask-v2 stage 02: shadow mask`. Questions from step 6 whose answers don't change this stage's work can go in the same message.
- Commit only after the user says yes. Only then set the stage's `status: done`, and the overview's `status: done` if every stage is now `done` or `cancelled`. Until then the stage stays `in-progress`, so if the run ends without a yes, the next run resumes it. Make one commit holding the stage's code and tests, the stage file, the overview if it changed, and any side-mission files parked during this run. If the user wants changes first, make them, run step 5 again, and show the summary again.

### 8. When the stage can't be finished

The stage stays `in-progress` and nothing is committed. The review (step 6) doesn't run on unfinished work. Criteria already ticked stay ticked, because their evidence still holds. Write into `## Result` what was tried, the current numbers, and what still fails or blocks. Then give the user the options, and put any decision they have to make in the Q/➡️ format with your recommendation:
- continue in a new run, which resumes this stage where this one stopped;
- if the reference data is wrong: the user fixes it in a commit of its own, then continues in a new run;
- re-plan with `/plan-change`, if a criterion, threshold or decision turned out to be wrong.

If the user re-plans now, `plan-change`'s commit may include this stage file with its "Not verified" Result. That is fine: the stage's code stays uncommitted and lands in the stage's own commit, with `status: done`.

### 9. Hand off

End with:
- the stage's outcome: `done` with its commit hash, or still `in-progress` and why;
- the stages that can be started or resumed now, with the command to run next, for example `/implement-change 03-cloud-mask-v2`;
- problems spotted in later stages (for example reference data that looks wrong), so the user can fix them before those stages start. Don't edit other stages' files;
- terms that were missing from `GLOSSARY.md` or conflicted with it, so the user can settle them, for example with the `domain-docs` skill.

If several stages are eligible at once, say they can be built in parallel, each in its own worktree with the stage named explicitly. If this stage was the last one, say the plan is done.

## Stage file sections this skill adds

Append these to the stage file, below its existing sections. Leave the plan's own sections as they are, except for ticking acceptance criteria.

```markdown
## Deviations
- <what the plan said> → <what was done instead> (why: <one line>)

## Result
Verified YYYY-MM-DD.
- `<command>`: <outcome, e.g. 14 passed>
- <metric> on <test set>: <value> (threshold <threshold>; current version <value>, if there is one)
- Review: N findings (fixed: ...; raised with the user: ...; dropped: ... because ...)
```

Leave out `## Deviations` when there were none.

If the stage stopped at step 8, start `## Result` with `Not verified YYYY-MM-DD.`, say what is missing, and write `Review: not run`.

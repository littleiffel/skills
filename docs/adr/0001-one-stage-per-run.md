# One stage per run, one commit per stage

`implement-change` builds exactly one stage per run, and the run ends with a single commit holding that stage's code, its ticked acceptance criteria and its `status: done`. We chose this over looping through every remaining stage in one run: each stage gets a human checkpoint (you see the summary and approve the commit), and the git history maps one to one onto stages, so any stage can be reviewed, reverted or bisected on its own.

## Considered Options

- **Loop through all remaining stages in one run**: faster, but context keeps piling up over a long session, and a mistake in an early stage only shows at the end. Rejected for now.
- **Separate commits per stage (a "claim" commit for `in-progress`, then the work)**: other worktrees would see which stage is taken, but stages would no longer map one to one onto commits. Rejected. Claiming is an open question in [side mission 01-parallel-stage-workers](../../agent-docs/side-missions/01-parallel-stage-workers.md).

## Consequences

Until a stage's commit lands, its `in-progress` status is visible only in the checkout working on it. When you run sessions in parallel by hand, each one must name its stage explicitly.

---
name: review-diff
description: Review everything in the current worktree that differs from main or a given branch, tag or SHA (commits, uncommitted changes and untracked files) on two separate axes, code standards and fit with the plan stage the work belongs to, with two reviewer subagents in parallel. Report only; it never edits code or plan files. Use this whenever the user wants their changes reviewed before a merge or a PR, with phrases like "review my changes", "review this branch", "review since <sha>", "review the worktree against main" or "check my diff against the plan". implement-change suggests it after a plan's last stage.
---

# Review a diff

Review all the work in this worktree that the ref doesn't have yet, on two axes that stay apart in the report:

- **Standards**: is the code written the way this repo wants it, and as small and plain as it can be?
- **Plan**: does it do what its plan stage asked for, and nothing else?

`implement-change` already checks each stage against its criteria while building it. This review looks across the whole branch, including style and smells, before it is merged.

This skill reports and never fixes. The only files it may write are side-mission files, and only after the user says yes to parking a finding. It never edits code, tests or plan files.

## Process

### 1. Pin the ref

The user runs `/review-diff [ref] [plan path]`. The ref is a branch, tag or SHA and defaults to `main`. An argument that is a path under `agent-docs/plans/` is the plan path (step 3). Work from the repo's top folder (`git rev-parse --show-toplevel`):

```sh
git rev-parse --verify --quiet "<ref>^{commit}"   # the ref exists
git merge-base <ref> HEAD                          # <base>, where this work branched off
git diff --stat <base>                             # committed and uncommitted changes to tracked files
git ls-files --others --exclude-standard           # untracked files, reviewed too
git log --oneline <base>..HEAD                     # commits since the base
git status --short --branch                        # branch name and uncommitted files, for the report's first line
```

Run them one at a time, in this order, and check each result before the next: an empty `<base>` would turn `git diff <base>` into a diff against the index. `git diff <base>` compares the working tree with the base, so staged and unstaged changes count. Untracked files don't show in it: they are reviewed as whole new files.

Stop here, before any subagent starts, with a clear message when:
- **the ref doesn't exist**: "There is no branch, tag or commit called `<ref>`." and name the local branches;
- **there is no merge-base** (`git merge-base` prints nothing and exits non-zero): "`<ref>` and HEAD share no history, so there is no point to diff from.";
- **the diff is empty**, with no tracked changes and no untracked files: "Nothing to review: this worktree has no changes that `<ref>` doesn't have (merge-base `<short sha>`)."

### 2. Find the standards

Collect the paths of what this repo documents about how code is written:
- `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md` and `CODING_STANDARDS.md`, at the top and in the folders the diff touches;
- linter and formatter config, such as `pyproject.toml`, `setup.cfg`, `ruff.toml`, `.eslintrc*`, `.prettierrc*`, `.editorconfig` or `.pre-commit-config.yaml`;
- `GLOSSARY.md` and `docs/adr/`, if present.

If there are none, the Standards reviewer still runs, on consistency with the surrounding code, the smells and simplicity.

### 3. Find the plan

The stages this work belongs to come from any of:
- the plan path the user passed: a plan folder, or one stage file;
- stage files under `agent-docs/plans/NN-slug/` (not `00-overview.md`) that the diff changes or adds;
- commit messages since the base of the form `<plan folder> stage NN: <title>`, which is how `implement-change` names its commits.

Keep the stages that are `in-progress` or `done`, each with its plan's `00-overview.md`. A `todo` or `cancelled` stage has no work to check yet; a diff that only writes a plan has nothing to review against it.

If no stage is left, skip the Plan reviewer, and say so in the report.

### 4. Two reviewers in parallel

Launch the two reviewer subagents in parallel, each with a fresh context; only Standards when step 3 found no stage. Give each one the repo root, the ref and the base SHA, how to see the work (`git diff <base>` plus the list of untracked files), and the paths for its role. Don't share your own view of the code, so their read stays independent. Tell them to read the code around each change, not only the changed lines, to report findings, and not to edit any files. They read and search but don't run the code or the tests: a run can write files into the repo or reach outside systems. Each finding comes back with: severity (blocker / major / minor), `file:line`, the problem, and a suggested fix.

- **Standards** gets the standards paths from step 2 and the absolute path of `references/code-smells.md` next to this file. It checks:
  - breaches of a documented standard, citing the file and the rule; these can have any severity;
  - consistency with the surrounding code: naming, structure, error handling, and reusing an existing helper instead of adding another that does the same;
  - the smells in `references/code-smells.md`, each labelled "possible <smell>" and never a blocker; a documented standard of the repo overrides the list;
  - small and readable: code that isn't needed, flexibility nobody asked for, anything that could be simpler;
  - terms used as `GLOSSARY.md` defines them, and not called by a word it lists under _Avoid_ for that term.

  It skips what tooling already enforces, such as formatting or lint rules. Cleanups are suggested only inside code the diff touches; anything bigger is labelled "side mission", with a recommended `kind` (bug, feature, improvement or idea).
- **Plan** gets the stage files and overviews from step 3. For each stage it checks:
  - acceptance criteria that are missing or only partly met;
  - criteria that look met, or are ticked, but are implemented wrongly;
  - work outside the stage's "What to build" or the plan's decisions;
  - departures from the stage's written details that aren't logged under its `## Deviations`.

  It quotes the line of the stage file, or of the overview, that each finding is about.

If you can't launch subagents, do both reviews yourself, one after the other, re-reading the diff with each lens in turn, and say so in the report.

### 5. Report

Check each finding against the code. Drop the ones that are wrong, and list them after the summary lines with one line on why. Keep the two axes apart: don't merge the lists or rank one axis against the other.

```markdown
Reviewed `<branch>` against `<ref>` (merge-base `<short sha>`): <N> commits, uncommitted changes in <files>, untracked <files>.

## Standards
- **<severity>** `<file:line>`: <problem>. Fix: <suggested fix>

## Plan
### <plan folder> stage NN
- **<severity>** `<file:line>`: <problem> (stage: "<quoted line>"). Fix: <suggested fix>

Standards: <N> findings, worst: <severity> <one-line problem>.
Plan: <N> findings, worst: <severity> <one-line problem>.
```

When the Plan reviewer was skipped, `## Plan` says "Skipped: no `in-progress` or `done` plan stage found in the diff, its commits or the arguments." and the summary line says "Plan: skipped".

Then offer to park each finding labelled "side mission", as one round of questions:

```
❓ **Q1 – <title>**: <the finding>. Park it as a side mission?

➡️ <park it as kind `<kind>`, or leave it, and why>
```

For each yes, invoke the `side-mission` skill with the finding, its `kind`, and the origin "found while reviewing <branch> against <ref>"; that skill then offers to commit the file. Nothing else is written. Fixing the findings is the user's call, for example in a new `implement-change` run or by hand.

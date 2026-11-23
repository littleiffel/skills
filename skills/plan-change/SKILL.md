---
name: plan-change
description: Plan a software change (a new feature, processing step or data product, a fix, a migration) before any code is written. Grill the user until every decision is settled, including how output quality will be measured when the change affects data, write a staged plan to agent-docs/plans/NN-slug/ (00-overview.md plus numbered stage files with todo/in-progress/done/cancelled status), then have two reviewer subagents (a skeptic and an engineer) critique it before the user approves. Use this whenever the user wants to plan, scope, spec, design or write a PRD for a feature or change, says things like "let's plan X", "grill me on this", "plan this before we build it" or "break this into stages", or describes work too big to just start coding, even if they never say "plan". Not for implementing an existing plan.
---

# Plan a change

Turn an idea into an approved, staged plan that a later session (or another agent) can implement stage by stage without having to ask the user anything again. This skill writes plan files only, never code: implementation happens later, one stage at a time, so the plan has to carry every decision the implementer needs.

This skill says "change" rather than "feature", because in geospatial and Earth-observation work "feature" already means a vector geometry or an ML input variable. Use "change" in the files you write.

## Output layout

```
agent-docs/plans/
└── NN-<slug>/
    ├── 00-overview.md        goal, scope, decisions, quality, stage index, review log
    ├── 01-<stage-slug>.md    status: todo
    ├── 02-<stage-slug>.md    status: todo
    └── ...
```

- `NN` is the next free number after the highest existing folder (start at `01`), zero-padded to two digits. Slugs are short, lowercase, hyphenated.
- Stage files are numbered in the order they should be implemented.
- Statuses live in each file's frontmatter, so anyone (human or agent) can see progress at a glance and the implementation step can pick the next `todo` stage whose blockers are `done`.

## Process

### 1. Gather context

- Read the user's idea. List `agent-docs/plans/` to find the next number and to check whether this change already has a plan. If it does, this is a re-plan: see "Re-running on an existing plan".
- Explore the codebase enough to ground your questions: the modules the change touches, existing patterns it should follow, how tests are run, and what data the affected code reads and writes. For a broad sweep, use an Explore subagent so file dumps stay out of your context.
- Finding facts is your job. Never ask the user something you could look up in the code.

### 2. Grill the user

Interview the user until you share an understanding of the change. Think of it as a decision tree: each decision unlocks the ones that depend on it.

Work in rounds. Each round, ask every question whose prerequisites are already settled. A question that depends on an answer still open this round waits for a later round. Number the questions and give your recommended answer for each, worded so that "yes" accepts it:

```
❓ **Q1 – <title>**: <question, with options if there are several>

➡️ <your recommended answer>

---

❓ **Q2 – <title>**: ...
```

Ground the questions in what you found in the code ("auth lives in `src/auth/`; reuse its session middleware?"). Areas worth covering, where they apply: user-visible behavior, scope and non-goals, data and interfaces, edge cases and errors, how each part gets tested, migration or rollout, and risks. Skip areas that don't apply. This is a conversation, not a form.

**Always ask, in the first round, whether the change affects data.** Does it change what the pipeline or system produces: values, geometry, coverage, format or metadata? Give your recommended answer based on what you found in the code, but let the user decide, because only they know whether, say, a "small refactor" is expected to shift outputs. The answer decides the quality branch:

- **Yes, it affects data.** Quality becomes part of the grilling. Settle which metrics show the output is right, which reference data they are measured against, what threshold is good enough, which small fixed test set (for example a few AOIs and dates covering the hard cases) every stage is checked on, and how results are compared with the current version. For Earth-observation or other geospatial data, read `references/eo-quality.md` for the checklist of what to cover. If no reference or baseline exists yet, recommend that the first stage measures the baseline before anything changes; without one, no later stage can show it made things better or kept them the same.
- **No, outputs should stay the same** (a refactor, a dependency upgrade, a performance change). Ask one question only: how will we prove the outputs are unchanged? Usually by comparing against the current version on the test set.
- **No data involved** (CI, tooling, docs, UI that doesn't touch data). Record "no data impact" as a decision and move on.

Stop when no open questions are left. Summarize the decisions in a short numbered list and ask the user to confirm before you write any files. Writing first and asking later would bake guesses into the plan.

### 3. Draft the plan

Write `00-overview.md` and the stage files using the templates below. The overview starts with `status: draft` until the user approves it.

How to cut stages:
- Each stage is a vertical slice that leaves the system working and testable, rather than a horizontal layer ("all the database work"). A stage that can't be verified on its own will hide bugs until the end.
- Size each stage so one focused agent session can finish it. 2–6 stages is typical. If you need more than about 8, the work is probably several changes, so suggest splitting it.
- Use `blocked-by` for the stages that really must come first, not as an automatic chain. Stages without blockers on each other can be built in parallel.
- Name the existing files and modules the implementer should start from; the engineer reviewer checks they exist. Include code only when it encodes a decision prose can't, such as a schema or a function signature.
- For a change that affects data, each stage's "How to verify" names the quality checks from the overview that prove the stage works, and the test data they run on.

### 4. Review with two subagents

Launch two reviewer subagents in parallel, each with a fresh context. Give them the plan folder path, the repo root and their role, but not your own opinion of the plan, so their read is independent. Tell them to report findings and not to edit any files.

- **Skeptic**: finds gaps. Missing requirements, unhandled edge cases and error paths, acceptance criteria or quality thresholds that are vague or can't be measured, a change that affects data but has a missing or hand-wavy Quality section, contradictions between the overview and the stages, and scope creep beyond the agreed decisions.
- **Engineer**: checks the plan against the actual code. Do the referenced files and modules exist and work the way the plan assumes? Is the stage order right? Can each stage be built and tested on its own? Do the reference and test data the plan relies on exist, and can the code reach them? Are there hidden migrations, breaking changes or callers the plan missed? Does the codebase already have something that makes a step simpler?

Ask each reviewer to return a list of findings, each with: severity (blocker / major / minor), the file it concerns, the problem, and a suggested fix.

If you can't launch subagents, do both reviews yourself one after the other, re-reading the files with each lens in turn.

### 5. Resolve findings with the user

- Check each finding against the code and drop the ones that are wrong. Say briefly which ones you dropped and why.
- Apply obvious minor fixes directly and list them.
- Put the rest to the user as one more grilling round, in the same Q/➡️ format, with your recommended fix for each.
- Update the files with the outcome and add an entry to the overview's review log.

When the user approves, set the overview to `status: todo`.

### 6. Hand off

End with the plan folder path, the stage list with each stage's status and blockers, and how to start implementing, for example: "implement stage 01 of agent-docs/plans/03-cloud-mask-v2".

## Templates

### 00-overview.md

```markdown
---
change: <Change name>
status: draft        # draft | todo | in-progress | done | cancelled
created: YYYY-MM-DD
---

# <Change name>

## Goal
Why this change exists and what it changes for its users. 2–4 sentences.

## Scope
**In:** ...
**Out (non-goals):** ...

## Decisions
1. <decision> (why: <one line>)
2. ...

## Quality
Affects data: yes | no, outputs unchanged | no data involved

If yes:

| Metric | Threshold | Reference data | How it is measured |
|--------|-----------|----------------|--------------------|
| <e.g. cloud-mask F1> | <e.g. ≥ 0.90, and not below current version> | <e.g. 200 labelled scenes> | <script, test or notebook> |

**Test set:** <the fixed AOIs/dates/inputs every stage is checked on, and where they live>
**Compared against:** <current version, a reference product, in-situ data>

If outputs should be unchanged: how that is proven (e.g. pixel-wise diff against the current version on the test set is zero).
If no data is involved: one line saying why.

## Risks
- <risk> (mitigation: ...)

## Stages
The status in each stage file's frontmatter is the source of truth; keep this table in sync with it.

| # | Stage | Status | Blocked by |
|---|-------|--------|------------|
| 01 | [<title>](01-<slug>.md) | todo | – |
| 02 | [<title>](02-<slug>.md) | todo | 01 |

## Review log
- YYYY-MM-DD skeptic: N findings (applied: ..., rejected: ... because ...)
- YYYY-MM-DD engineer: N findings (applied: ..., rejected: ... because ...)
```

### NN-<stage-slug>.md

```markdown
---
status: todo         # todo | in-progress | done | cancelled
blocked-by: []       # stage numbers, e.g. [01, 02]
---

# NN – <Stage title>

Part of [<Change name>](00-overview.md).

## What to build
The end-to-end behavior this slice adds. Name the files and modules to start from.

## Acceptance criteria
- [ ] <observable, testable outcome>
- [ ] ...

## How to verify
The tests to run or write, and any manual check that proves this stage works. For a change that affects data: which quality checks from the overview, on which test data, with which threshold.

## Notes
Decisions from the overview that apply here (e.g. "Decision 3"), and gotchas found while exploring.
```

## Re-running on an existing plan

When the plan folder already exists, update it rather than starting over. Humans or agents may already be implementing it, so the status record is what you must protect:

- Keep every stage's `status`. Leave `done` and `cancelled` stages untouched.
- Update `todo` and `in-progress` stages to match the new decisions.
- Never delete a stage file. If a stage is no longer needed, set it to `cancelled` and add a one-line reason under Notes.
- Give new stages the next free number and express ordering through `blocked-by` instead of renumbering, because other files and commits may already refer to the existing numbers.
- Grill only on what changed (including whether the change now affects data differently), run the two reviewers again, and add the outcome to the review log.

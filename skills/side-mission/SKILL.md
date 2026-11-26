---
name: side-mission
description: Keep work found along the way that the current context doesn't need to solve (a bug, a feature, an improvement or an idea) as side-mission files in agent-docs/side-missions/NN-slug.md (status open, planned, done or rejected), so it survives across sessions and can be picked up later. Use this whenever the user wants to note, park or save something for later, resume or continue the analysis on an earlier side mission, list or review side missions, or close or reject one, with phrases like "note this bug for later", "park this idea", "add a side mission", "resume side mission 01-parallel-stage-workers", "what side missions do we have". Also used by plan-change, implement-change and review-diff to park findings that are out of scope. Not for planning committed work: that is plan-change.
---

# Side missions

A side mission is work found along the way that the current context doesn't need to solve: a bug, a feature, an improvement or an idea. It lives in `agent-docs/side-missions/NN-slug.md`, so it survives the end of a session and can be picked up later by the user, by another session, or by `plan-change` when it becomes real work.

Three kinds of project knowledge outlive a session, each in its own place: `GLOSSARY.md` holds the terms, ADRs in `docs/adr/` hold settled decisions, and side missions hold what is not yet settled or not yet done.

A side mission is useful only if someone who has never seen the original conversation can read it and continue from where it stopped. Write for that reader: concrete findings, numbers, file and data paths, and a clear next step, rather than a transcript of what was said.

Side-mission files belong in git. After capturing, updating or closing one, offer to commit it, unless the skill that invoked this one commits the file itself (`plan-change` and `implement-change` do): an uncommitted file exists only in the current checkout, so other worktrees and sessions would not see it. Large data and analysis outputs stay out of git; link their location from "What we know" instead.

## Kinds

- `bug`: something behaves wrongly.
- `feature`: new behaviour someone wants.
- `improvement`: making existing code, docs or tooling better without new behaviour (a cleanup, a refactor, a faster path).
- `idea`: something worth exploring before anyone knows whether to build it.

## Statuses

- `open`: nothing is scheduled yet, or the analysis is ongoing.
- `planned`: everything in the side mission is now covered by one or more plans.
- `done`: solved directly, without a plan (for example a small bug fixed in one commit). The commit goes in History.
- `rejected`: decided against. The reason goes in History, so it isn't proposed again without new information.

When only part of a side mission has become a plan, it stays `open`, and its `plans:` list shows which plans took which parts.

## Numbering and references

Take the next free number after the highest existing file (start at `01`), zero-padded to two digits. The slug is short, lowercase and hyphenated.

Always refer to a side mission by its full `NN-slug`, for example `01-parallel-stage-workers`, never by its number alone. Two worktrees can each create a side mission with the same number before they merge; the different slugs keep the files apart, and git sees no conflict. Never renumber or delete a side-mission file: plans and other files link to it by name, and rejected ones are a record of what was considered.

## What to do

Work out which of these the user wants. If it's unclear, ask.

### Capture a new side mission

1. List `agent-docs/side-missions/`. If an existing side mission already covers this, add to that one instead of creating a duplicate, and tell the user you did.
2. Pick the number and slug as above, and a `kind`.
3. Write the file from the template below, using what the conversation already contains. Ask only for what you can't infer. Capturing should be quick, because a slow capture means things don't get captured. "Summary" and "Resume here" are the two sections that must be filled; for a small bug, "Resume here" is the reproduction or the fix step. The others can say "Nothing yet".

When another skill (such as `plan-change`, `implement-change` or `review-diff`) invokes this one to park something, write the file straight from the conversation without asking questions, use the `kind` the caller passes (infer one if it passes none), then return to that workflow. Add a History line saying where it came from, using what the caller tells you, e.g. "split off while planning 03-cloud-shadow" or "found while building stage 02 of 03-cloud-shadow".

### Resume a side mission

1. Read the side-mission file and any plans in its `plans:` list.
2. Tell the user in a few lines where it stands: the summary, what is known, what is still open, and the "Resume here" step.
3. Continue the work with the user. Finding facts in the code, data or docs is your job, so use subagents for broad searches instead of asking the user.
4. Write findings into the file as you go, not only at the end. Sessions end without warning and context gets compacted, and anything not written down is lost, which is the very problem side missions exist to solve.
5. Before you stop, update "Resume here" with the next concrete step, update `updated:`, and add a History line summarizing the session.

If the work shows the side mission is ready to build, suggest `/plan-change from side mission NN-slug`. Planning is that skill's job. If it is small enough to fix directly, fix it with the user and close it as `done`.

### List side missions

Read the frontmatter and the "Summary" section of each file, and show a table: `NN-slug`, kind, status, last updated, and the summary in one line. Put `open` first, then `planned`, then the rest.

### Close a side mission

- **Done**: set `status: done`, add a History line with the commit that solved it, and replace "Resume here" with "Nothing left: done in <commit>."
- **Rejected**: set `status: rejected` and add a History line with the reason. If the user hasn't given one, ask for it: a rejection without a reason will be proposed again later.
- **Planned**: set `status: planned` only when its `plans:` list covers all of it. Otherwise, leave it `open` and record in History which part was planned.

In every case, set `updated:` to today.

## Template

```markdown
---
kind: idea           # bug | feature | improvement | idea
status: open         # open | planned | done | rejected
created: YYYY-MM-DD
updated: YYYY-MM-DD
plans: []            # plan folders built from this side mission, e.g. [03-cloud-shadow]
---

# <Side mission name>

## Summary
What it is, in 2–4 sentences.

## Why it matters
The problem it solves or the opportunity, and for whom.

## What we know
Findings so far, each with its evidence: numbers, file paths, data locations, links to notebooks or runs. Date entries when they come from different sessions.

## Options considered
- <option>: <pros, cons, and whether it is still open or ruled out, and why>

## Open questions
- <question> (what would answer it)

## Resume here
The next concrete step for whoever picks this up, specific enough to start without re-reading the whole history.

## History
- YYYY-MM-DD: created (where it came from)
```

---
name: concept
description: Keep ideas and analysis that aren't committed work yet as concept files in agent-docs/concepts/NN-slug.md (status exploring, parked, planned or rejected), so they survive across sessions and can be resumed later. Use this whenever the user wants to park, save or note an idea for later, resume or continue the analysis on an earlier idea or concept, list or review existing concepts, or reject or close one, with phrases like "park this idea", "save this as a concept", "resume concept 04", "where were we on the S1 fusion idea", "what concepts do we have". Also used by plan-change to park out-of-scope ideas. Not for planning committed work: that is plan-change.
---

# Concepts

A concept is an idea or analysis worth keeping that nobody has committed to building yet. It lives in `agent-docs/concepts/NN-slug.md`, so it survives the end of a session and can be picked up later by the user, by another session, or by `plan-change` when part or all of it becomes real work.

The point is continuity. A concept is useful only if someone who has never seen the original conversation can read it and continue from where it stopped. Write for that reader: concrete findings, numbers, file and data paths, and a clear next step, rather than a transcript of what was said.

## Statuses

- `exploring`: being analysed now, or expected to be soon.
- `parked`: the analysis so far is saved, and nothing is scheduled.
- `planned`: everything in the concept is now covered by one or more plans.
- `rejected`: decided against. The reason goes in History, so the idea isn't proposed again without new information.

When only part of a concept has become a plan, it stays `exploring` or `parked`, and its `plans:` list shows which plans took which parts.

## What to do

Work out which of these the user wants. If it's unclear, ask.

### Capture a new concept

1. List `agent-docs/concepts/`. If a concept already covers this idea, add to that one instead of creating a duplicate, and tell the user you did.
2. Take the next free number after the highest existing one (start at `01`), zero-padded to two digits. The slug is short, lowercase and hyphenated.
3. Write the file from the template below, using what the conversation already contains. Ask only for what you can't infer. Capturing should be quick, because a slow capture means ideas don't get captured. "Idea" and "Resume here" are the two sections that must be filled. The others can say "Nothing yet".
4. Recommend `exploring` or `parked`, and use the user's answer.

When another skill (such as `plan-change`) invokes this one to park an idea, write the file straight from the conversation without asking questions, then return to that workflow. Add a History line saying where the idea came from, e.g. "split off while planning 03-cloud-shadow".

### Resume a concept

1. Read the concept file and any plans in its `plans:` list.
2. Tell the user in a few lines where it stands: the idea, what is known, what is still open, and the "Resume here" step.
3. Continue the analysis with the user. Finding facts in the code, data or docs is your job, so use subagents for broad searches instead of asking the user.
4. Write findings into the file as you go, not only at the end. Sessions end without warning and context gets compacted, and anything not written down is lost, which is the very problem concept files exist to solve.
5. Before you stop, update "Resume here" with the next concrete step, update `updated:`, and add a History line summarizing the session.

If the analysis shows the concept is ready to build, suggest `/plan-change from concept NN`. Planning is that skill's job.

### List concepts

Read the frontmatter and the "Idea" section of each file, and show a table: number, name, status, last updated, and the idea in one line. Put `exploring` first, then `parked`, then the rest.

### Close a concept

- **Rejected**: set `status: rejected` and add a History line with the reason. A rejection without a reason will be proposed again later.
- **Planned**: set `status: planned` only when its `plans:` list covers all of it. Otherwise, leave the status alone and record in History which part was planned.

Never delete a concept file, and never renumber one. Rejected concepts are a record of what was considered, and plans and other concepts link to them by number.

## Template

```markdown
---
status: exploring    # exploring | parked | planned | rejected
created: YYYY-MM-DD
updated: YYYY-MM-DD
plans: []            # plan folders built from this concept, e.g. [03-cloud-shadow]
---

# <Concept name>

## Idea
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
- YYYY-MM-DD: created (where the idea came from)
```

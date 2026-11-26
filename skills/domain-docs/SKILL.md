---
name: domain-docs
description: Keep a project's language and settled decisions written down, the terms in GLOSSARY.md at the repo root and the hard-to-reverse decisions in docs/adr/NNNN-slug.md. While a conversation runs, check the words used against the glossary, sharpen vague ones into one term, update GLOSSARY.md as soon as a term is settled, and offer an ADR only for a decision that is hard to reverse, surprising without context and the result of a real trade-off. Use this whenever the user wants to add, change or look up a term ("add X to the glossary", "what do we call X", "update the glossary") or record a decision ("record this decision as an ADR", "write an ADR for this"). Also invoked by plan-change while grilling and by side-mission when an analysis settles a term.
---

# Domain docs

Two files hold what a project has settled and outlive every session: `GLOSSARY.md` holds its words, and `docs/adr/` holds its hard decisions. What is not settled yet belongs in side missions (the `side-mission` skill). This skill keeps the two files current while the conversation that settles them is still going, because a term agreed on and not written down gets argued over again next week.

## Files

- `GLOSSARY.md`, one per repo, in its top folder.
- `docs/adr/NNNN-slug.md`: a four-digit number, the next free one after the highest existing file (start at `0001`), and a short, lowercase, hyphenated slug.

Read the parts of both that touch the topic before you start. Create them only when there is something to write: `GLOSSARY.md` with the first settled term, `docs/adr/` with the first ADR. Until then, don't point out that they are missing or suggest creating them. For a new glossary, take the project name and its one-sentence description from the README or the code; if neither tells you, ask for them in one line.

## During the conversation

- **Hold terms against the glossary.** When a word the user says clashes with what `GLOSSARY.md` defines for it, say so straight away and ask which meaning holds, for example: "The glossary says a scene is one acquisition, but here you seem to mean a mosaic of several. Which one?" Don't write anything about that term until it is settled.
- **One word per thing.** When a word is vague or covers two things, propose one precise term for each and say which words to avoid.
- **Probe with scenarios.** Find where a term ends with concrete cases: "Is a reprocessed scene the same scene, or a new one?"
- **Check the code.** When the user says how something works, check that the code says the same. If it doesn't, show both and ask which one is right. If the code names things with a word the glossary avoids, mention it in one line; renaming code is not this skill's job.
- **Write as soon as it's settled.** A term is settled once the user has answered any boundary question you needed to ask. Update `GLOSSARY.md` the moment that happens, not in a batch at the end, and say so in one line ("Added **Scene** to GLOSSARY.md."). Terms left for the end of a long session often never get written.

## Glossary rules

- Terms and their meanings, nothing more. No implementation details, file names, decisions or plans; those belong in the code, ADRs and plans.
- Only words specific to this project. General programming words (cache, retry, config) stay out, even when the code is full of them.
- One or two sentences per term, saying what it is, not how it is built or used.
- Be opinionated: choose one word, and put the words that lost under `_Avoid_`.
- Once terms start forming clusters, group them under `###` headings; until then, one list will do.

```markdown
# <Project name>

<One or two sentences on what the project does and for whom.>

## Language

### <Group, once there are clusters>

**<Term>**:
<What it is, in one or two sentences.>
_Avoid_: <other words for the same thing>
```

## ADRs

Offer an ADR only when all three hold:

1. **Hard to reverse.** Changing your mind later costs real work.
2. **Surprising without context.** A later reader would wonder why it was done this way, or would "fix" it.
3. **A real trade-off.** Other options were on the table, and this one won for reasons you can state.

If one is missing, skip it: an easy decision just gets reversed, an obvious one needs no record, and one without alternatives has nothing to explain. Examples that pass: building one plan stage per run (this repo's ADR 0001), keeping side missions in files instead of an issue tracker (ADR 0002), storing rasters in a format downstream users can open rather than the fastest one, fixing one grid or projection for every product, or a limit set by a data provider's licence.

Ask before writing one, with your recommendation. When the user asks for an ADR directly, the request counts as the yes, but still apply the three tests: if a decision fails one, say which and why, and write it only if they still want it. Keep it short: a reader months later needs to see that this was chosen on purpose, and for what reason.

```markdown
# <The decision, as a short title>

<One to three sentences: the context, what was decided, and why.>

## Considered Options
- **<option>**: <why it was rejected>

## Consequences
<What follows from the decision that isn't obvious.>
```

Leave out "Considered Options" unless someone would otherwise propose a rejected option again, and "Consequences" unless the effects aren't obvious.

## When another skill invokes this one

`plan-change` invokes this skill while it grills, and `side-mission` while it resumes an analysis. From then on, these rules hold for the rest of that conversation, next to the calling skill's own. The calling skill keeps control of the flow: don't start an interview of your own, and put any question about a term into the calling skill's current round. When it summarizes, give it the terms you added or changed and the ADRs you wrote, so it can list them.

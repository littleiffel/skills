---
status: todo         # todo | in-progress | done | cancelled
blocked-by: [01]
---

# 02 – domain-docs skill

Part of [Own skill workflow](00-overview.md).

## What to build
A `domain-docs` skill that keeps the repo's language and settled decisions in `GLOSSARY.md` and `docs/adr/`, written in this repo's own words.

**`skills/domain-docs/SKILL.md`**:
- Description: keeps `GLOSSARY.md` and ADRs up to date; trigger phrases such as "add X to the glossary", "what do we call X", "update the glossary", "record this decision as an ADR"; used by plan-change while grilling and by side-mission when an analysis settles a term.
- Files it owns: one `GLOSSARY.md` at the repo root and `docs/adr/NNNN-slug.md` (four digits, next free number). Create each lazily, only when there's something to write. If the files are missing, say nothing about it.
- During a conversation:
  - check terms the user uses against the glossary and call out a conflict straight away;
  - turn vague or overloaded words into one precise term;
  - test a term's boundaries with concrete scenarios;
  - check that the code agrees with what the user says, and surface contradictions;
  - update `GLOSSARY.md` as soon as a term is settled, not in a batch at the end, and mention the update in one line.
- Glossary rules: terms only, no implementation details, not a spec; only terms specific to this project, not general programming words; definitions of one or two sentences saying what a term is; pick one word and list the others under `_Avoid_`; group terms under `###` headings when clusters appear. Format as in this repo's `GLOSSARY.md`: `# <Context>`, a one- or two-sentence intro, `## Language`, then `**Term**:` / definition / `_Avoid_:`.
- ADRs: offer one only when the decision is hard to reverse, surprising to a later reader without context, and the result of a real trade-off; ask before writing it. Format: a title and one to three sentences of context, decision and why; optional `## Considered Options` and `## Consequences` only when they add something. Format as in `docs/adr/0001-one-stage-per-run.md`.
- When invoked by another skill, its rules apply for the rest of that skill's conversation; it doesn't take over the conversation's flow.

**`skills/side-mission/SKILL.md`**: in "Resume", when the analysis settles a term or a decision that passes the ADR test, invoke `domain-docs`.

**`skills/implement-change/SKILL.md`** (Decision 6): step 3 already reads `GLOSSARY.md` and `docs/adr/`. Add to step 4 that code, tests and the commit message use the glossary's terms, and to step 9 that terms missing from or conflicting with the glossary are reported, without editing `GLOSSARY.md`.

**`docs/adr/0003-no-dependencies-on-other-skill-packs.md`**: this repo's skills never call or read another pack's skills; borrowed ideas are rewritten here. Considered: calling Matt's `grilling` and `domain-modeling` from plan-change, rejected because his updates would change this workflow and the user wants to own it.

No new glossary terms are expected. If one comes up, report it in the hand-off (Decision 6).

## Acceptance criteria
- [ ] `skills/domain-docs/SKILL.md` exists and covers the files, conversation rules, glossary rules, ADR rules and invoked mode above.
- [ ] side-mission's Resume step invokes `domain-docs`.
- [ ] implement-change uses glossary terms and reports missing or conflicting ones in its hand-off, without editing `GLOSSARY.md`.
- [ ] ADR 0003 exists.
- [ ] Dry run A: in a repo without `GLOSSARY.md`, when the user settles a term ("we call it a scene, not an image"), `GLOSSARY.md` is created in the format above with an `_Avoid_` line, and no `docs/adr/` is created.
- [ ] Dry run B: in a repo whose `GLOSSARY.md` defines a term, the skill calls out a user's conflicting use before writing anything.
- [ ] Dry run C: a decision that is easy to reverse gets no ADR offer; a decision that passes all three tests gets an offer and, on yes, `docs/adr/0001-<slug>.md` (or the next free number).
- [ ] The independence check and description-length check from the overview pass.

## How to verify
- Dry runs A–C, following the overview's protocol, in a throwaway repo with a tiny code file; the user briefs describe the scenarios above without saying what the skill should do. Inspect `GLOSSARY.md` and `docs/adr/` afterwards.
- Read side-mission and implement-change for the new lines.
- Run the independence check.

## Notes
- Decisions 4, 5 and 6.
- Write the rules from scratch: don't copy text from `~/.agents/skills/domain-modeling/` (ADR 0003).
- `GLOSSARY.md` changes in stage 01 too; this stage starts after 01 is done, so they don't conflict.

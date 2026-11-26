---
status: done         # todo | in-progress | done | cancelled
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
- [x] `skills/domain-docs/SKILL.md` exists and covers the files, conversation rules, glossary rules, ADR rules and invoked mode above.
- [x] side-mission's Resume step invokes `domain-docs`.
- [x] implement-change uses glossary terms and reports missing or conflicting ones in its hand-off, without editing `GLOSSARY.md`.
- [x] ADR 0003 exists.
- [x] Dry run A: in a repo without `GLOSSARY.md`, when the user settles a term ("we call it a scene, not an image"), `GLOSSARY.md` is created in the format above with an `_Avoid_` line, and no `docs/adr/` is created.
- [x] Dry run B: in a repo whose `GLOSSARY.md` defines a term, the skill calls out a user's conflicting use before writing anything.
- [x] Dry run C: a decision that is easy to reverse gets no ADR offer; a decision that passes all three tests gets an offer and, on yes, `docs/adr/0001-<slug>.md` (or the next free number).
- [x] The independence check and description-length check from the overview pass.

## How to verify
- Dry runs A–C, following the overview's protocol, in a throwaway repo with a tiny code file; the user briefs describe the scenarios above without saying what the skill should do. Inspect `GLOSSARY.md` and `docs/adr/` afterwards.
- Read side-mission and implement-change for the new lines.
- Run the independence check.

## Notes
- Decisions 4, 5 and 6.
- Write the rules from scratch: don't copy text from `~/.agents/skills/domain-modeling/` (ADR 0003).
- `GLOSSARY.md` changes in stage 01 too; this stage starts after 01 is done, so they don't conflict.

## Deviations
- `skills/domain-docs/SKILL.md` got four clarifications beyond "What to build", from gaps the dry runs found: a new glossary takes its project name and description from the README or code, or asks in one line; a term counts as settled once its boundary questions are answered; code that names things with an avoided word is mentioned, not renamed; a direct request for an ADR still goes through the three tests (why: each run had to guess there).
- After the review, `skills/domain-docs/SKILL.md` was reworded where it shared phrases with the other pack's glossary skill, and its list of typical ADR material was replaced by examples from this repo and EO work (why: Notes ask for own words; a 5-word overlap search found 27 shared phrases).
- `skills/implement-change/SKILL.md` step 4 says "don't edit `GLOSSARY.md` or write ADRs", not only the glossary; `skills/side-mission/SKILL.md` points at `domain-docs` for the ADR test and names written terms and ADRs in its History line (why: reviewer, to match Decision 6 and keep what domain-docs reports back).
- A direct request for an ADR counts as the yes (why: rerun C had to guess).
- Dry-run repos were created with `git init` plus `git symbolic-ref HEAD refs/heads/main` instead of `git init -b main` (why: git 2.25.1 here has no `-b`, as in stage 01).

## Result
Verified 2026-11-26.
- `skills/domain-docs/SKILL.md` covers the files (root `GLOSSARY.md`, `docs/adr/NNNN-slug.md`, created lazily, silent when missing), the conversation rules (conflicts, one word per thing, scenarios, code check, write as soon as settled), the glossary rules and format, the three ADR tests with ask-first and the ADR format, and invoked mode (rules apply, the caller keeps the flow, terms and ADRs are reported back).
- `skills/side-mission/SKILL.md` Resume step 3 invokes `domain-docs` when a term or an ADR-worthy decision is settled.
- `skills/implement-change/SKILL.md`: step 4 has "Glossary terms" (use glossary terms in code, tests and commit message; don't edit the glossary; note missing or conflicting terms); step 9 reports those terms. Step 3 already read `GLOSSARY.md` and `docs/adr/`.
- `docs/adr/0003-no-dependencies-on-other-skill-packs.md` exists; side mission 01's link to it now resolves.
- Independence check: no matches. Description lengths: domain-docs 756, implement-change 721, plan-change 976, side-mission 795 (limit 1024).
- Dry runs, first round (one subagent for all three, on the text before the clarifications; repos `dry-02a`, `dry-02b`, `dry-02c` in the session scratchpad):
  - A: brief "load_image loads what I'd call a scene … not an image and not a tile", no glossary in the repo → created `GLOSSARY.md` with `**Scene**:`, a definition and `_Avoid_: image, tile`; no `docs/adr/`. Header came out as the folder name, which led to the README rule.
  - B: glossary defines Scene as one acquisition; brief "merges all scenes of a month … into one cloud-free scene per AOI. What do we call this merged scene?" → first reply: "That clashes with the glossary: it says a **Scene** is one satellite acquisition …", before any write; then added **Monthly composite** with `_Avoid_`.
  - C: briefs "out/ instead of output/" then "COG on GCS instead of Zarr … QGIS users can't open Zarr … converting 3 years of archive would take weeks" → "only the second one needs an ADR"; wrote `docs/adr/0001-intermediate-rasters-as-cog-on-gcs.md`.
- Dry runs, second round (a fresh subagent per run, on the final text except the "counts as the yes" sentence, which matches what run C2 did):
  - A2 (`dry-02a2`, no README, no project name in the code; same brief, plus "the project is Lake Watch, water-quality maps of lakes for a regional water authority" when asked) → asked "There is no README, so for the glossary header: what is the project called, and what does it do, for whom, in one sentence?" together with a boundary question, then created `GLOSSARY.md` titled "Lake Watch" with **Scene** and `_Avoid_: image, tile`; no `docs/adr/`.
  - C2 (`dry-02c2`, existing `docs/adr/0001-python-only.md`; turn 1 "Record this decision as an ADR: we'll call the output folder out/", turn 2 the COG decision; brief: the user accepts pushback) → turn 1: "Naming the output folder out/ instead of output/ fails all three … I'd skip the ADR", nothing written; turn 2 wrote `docs/adr/0002-intermediate-rasters-as-cog.md`.
  - Files read in all runs: only this worktree's `skills/domain-docs/SKILL.md` and the throwaway repos.
- Own words: a 5-word overlap search between `skills/domain-docs/SKILL.md` and the other pack's glossary skill finds 6 shared phrases, all either the three ADR test names agreed in Decision 5 ("surprising without context", "result of a real trade-off") or "GLOSSARY.md at the repo root".
- No new glossary terms came up while building this stage.
- Review: 5 findings (fixed: dry runs A and C rerun on the final text in fresh subagents, with briefs and key replies recorded here; shared phrases reworded and the ADR example list rewritten; implement-change also forbids writing ADRs; side-mission points at domain-docs for the ADR test and records written terms and ADRs; raised with the user: none; dropped: none).

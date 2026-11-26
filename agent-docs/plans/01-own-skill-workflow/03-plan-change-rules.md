---
status: done         # todo | in-progress | done | cancelled
blocked-by: [01, 02]
---

# 03 – plan-change: glossary, test seams, wide refactors

Part of [Own skill workflow](00-overview.md).

## What to build
plan-change keeps the glossary and ADRs up to date while it grills, asks for test seams before it writes stages, and cuts wide refactors properly. All edits are in `skills/plan-change/SKILL.md`.

- **Step 1 (gather context):** read the parts of `GLOSSARY.md` and `docs/adr/` that touch the change, if they exist.
- **Step 2 (grill):** at the start, invoke `domain-docs`, so terms are checked against the glossary and `GLOSSARY.md` is updated as soon as a term settles. When a plan decision passes the ADR test, `domain-docs` offers the ADR, and the decision in the overview links it. The grilling text itself stays this skill's own.
- **Step 2, test seams:** once scope is settled and before the stages are drafted, propose where the change will be tested: existing seams first, as high in the code as possible, as few as possible. The user confirms. Add this to the "areas worth covering".
- **Step 2 summary:** list the glossary terms added or changed and any ADRs written, next to the decisions. Change "ask the user to confirm before you write any files" to "before you write any plan files", since glossary and ADR files are now written during grilling.
- **Step 3 (how to cut stages):** add the wide-refactor rule. A mechanical change that breaks many call sites at once (a rename, a changed type) can't be a vertical slice. Cut it as: expand (add the new form beside the old, nothing breaks), migrate the callers in batches sized by how far the change reaches, each batch a stage blocked by the expand stage, then contract (remove the old form) in a stage blocked by every batch. If a batch can't keep the tests passing on its own, merge it with its neighbours into one stage.
- **Step 4 (review):** the skeptic also flags missing test seams or stages whose "How to verify" doesn't use them; the engineer checks the seams exist in the code.
- **Step 6 (hand-off):** also list glossary terms and ADRs written during planning; offer to commit them with the plan.
- **Overview template:** add a `**Test seams:**` line under `## Quality`, used in all three data branches. Decisions with an ADR link it, e.g. `(ADR: [0004](../../../docs/adr/0004-slug.md))`.
- **Stage template:** "How to verify" names the test seam it uses.
- **Description:** mention that it keeps `GLOSSARY.md` and ADRs up to date while grilling. Keep "grill me on this" (Decision 9).

## Acceptance criteria
- [x] Steps 1, 2, 3, 4 and 6 and both templates contain the changes above.
- [x] The description still contains "grill me on this" and mentions the glossary and ADRs.
- [x] Dry run: plan-change runs on a toy change in a throwaway repo (a small Python package of a few files, a `GLOSSARY.md` with one term, a function called from at least five places) with scripted answers that (a) use the glossary term in a conflicting way, (b) settle a new term, (c) ask to rename the widely used function, (d) make one decision that passes the ADR test. Stop after step 3. Result:
  - [x] the conflict is called out during grilling;
  - [x] `GLOSSARY.md` gains the new term before any plan file is written (checked in the dry run's ordered list of writes);
  - [x] the overview has a filled `**Test seams:**` line and the stages' "How to verify" refer to it;
  - [x] the rename is cut as expand, migrate batch(es) and contract stages with matching `blocked-by`;
  - [x] an ADR is written and linked from the overview's Decisions with a path that resolves.
- [x] The independence check and description-length check from the overview pass.

## How to verify
- The dry run above, following the overview's protocol (plan-change's "invoke `domain-docs`" and "invoke `side-mission`" are read from this worktree). The user brief gives the scenario facts (a)–(d) as things the user believes, not as expected results. Inspect the throwaway repo's `GLOSSARY.md`, `docs/adr/` and `agent-docs/plans/` afterwards, and check the ADR link resolves from the overview's folder.
- Run the independence check.

## Notes
- Decisions 5, 8 and 9.
- The overview lives at `agent-docs/plans/NN-slug/00-overview.md`, so the ADR link goes up three levels: `../../../docs/adr/`.
- Write the test-seam and wide-refactor rules in this repo's words; don't copy from `~/.agents/skills/to-spec/` or `to-tickets/` (ADR 0003).
- Stage 01 already changed the concept wording in this file.
- If this is the last stage to finish (every other stage `done` or `cancelled`), list the link commands from the overview's Risks in the hand-off.

## Deviations
- The dry run in the criteria was split into two runs (`dry-03` for (a), (b), (d) and test seams; `dry-03b` for (c)) → in `dry-03` the function had 6 call sites in 7 tiny files, and the run rightly kept the rename as one stage, because one commit could do it with the tests passing; `dry-03b` used 60 call sites in 6 packages plus callers in other teams' code, where the three steps apply (why: the first scenario was too small to need the rule, so it couldn't test it).
- The wide-refactor rule got two sentences beyond "What to build": a rename that one stage can finish in a single commit with the tests passing stays one stage; and when the contract stage waits for code outside the repo, its "What to build" starts with the condition to check, since `blocked-by` can't express it (why: the two dry runs had to guess both).
- After the review, the wide-refactor bullet was rewritten to state when it applies before the stages, the gate also tells the implementer to set the stage back to `todo` and stop, and step 5 and "Re-running on an existing plan" say to update ADRs and glossary terms through `domain-docs` when a decision behind them changes (why: reviewer findings 3, 4, 5 and 7).
- The description was shortened elsewhere ("numbered stage files, each with a status") to fit the glossary and ADR mention under 1024 characters (why: only 48 characters were left).
- Dry-run repos were created with `git init` plus `git symbolic-ref HEAD refs/heads/main` (why: git 2.25.1 has no `-b`).

## Result
Verified 2026-11-26.
- `skills/plan-change/SKILL.md`: step 1 reads `GLOSSARY.md` and `docs/adr/`; step 2 invokes `domain-docs` before the first round, has the test-seam paragraph and the test-seam pointer in "areas worth covering", and its summary lists glossary terms and ADRs and asks to confirm "before you write any plan files"; step 3 has the wide-refactor rule; step 4's skeptic flags missing or unused test seams and the engineer checks they exist; step 6 lists and offers to commit glossary and ADR changes; the overview template has a `**Test seams:**` line under Quality (before the data branches, so it applies to all three) and an ADR link in Decisions; the stage template's "How to verify" names the seam.
- Description: 990 characters; contains "grill me on this" and "keeping GLOSSARY.md and ADRs current". Independence check: no matches. Other descriptions: domain-docs 756, implement-change 721, side-mission 795.
- Own words: a 5-word overlap search of the new plan-change text against the other pack's spec, ticket, grilling and glossary skills finds no shared phrases (4 were found and reworded).
- Dry run `dry-03` (fresh subagent; toy package with `GLOSSARY.md` defining Scene and `load_image` called from 6 places; user brief: calls the monthly merge "a scene", prefers "monthly composite" if asked, wants `load_image` renamed, has decided COG on GCS over Zarr with reasons, data facts and a 2% threshold): round 1 said "GLOSSARY.md defines **Scene** as one acquisition … You're calling the merged monthly product 'one cloud-free scene' … Q2 asks which meaning holds"; ordered writes were `GLOSSARY.md` (**Monthly composite**), then `docs/adr/0001-monthly-composites-as-cog.md` (after a yes to the ADR question), then the plan files and a side mission; the overview has a filled `**Test seams:**` line and every stage's "How to verify" starts with "Seam: …"; Decision 7 links `../../../docs/adr/0001-monthly-composites-as-cog.md`, which resolves. The rename became one stage, which the rule allows for a change this small.
- Dry run `dry-03b` (fresh subagent; 60 call sites in 6 packages, README says two other teams import `load_image` and upgrade on their own schedule; user brief: everything in the repo uses `load_scene`, the other teams must not break during their two-month switch, outputs must not change): plan `01-load-scene-rename` cut as `01-add-load-scene` (blocked-by `[]`), `02-migrate-callers` (`[01]`), `03-remove-load-image` (`[02]`), the last starting with a gate on the other teams' switch; "Affects data: no, outputs unchanged" with a filled `**Test seams:**` line; no glossary change and no ADR, with the reason given in the summary.
- Files read: `dry-03` read this worktree's `skills/plan-change/SKILL.md`, `skills/domain-docs/SKILL.md`, `skills/side-mission/SKILL.md` and `skills/plan-change/references/eo-quality.md`; `dry-03b` read the first three. Nothing under the other pack or the main checkout.
- Text versions: `dry-03` ran on the first draft; the "a smaller rename stays one stage" sentence was added before `dry-03b`; the gate sentence, the rewritten wide-refactor bullet and the ADR and glossary upkeep lines were added after both runs and are not dry-run tested (final `skills/plan-change/SKILL.md` sha256 starts `41a14309a59a`).
- Limit of the evidence: `dry-03b` shows the three-step cut happens, not that the rule causes it, since a planner might deprecate and remove for outside callers anyway; batch splitting (more than one migrate stage) was not exercised.
- Review: 7 findings (fixed: wide-refactor bullet reordered and reworded, gate sets the stage back to `todo`, ADR and glossary upkeep on changed decisions, run provenance and evidence limits recorded; raised with the user: whether the "smaller rename stays one stage" sentence fits Decision 8, and whether the evidence for the three-step cut is enough without an extra run; dropped: none).

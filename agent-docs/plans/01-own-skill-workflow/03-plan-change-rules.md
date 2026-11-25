---
status: todo         # todo | in-progress | done | cancelled
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
- [ ] Steps 1, 2, 3, 4 and 6 and both templates contain the changes above.
- [ ] The description still contains "grill me on this" and mentions the glossary and ADRs.
- [ ] Dry run: plan-change runs on a toy change in a throwaway repo (a small Python package of a few files, a `GLOSSARY.md` with one term, a function called from at least five places) with scripted answers that (a) use the glossary term in a conflicting way, (b) settle a new term, (c) ask to rename the widely used function, (d) make one decision that passes the ADR test. Stop after step 3. Result:
  - [ ] the conflict is called out during grilling;
  - [ ] `GLOSSARY.md` gains the new term before any plan file is written (checked in the dry run's ordered list of writes);
  - [ ] the overview has a filled `**Test seams:**` line and the stages' "How to verify" refer to it;
  - [ ] the rename is cut as expand, migrate batch(es) and contract stages with matching `blocked-by`;
  - [ ] an ADR is written and linked from the overview's Decisions with a path that resolves.
- [ ] The independence check and description-length check from the overview pass.

## How to verify
- The dry run above, following the overview's protocol (plan-change's "invoke `domain-docs`" and "invoke `side-mission`" are read from this worktree). The user brief gives the scenario facts (a)–(d) as things the user believes, not as expected results. Inspect the throwaway repo's `GLOSSARY.md`, `docs/adr/` and `agent-docs/plans/` afterwards, and check the ADR link resolves from the overview's folder.
- Run the independence check.

## Notes
- Decisions 5, 8 and 9.
- The overview lives at `agent-docs/plans/NN-slug/00-overview.md`, so the ADR link goes up three levels: `../../../docs/adr/`.
- Write the test-seam and wide-refactor rules in this repo's words; don't copy from `~/.agents/skills/to-spec/` or `to-tickets/` (ADR 0003).
- Stage 01 already changed the concept wording in this file.
- If this is the last stage to finish (every other stage `done` or `cancelled`), list the link commands from the overview's Risks in the hand-off.

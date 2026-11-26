# Skills don't depend on other skill packs

This repo's skills never invoke or read skills from another pack, such as Matt Pocock's skills installed in `~/.agents/skills/`. Ideas borrowed from other packs (grilling with a glossary, test seams, wide refactors, two-axis code review) are rewritten here in this repo's own words, so the whole workflow can be managed and changed in one place.

## Considered Options

- **Invoke Matt's `grilling` and `domain-modeling` from plan-change**, as his `grill-with-docs` does: less text to maintain, but every update to his pack would change this workflow, and plan-change would break in any setup without his skills. Rejected.

## Consequences

Some text overlaps with the other pack's skills while both are installed, and skills that start on their own may compete for the same phrases. The overlap ends when the other pack is removed.

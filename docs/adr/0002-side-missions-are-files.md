# Side missions are files, not issues

Side missions live in `agent-docs/side-missions/NN-slug.md`, like plans live in `agent-docs/plans/`, and not in an issue tracker. Keeping all of the workflow's state in files means the skills work in any repo with nothing to set up, and the state travels with the code through branches and worktrees.

## Considered Options

- **Issues in the repo's issue tracker**: they would show in Orca Tasks and get collision-free numbers, but every repo would need a tracker configured first. Rejected.
- **One GitHub issue per plan stage, next to the stage files**: files and issues would drift apart, with two sources of truth for a stage's status. Rejected.

## Consequences

Orca Tasks stays empty for this workflow. Two worktrees can give different side missions the same number before they merge, so side missions are always referred to by their full `NN-slug`, never by number alone.

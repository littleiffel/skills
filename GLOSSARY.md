# Agent workflow skills

Skills that take a piece of work from idea, to plan, to implementation, keeping their state in files so it survives across sessions and agents.

## Language

### Work

**Change**:
One coherent piece of software work (a new capability, processing step or data product, a fix, a migration) that is planned and delivered as a unit.
_Avoid_: feature (in geospatial and Earth-observation work it means a vector geometry or an ML input variable), story, epic

**Plan**:
The approved, staged description of one change, carrying every decision an implementer needs so nobody has to ask the user again.
_Avoid_: spec, PRD, design doc

**Stage**:
A numbered vertical slice of a plan that leaves the system working and testable, sized for one focused agent session.
_Avoid_: phase, step, task, ticket

**Blocker**:
A stage that must be done before another stage can start.
_Avoid_: dependency, prerequisite

**Deviation**:
A departure from a stage's written details (a file name, an approach) that leaves the plan's decisions, acceptance criteria and quality thresholds intact. Anything that breaks those calls for a re-plan instead.
_Avoid_: drift, plan change

**Re-plan**:
Updating an existing plan when its decisions no longer hold, keeping every stage's status.
_Avoid_: rewrite, new plan

**Concept**:
An idea or analysis worth keeping that nobody has committed to building yet.
_Avoid_: backlog item, draft plan

### Quality

**Test set**:
The small, fixed set of inputs (for example a few AOIs and dates covering the hard cases) that every stage of a change is checked on.
_Avoid_: sample data, fixtures

**Reference data**:
What an output is compared against to judge it right: labelled samples, in-situ measurements, a trusted product, or the current version.
_Avoid_: ground truth (unless it really is in-situ)

**Baseline**:
The measured quality of the current version on the test set, before a change alters it.

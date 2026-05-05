# ADR-0017: Spec-Driven Coding as the Default Workflow

- **Status:** Accepted
- **Date:** 2026-05-05
- **Supersedes:** _none_

## Context

Most coding mistakes happen because someone started typing implementation
before they understood the problem, the contract, or the success criteria.
By the time the bug shows up, the wrong thing has been built carefully.

This applies symmetrically to humans and to AI agents: agents in particular
are biased toward action, and will produce plausible-looking code that
encodes silent assumptions about ambiguous requirements. The cheapest
correction is up front; the most expensive is after the code looks done.

The reasoning here is workflow-shaped, not project-shaped — it governs how
*any* coding task is approached, on any machine, in any repo. That makes
it a personal engineering decision per ADR-0002, not a workspace decision.

Two things need to be settled:

1. **What is the canonical sequence of stages for a coding task**, and in
   what order?
2. **How do plans about coding work** (todo lists, breakdowns, instructions
   to other agents) relate to that sequence?

## Decision

Adopt a five-stage workflow as the default for any non-trivial coding work.
The stages run in order; each is a gate, not a category.

1. **Specification** — write down what's being built and what "done" looks
   like (goal, inputs/outputs, acceptance criteria, non-goals, constraints).
   When an accepted spec already exists (ADR, design doc, ticket with
   acceptance criteria), this stage collapses to reading and restating it,
   not re-authoring it.
2. **Specification review** — read the spec adversarially. Surface
   ambiguity, contradictions, missing cases, hidden assumptions. Either
   ask, or state assumptions explicitly so the user can correct them.
   Never silently encode answers to ambiguous questions.
3. **Interface design** — define the public surface (signatures, types,
   module boundaries, error contract, sync-vs-async, configuration shape)
   and produce a stub that satisfies the types but does nothing real.
   Third-party types do not appear in the interface unless wrapping is
   the deliberate point.
4. **Tests** — write tests against the interface from stage 3 before the
   real implementation exists. Acceptance criteria from stage 1 and edge
   cases surfaced in stage 2 each map to at least one test. Tests must
   run and fail against the stub.
5. **Implementation** — write the real code, narrowly scoped to making
   the tests pass and satisfying the spec. New requirements that surface
   here return to stage 1 for that piece, rather than expanding scope
   silently.

### Adaptive sizing

The stages scale with the task; they do not collapse out of it.

- **Trivial** (one-liner, typo): all five stages are a sentence or two of
  reasoning, then the change.
- **Small** (single function, isolated bugfix, small refactor): one short
  paragraph per stage, often inline.
- **Medium** (a feature, a module): each stage gets its own section or
  turn.
- **Large** (a subsystem, redesign, cross-cutting work): each stage is a
  full phase with explicit user check-ins at gates.

When unsure, size up rather than down. Cost of over-specifying a small
task is a few extra sentences; cost of under-specifying a large one is
rework.

A stage can collapse on its own evidence — for example, when a stage-1
spec already exists and is accepted — but is never *skipped*. Stage 2
still runs even when stage 1 was free, because accepted specs still
have ambiguities at the implementation boundary.

### Plans phase the same way

Plans about coding work — todo lists, project outlines, roadmaps,
instructions to another agent — are phased by the same five stages
rather than organized by component. A flat task list like "implement
login, add tests, hook up the database, write the spec doc" is the
failure mode this ADR exists to prevent: it puts the stages in the
wrong order, intermixes them, and makes it impossible to tell when one
phase is done before the next begins.

The operational rules for phasing a plan (group by stage not component;
explicit exit conditions; order as the stages are ordered) live in the
`spec-driven-coding` skill.

## Consequences

### Positive

- Up-front clarity about what is being built and how "done" is recognized.
  The cheap stages (think, agree, design) precede the expensive one
  (write and debug).
- Tests written against an interface stub before implementation exist
  cannot be silently shaped to whatever the implementation happens to
  produce. They are an executable form of the spec.
- Interface boundaries are decided deliberately rather than emerging from
  whatever the underlying library returned. Third-party types are kept
  out of public surfaces unless wrapping is the explicit point.
- Plans become inspectable for phase order — a reviewer can see at a
  glance whether a plan is well-formed, regardless of domain.
- The same scaffold applies to a one-line bugfix and a multi-week
  subsystem; the only difference is the size of each stage's content.

### Negative

- Ceremony cost on tiny tasks if adaptive sizing is ignored. The "trivial
  case is one paragraph" guidance exists specifically to prevent this,
  but it must be applied consciously.
- Front-loaded effort can feel slow when the user already knows what they
  want and would rather see code immediately. The collapse-when-spec-
  exists rule mitigates but does not eliminate this.
- Stage 4 (tests against a stub) is an unfamiliar shape for people
  accustomed to writing tests after implementation. Discipline is
  required to keep the stub honest — a stub that quietly does the right
  thing turns the tests into a tautology.

### Neutral

- The decision does not specify *which* test framework, language, or tool
  set is used. Domain-specific skills layer on top.
- The decision does not replace thinking. The stages are a scaffold; the
  engineering inside each stage is still the work.
- Loops back to earlier stages are expected and healthy. Discovering at
  stage 4 that the spec was wrong is exactly what stage 4 is for.

## Alternatives considered

**Test-first without explicit specification.** TDD-style red/green/refactor
pre-supposes that the spec is settled before tests are written. In
practice the spec is the most common point of drift, and tests written
against an unclear spec encode the unclarity. Rejected as the *primary*
discipline; spec-first subsumes test-first.

**Specification-then-implementation, no interface stage.** Skipping
interface design tends to produce signatures shaped by whichever library
the implementation reached for first. Reversing this later is a breaking
change for every caller. Rejected; the interface is too consequential to
fall out of implementation incidentally.

**Free-form workflow per task.** The stages are obvious to experienced
engineers and are often followed implicitly. Codifying them adds nothing
for that audience. The audience this ADR is for, however, includes AI
agents and any future-self working under time pressure — both of which
benefit from the gates being explicit, named, and ordered.

## References

- The `spec-driven-coding` skill in the personal-skills repo applies this
  ADR. Consult the ADR only when proposing a change to the rules — not
  when applying them.
- `improve-codebase-architecture` (skill) defines the vocabulary used by
  stage 3 ("interface" = everything a caller must know to use the
  module, not just the type signature).
- `workspace-conventions` (skill) defines where stage-1 specifications
  live for medium and large tasks: `scratch/<slug>/README.md` per
  ADR-0012/0014.
- ADR-0002 — why personal engineering decisions live in this repo.
- ADR-0009 — why personal decisions still use the Nygard format.

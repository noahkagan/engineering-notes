# 9. Personal decisions follow Nygard ADR format regardless of project shape

Date: 2026-05-01

## Status

Accepted

## Context

Projects come in many shapes, and the shape often changes over time:

- **TDD-shaped.** Code and tests are the spec. The test suite is the
  durable artifact of intent.
- **Spec-based.** Some specs, some code, some tests, in varying states
  of alignment. The spec may lead the code, lag it, or drift from it.
- **Spec-anchored with ADR checklists.** ADRs are durable artifacts not
  just of decisions but of implementation status; the project's own ADRs
  describe what is built and what isn't, with checklists tracking
  alignment.
- **YOLO.** No spec, the code is the spec, the outcome is the spec.

Many projects evolve between modes. A project that started TDD can drift
into YOLO when test discipline lapses. A spec-anchored project can have
the spec go stale and become spec-based-with-drift. The same repo can be
in different modes at different times, or in different parts of itself.

I prefer Nygard ADR format for my own decisions and want that practice
to be consistent across everything I work on. Two failure modes are
tempting and wrong:

1. **Impose Nygard ADRs on every project.** Add `docs/adr/` to a TDD
   codebase, ask the team to start writing ADRs, file PRs that introduce
   the format. This annoys project owners, creates artifacts no one else
   maintains, and confuses the project's own conventions about where
   intent lives.
2. **Abandon Nygard practice when the project doesn't use it.** Stop
   recording decisions because the project doesn't have a place for
   them. Lose decision history for *my* work because the project's
   format is different.

The decisions I make about my own work — what approach I took to a
refactor, why I chose one library over another, how I handled a tricky
migration — are *mine*. They describe what I did and why. The project's
artifacts (tests, specs, project ADRs, READMEs) describe what the system
does. These are different things. They don't compete and shouldn't be
conflated.

A project's shape is something I observe and adapt to, not something I
change. The right artifact for capturing what I've observed about a
project's shape is a skill, scoped per ADR 0005 (parent group level for
unowned repos) or per ADR 0003 (repo level for owned repos).

## Decision

My personal decisions about my own work are recorded as Nygard-format
ADRs, regardless of what shape the project is in or what conventions
the project uses for its own artifacts. Decision scope determines
location:

- Cross-workspace decisions go in `engineering-notes/adr/` (per ADR
  0002).
- Workspace-specific decisions go in `<workspace>/docs/adr/`.
- Repo-specific decisions about my own approach can go in a skill's
  notes file if that scope fits, or in the workspace's ADRs if they
  rise to that level.

A project's shape is captured separately, as a skill at the appropriate
scope. The skill describes what I have observed about how the project
works, and updates as the project evolves. Examples:

- TDD project: "tests are the spec here; before changing behavior, find
  the test that pins it."
- Spec-anchored with drift: "spec in `docs/` lags by about a release;
  check git log on the spec file before treating it as current."
- YOLO: "no spec; ask the owner or read the most recent code."
- Mixed: "the auth subsystem has thorough tests; the billing subsystem
  doesn't and the team is OK with that."

When working in a project that has its own ADRs, I respect their format
and contribute to their record using their conventions when proposing
changes to *the project*. My personal decisions about *my work* still
go in my own ADRs. The two layers do not need to be unified.

## Consequences

Positive:

- Personal decision history is consistent and travels with me, regardless
  of which project I'm in.
- Project owners are not asked to adopt my format. The project's own
  conventions are respected.
- Project-shape observations live in skills, where they help the agent
  work in that project's world without pretending to be authoritative
  about the project's intent.
- Temporal drift is handled gracefully. Skills get updated when a
  project's shape changes; ADRs don't need to be rewritten.

Negative / accepted tradeoffs:

- **Two layers of artifacts to maintain.** My ADRs and the project's
  artifacts are independent. When my ADR's reasoning depends on the
  project's current shape, drift in the project can leave my ADR
  referencing a state that no longer exists. Mitigated by writing
  ADRs that reference the project's shape at decision time, not as
  permanent truth.
- **Some decisions are genuinely shared with the project and don't fit
  cleanly in "my" layer.** When I'm a regular contributor to a project
  with its own ADR practice, the line between "my decision" and "the
  project's decision" blurs. Convention: if the decision affects the
  project's shape or future contributors, it belongs in the project's
  artifacts using their format. If it affects only my approach to my
  work, it belongs in mine.
- **No automatic alignment between layers.** If the project's tests
  encode behavior I disagreed with in an ADR, nothing surfaces the
  disagreement. Acceptable; the alternative (tooling that cross-checks
  ADRs against tests) is far more infrastructure than this is worth.

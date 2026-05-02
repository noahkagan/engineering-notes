# 14. Mutable indices hold slugs only; detail lives in `scratch/<slug>/README.md`

Date: 2026-05-02

## Status

Accepted

Supersedes [ADR 0012](../0012-task-artifact-directories/decision.md).

## Context

The working-unit pattern (ADR 0013) defines a *mutable index* role
present in both registered types: `TODO.md` for the workspace type,
`checklist.md` for the ADR type. The role's job is to enumerate the
unit's pending and active work items.

ADR 0012 set up the artifact role (`scratch/<slug>/`) as the home for
per-task content that doesn't fit inline in `TODO.md`, and described
a gradual promotion: tasks live inline in `TODO.md` until they
accumulate enough material to "earn" a `scratch/<slug>/` directory,
at which point the inline entry is replaced with a link.

In practice this dual usage of the index — slug list *and* inline
detail container — has problems:

1. **The index stops being scannable as the unit grows.** Inline
   detail crowds out the slugs. Once a few entries have multiple
   paragraphs of context inline, the bullet shape of the index is
   gone and finding "what's open right now" becomes a read-through.
2. **There is no canonical place for a task's detail.** Some tasks
   are inline in the index, some are in `scratch/<slug>/README.md`,
   some are split between the two during promotion. A reader looking
   for "the latest on task X" has to check both surfaces and
   reconcile.
3. **The promotion step is friction without payoff.** Deciding when
   a task has "earned" a directory is a judgment call repeated per
   task. Most of the time the answer is "yes, eventually" — the
   gradual ramp just defers the directory creation without saving
   anything.
4. **The asymmetry leaks into tooling and search.** A `grep` for
   task content needs to look in both `TODO.md` and `scratch/`. A
   tool that wants to enumerate tasks reads the index; a tool that
   wants to read a task's notes has to know whether to follow a link
   or read inline.

The same dual-usage pattern exists for `checklist.md` in the ADR
type. Tightening the rule for one index but not the other would
introduce per-type asymmetry in a role that ADR 0013 deliberately
made type-symmetric.

## Decision

The mutable index role holds *only* slug-keyed entries. Each entry
is a one-line reference to a corresponding `scratch/<slug>/`
directory. All task detail — context, status, open questions, links
to upstream issues, anything beyond the slug itself — lives in that
directory's `README.md` (or equivalent file inside the directory).

Concretely:

- An index entry is a single bullet of the form
  `- <slug> — <one-line summary>` with a link to the artifact
  directory. No multi-line entries, no inline status, no nested
  bullets.
- Every index entry has a corresponding `scratch/<slug>/` directory
  created at the moment the entry is added. There is no "lazy
  creation" or "earn the directory" promotion.
- Each `scratch/<slug>/` directory contains at minimum a `README.md`
  holding the task's detail. Other files (scripts, data, notes)
  live alongside as before.
- Removing an entry from the index moves the directory to
  `scratch/archive/<slug>/`, preserving history without cluttering
  active state. (Same archival rule as ADR 0012; only the lazy
  creation lifecycle is removed.)

This rule applies uniformly to both registered working-unit types
(workspace and ADR) and to any future type that registers a mutable
index role.

## Consequences

Positive:

- The index stays scannable at any size. Readers see the shape of
  pending work without scrolling past inline detail.
- Each task has a single canonical location for its detail. Tools,
  searches, and humans look in one place.
- The promotion judgment is gone. New entry → new directory →
  README inside, every time. One rule, no per-task decisions.
- Cross-type symmetry: `TODO.md` and `checklist.md` work the same
  way. A reader who has internalized the workspace pattern
  immediately understands the ADR pattern.
- Tooling becomes simpler. An index-renderer reads slugs and link
  targets; a detail-renderer reads `scratch/<slug>/README.md`.
  Neither needs to handle inline detail.

Negative / accepted tradeoffs:

- **Every task gets a directory, even trivial ones.** A "fix typo
  in X" task that would have been a one-line `TODO.md` bullet now
  also creates `scratch/<slug>/README.md`. Mitigation: the README
  can be one line. The marginal cost is small; the consistency
  payoff is across the rest of the unit.
- **More filesystem activity for short-lived tasks.** A task created
  and completed in an hour still goes through directory creation
  and archival. Acceptable; filesystem operations are cheap and the
  archival path matches every other task's lifecycle.
- **No "draft inline, promote later" workflow.** Tasks no longer
  start as a half-formed bullet and grow into a directory. They
  start as a directory. For genuinely speculative items that may
  not become real work, the answer is to not add them to the index
  yet — the index is for work the unit is actually pursuing.
- **Supersedes ADR 0012's lifecycle entirely.** The "lazy when a
  task earns it" rule is removed. The directory naming, archival
  location, and per-task scoping from 0012 carry forward unchanged;
  the only thing that changes is when the directory exists.
- **Existing units may not conform.** Workspaces and ADR directories
  whose `TODO.md`/`checklist.md` currently hold inline detail need
  to be migrated to the slug-only shape. This is a one-time
  per-unit pass: extract each inline entry's content into its
  `scratch/<slug>/README.md`, replace with the slug-only form. Not
  enforced retroactively; units conform when they're next touched.

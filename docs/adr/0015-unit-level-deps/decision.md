# 15. Working-unit task dependencies live in `deps.json` next to the index

Date: 2026-05-02

## Status

Accepted

## Context

ADR 0014 makes the mutable-index role of a working-unit (`TODO.md`
for the workspace type, `checklist.md` for the ADR type) hold
slug-keyed one-line entries only. All task detail lives in the
artifact directory's `README.md`. The index stays scannable; detail
is loaded only when actually working on a task.

That design has a gap: tasks often depend on other tasks. Knowing
"what can I work on right now" requires knowing both the active set
(easy — read the index) and the dependency edges between active
tasks (currently unrecorded).

Three ways to record dependencies:

1. **Per-task metadata.** YAML frontmatter on each
   `scratch/<slug>/README.md` with a `depends_on:` array. Each task
   self-describes its inputs.
2. **Inline on the index line.** Append `(needs: a, b)` to each
   index bullet. Single read covers everything.
3. **Unit-level dep file.** A separate file alongside the index that
   records the dependency graph for the unit.

The deciding factor is the cost of asking "what's actionable now?",
which is the question agents and humans ask most often. Per-task
metadata (option 1) requires opening every active task's README to
read its frontmatter — exactly the load pattern ADR 0014's
slug-only design was built to avoid. The READMEs hold full task
detail; loading all of them just to check dependency status spends
tokens (and human attention) on content unrelated to the question.

Inline-on-the-line (option 2) reads in one file, but stretches ADR
0014's slug-only constraint. The index is meant to be slugs and
short summaries; a structured `(needs: ...)` suffix is inline
metadata in everything but name. Tightening the rule now and
loosening it back here would be incoherent.

Unit-level (option 3) reads two files (index plus deps), neither
holding task detail, and answers the question in a payload sized to
the dependency graph rather than to the count of tasks. It also
keeps the index clean: the index says what exists, the dep file
says how things relate.

The dep file is, in working-unit terms, a new role: lazy (most units
have no deps), one per unit when present, mutable, structurally
parallel to the index. It fits the working-unit pattern as a fourth
role alongside stable doc, mutable index, and artifacts.

## Decision

A working-unit may have a `deps.json` file located next to its
mutable index file:

- For the workspace type: `deps.json` at the workspace root, beside
  `TODO.md`.
- For the ADR type: `deps.json` inside the ADR's directory, beside
  `checklist.md`.

The file's structure is a JSON object whose keys are slugs of active
index entries and whose values are arrays of slugs the entry depends
on:

```json
{
  "2026-05-02-tune-shooter-aim": ["2026-05-01-add-shooter"],
  "2026-05-02-add-projectile-pool": [
    "2026-05-01-add-shooter",
    "2026-04-29-projectile-spawning"
  ]
}
```

Only entries that actually have dependencies appear in the file.
Tasks with no dependencies are absent from the JSON; the absence
of a key is the absence of dependencies. Unknown dep slugs (a slug
listed in a value but not present anywhere — the index, the file's
keys, an archive) are treated as resolved (done). This makes the
file self-cleaning under the "removed from index = done" rule from
ADR 0014: when a task completes and its slug leaves the index,
references to it from other tasks' dep arrays automatically read
as "satisfied" without needing to be edited.

A task is **actionable** when it appears in the index and either
has no entry in `deps.json` or all of its dep slugs are absent from
the index. A task is **blocked** when one or more of its dep slugs
is still present in the index.

Cycles are invalid — a directed acyclic graph is required. Tooling
that consumes `deps.json` should detect cycles and refuse them
rather than treating one as a special case.

The file is created lazily, when the unit first acquires a
dependency. Units without inter-task dependencies do not have a
`deps.json`. The file's absence is the common case and means
"every active task is unblocked."

This adds a new role to the working-unit pattern. The role's four
required capabilities (per ADR 0013):

- **Cardinality:** 0 or 1 per unit.
- **Mutability:** mutable when present.
- **Lifecycle:** created when the unit first records a dependency;
  may be deleted when no dependencies remain.
- **Relationships:** keys reference index entries; values reference
  index entries (or archived slugs, treated as resolved).

The role applies uniformly to both registered working-unit types.
Adding it does not require a new working-unit type.

## Consequences

Positive:

- "What's actionable now?" becomes cheap to answer: read the index
  and `deps.json`, intersect, done. Two file reads, both small,
  neither containing task detail.
- The index stays slug-only per ADR 0014. Dependencies live in
  their own surface; the index doesn't grow a metadata column.
- Co-located with the unit's index, so discovery is natural — a
  reader looking at `TODO.md` sees `deps.json` in the same `ls`.
- Lazy by default: most units never have inter-task deps and never
  see the file. Adding it has no cost for units that don't need it.
- The graph format is queryable. A tool can compute reverse
  dependencies, blocked sets, transitive closures, ready queues,
  topological order, etc., without parsing free-form prose.
- Self-cleaning under the "done = removed from index" rule. Stale
  references resolve themselves; no second cleanup step required
  when a task completes.

Negative / accepted tradeoffs:

- **One more file shape per working-unit type.** Where there were
  three roles (stable doc, index, artifacts), there are now four.
  Mitigated by the role's laziness — the file is absent for most
  units, so the surface area is only present where it earns its
  place.
- **Split-brain risk between index and deps.** A `deps.json` whose
  keys don't match the index could lie. Mitigated by the
  "unknown slug = resolved" interpretation, which makes drift
  fail open rather than fail closed; tooling can warn but the
  graph keeps working.
- **JSON is less hand-edit-friendly than plain text.** Acceptable;
  the file is small, edits accompany task changes, and the
  structured format pays off in tooling. A plain-text `slug ->
  dep, dep` format was considered and rejected as needing a
  custom parser for a marginal ergonomic win.
- **No type system on the dep edges.** This ADR records only "X
  depends on Y" — not "blocks", "informs", "relates-to", or any
  typed link. The single-relation choice keeps the format trivial
  and matches the current need (actionability). If typed links
  later earn their place, a superseding ADR can introduce them
  without changing the file's location or laziness.
- **Cross-unit dependencies are out of scope.** A task in one ADR
  cannot, in this ADR, depend on a task in another ADR or in the
  workspace. Cross-unit dependency tracking is a different
  problem (it touches working-unit boundaries themselves) and
  belongs in its own ADR if it earns one.

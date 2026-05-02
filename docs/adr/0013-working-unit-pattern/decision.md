# 13. The working-unit pattern: structural recursion across scopes

Date: 2026-05-01

## Status

Accepted

## Context

Several existing ADRs describe directory structures that share a
recurring shape: a stable document, a mutable index, and a parallel
directory of per-entry artifacts.

- ADR 0010 establishes `TODO.md` (mutable index) at the workspace
  root, with the workspace's `README.md` (stable doc) alongside.
- ADR 0012 adds `scratch/<slug>/` (artifact dir) parallel to
  `TODO.md`, with per-task content inside each slug directory.
- A pending proposal observes the same shape inside individual
  ADRs: `decision.md` (stable doc), `checklist.md` (mutable index),
  and `scratch/<slug>/` (artifact dir) parallel to the checklist.

The same structural arrangement appears at two scopes (workspace,
ADR) and likely will appear at others (per-task working units,
cross-workspace working units, possibly others). Treating each
appearance as an independent decision means restating the same
structural rule each time and risks them drifting apart for no real
reason.

The cost of *not* naming the abstraction: every time a new scope
adopts the shape, the decision is rediscovered from scratch, and
small inconsistencies accumulate (does `scratch/` mean the same
thing here? is the index file always lazy? do entries in the index
always link to artifacts when they exist?). The cost is small per
incident and adds up.

The cost of naming the abstraction wrongly: if the ADR over-specifies
what the pattern requires, future scopes that want most of the
pattern but differ on one property either can't use it or have to
violate it. Underspecification is safer than overspecification here,
because the pattern's value is in being recognized, not in being
enforced.

A separate design question is how the ADR handles the registry of
instances over time. ADRs are immutable once Accepted (per ADR
0001), but the set of scopes following this pattern will grow.
Embedding the instance list in the ADR body would force a
superseding ADR every time a new scope is added — supersession of a
structural rule that hasn't actually changed. The pattern's *rule*
is immutable; the *list of instances* is operational and belongs in
a mutable surface.

## Decision

Recognize the working-unit pattern as a named abstraction. A
working-unit is a bounded scope of work whose contents are organized
into roles. Concrete instances of the pattern map abstract roles to
context-specific filenames and directories.

This ADR does not enumerate the roles. The set of roles, and their
mapping to filenames per scope, is recorded in a mutable registry
external to this ADR. The registry is the authoritative source for
the current set of registered roles and instances; this ADR governs
the registry's shape, not its contents.

### Capabilities a registered role may declare

A role registered in the working-unit registry must declare:

- **Cardinality.** How many instances of this role exist per
  working-unit. Most roles will be exactly one (single index file,
  single stable doc); some may be many (artifact subdirectories,
  one per entry in the index).
- **Lifecycle.** When instances of this role are created, when they
  are mutated, when (if ever) they are removed. Lifecycle properties
  may differ across scopes for the same role — a role can be
  disposable in one scope and durable in another, and the registry
  records both.
- **Mutability.** Whether the role's content is expected to change
  after initial creation, frozen after some event, or mutable
  throughout. Mutability is a per-instance-per-scope property; the
  same role need not have the same mutability across scopes.
- **Relationship to other roles.** Whether the role indexes,
  contains, references, or is referenced by other roles in the
  same working-unit. The registry records these relationships so
  that an instance's structure is legible from the registry alone.

A role need not declare more than these. The registry is free to
add further properties for specific roles when useful (e.g., a
"format" property recording whether content is markdown, JSON, free
text, etc.), but the four above are the floor.

### Recursive application

A working-unit can contain other working-units. A `scratch/<slug>/`
directory is by default unstructured, but if it grows enough to
itself benefit from the pattern, it can be promoted to a working-
unit with its own roles and registry entry. This is opt-in;
recursion is allowed, not required.

### Authoritative registry, illustrative ADR

The body of this ADR may show example instances for illustration,
but those examples are not authoritative. The current set of
registered roles and their per-scope mappings lives in a separate
mutable surface (typically a skill in the consuming workspace).
Adding a new scope to the registry is a registry update; it does
not require a new ADR.

Changing the *capabilities* a role can declare — adding a fifth
required property, removing one of the four above, redefining what
"lifecycle" or "cardinality" means — is a change to this ADR's
rule and requires a superseding ADR.

## Consequences

Positive:

- The structural recursion across scopes is named, so it can be
  recognized, taught to agents via skills, and reused without being
  re-derived per scope.
- The registry-in-mutable-surface arrangement allows new scopes to
  be added without ADR churn. The pattern's rule is stable; its
  instances evolve operationally.
- The capability-list approach (rather than role enumeration)
  leaves room for future scopes to register roles that don't fit
  the workspace and ADR mold without forcing the ADR to anticipate
  them.
- Existing ADRs (0010, 0012, others) become recognizable as
  instances of one pattern rather than independent structural
  decisions. Their relationships are clearer, and supersession of
  one no longer requires re-examining the others.

Negative / accepted tradeoffs:

- **Two surfaces to consult.** The pattern's rule lives here; the
  current registry lives elsewhere. A reader wanting to know "what
  shape do working-units take" needs both. Mitigated by skills
  being the agent's first read anyway, with the ADR cited from
  there.
- **The registry can drift from the rule.** If the registry grows
  capabilities the rule didn't sanction (a fifth property snuck
  in), the registry no longer matches the rule. Mitigated by
  treating the registry as a specification — adding a property
  there should prompt a check against this ADR, and if the property
  isn't in the rule, either the rule needs amendment (new ADR) or
  the registry shouldn't have it.
- **Capability list is a floor, not a ceiling.** The four properties
  named here are required; further properties may be added per role
  as useful. This means roles with substantially different metadata
  can coexist in one registry, which is good for flexibility and
  bad for uniformity. Acceptable tradeoff; the alternative
  (stricter property enumeration) trades flexibility for
  consistency, and the working-unit pattern's value is in being
  recognized loosely, not enforced strictly.
- **The pattern is a shape, not a contract.** A directory that
  looks like a working-unit but isn't registered isn't bound by
  this ADR. Conversely, a registered working-unit must follow the
  capability rules, but nothing prevents someone from making
  unregistered look-alikes. This is fine; the pattern is for
  recognition and consistency, not enforcement.
- **Recursive application is allowed but not specified.** Whether
  a recursively-promoted working-unit inherits its parent's
  registry or registers fresh roles is left to the registry to
  decide per case. If recursive promotion becomes common enough
  that this ambiguity bites, a follow-up ADR can specify.

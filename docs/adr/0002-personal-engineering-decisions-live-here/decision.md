# 2. Personal engineering decisions live in this repo, not in individual workspaces

Date: 2026-05-01

## Status

Accepted

## Context

While setting up a work meta-repo (a multi-repo workspace mirroring the
upstream forge's group/org structure, with `.claude/skills/` at multiple
scopes), I
wrote ADRs documenting the structural decisions: how to organize skills,
how to sync repos across machines, how to handle repos I don't own, how
to bundle tools with skills, and so on.

The intent was to put those ADRs in `<work-meta-repo>/docs/adr/`. That
matches the conventional Nygard pattern: ADRs live in the repo whose
architecture they describe.

When I considered reusing the same setup for a separate hobby workspace,
the conventional pattern broke down. Three options surfaced:

1. **Duplicate ADRs into each workspace.** Each workspace's `docs/adr/`
   contains the same set of decisions, copied. They may diverge over
   time.
2. **Reference a shared source from each workspace.** Decisions live in
   one place, each workspace points at it. Numbering across "shared" and
   "workspace-specific" ADRs becomes ambiguous.
3. **Treat the cross-workspace decisions as a separate concern.** They
   are not really decisions about any one workspace; they are decisions
   about how I build agent workspaces in general. They belong in their
   own home, and individual workspaces can reference them.

Option 3 matches the actual scope of the decisions. The decisions about
forge-mirrored layout, `meta` for sync, bundled tools, bootstrap script
behavior, etc. are not properties of a specific workspace — they are
properties of *me* and how I work. Putting them in any single workspace
implies they are about that workspace, which makes them harder to find
and reuse, and makes their intended scope unclear to a future reader
(including future-me).

This is a slight stretch of the Nygard format, which assumes ADRs are
about a specific system. The decision to use the format anyway is
recorded in ADR 0001.

## Decision

This repository (`engineering-notes`) holds ADRs for cross-workspace
decisions about how I organize agent workspaces and personal
infrastructure. ADRs here describe decisions whose scope is "me as an
engineer," not "a specific project."

Specific workspaces (work meta-repo, hobby meta-repo, etc.) maintain
their own `docs/adr/` directories for decisions whose scope is that
workspace alone (e.g. workspace-specific secrets handling, CI setup,
team conventions). Workspace ADRs start their numbering at 0001
independently — they are not a continuation of the sequence here.

A workspace's README references this repo when shared decisions apply.
Example wording: "This workspace follows the conventions in
[engineering-notes](url), specifically ADRs 0003–0008."

## Consequences

Positive:

- Cross-workspace decisions have one canonical home. Updating a decision
  means writing one new ADR, not synchronizing copies.
- Each workspace's `docs/adr/` stays scoped to that workspace.
  Workspace-specific decisions are not lost in a sea of generic ones.
- The repo travels with me across jobs, machines, and projects without
  being tangled up in any single workspace's history.
- Sharing the conventions with someone else (a teammate, a blog post)
  has a natural unit: this repo.

Negative / accepted tradeoffs:

- **Two places to look.** A future reader of any specific workspace
  has to know that some decisions are recorded here and some in the
  workspace's own `docs/adr/`. The workspace README must make this
  explicit.
- **Numbering ambiguity if cross-references appear.** If a workspace
  ADR refers to "ADR 0003," it must be unambiguous which sequence is
  meant. Convention: workspace ADRs reference cross-workspace ADRs as
  `engineering-notes/0003`, and refer to their own as bare numbers.
- **The line between "personal" and "workspace-specific" can blur.**
  Some decisions feel general but turn out to be workspace-specific
  on closer inspection. When in doubt, start a decision in the
  workspace and promote it here later if it generalizes.

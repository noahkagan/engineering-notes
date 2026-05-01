# 4. Use `meta` to manage multi-repo cloning and updates

Date: 2026-05-01

## Status

Accepted

## Context

The workspace contains many repos that need to be cloned onto each machine
and kept up to date. Three approaches are common:

1. **Git submodules.** Built into git, no extra tooling, pins exact commits
   of child repos.
2. **`meta` (or similar manifest tools like `vcstool`, `gita`).** A JSON
   manifest of repo URLs and paths; one command clones or updates all of
   them.
3. **A bespoke setup script.** A shell script that runs `git clone` for each
   repo.

Submodules pin commits, which is valuable when child repos must be at exact
versions for reproducibility (e.g. a build that must succeed against a
known dependency tree). They impose ongoing cost: detached HEADs by default,
two-step commits when updating a child, easy to forget to push the
submodule, friction every day from active development inside child repos.

A bespoke script reinvents what `meta` already does and accumulates ad hoc
features (parallel pulls, status across repos, exec-in-each) until it is a
worse `meta`.

The repos in this workspace are actively-developed working copies, not
pinned dependencies. Reproducibility of exact commits is not a goal;
"have all my repos cloned and current" is.

## Decision

Use [`meta`](https://github.com/mateodelnorte/meta) to manage the set of
repos in the workspace. Repo locations and URLs live in a `.meta` file at
the workspace root. The bootstrap script installs `meta` if missing and
runs `meta git clone` to populate the workspace.

## Consequences

Positive:

- Active development inside child repos has no extra friction; each repo is
  a normal git clone.
- `meta git status`, `meta git update`, and `meta exec` give workspace-wide
  operations without custom scripting.
- The `.meta` file is plain JSON — trivial to migrate to another tool, or
  to read with a one-liner if `meta` itself becomes unavailable.

Negative / accepted tradeoffs:

- Requires installing `meta` (a Node package) on each machine. The bootstrap
  script handles this but adds a Node dependency to the setup path.
- No commit pinning. If a workspace-wide change needs all repos at specific
  commits to work, that must be coordinated externally (e.g. by tagging).
- One more tool to learn. Mitigated by the fact that 90% of daily use is
  `meta git update` and `meta git status`.

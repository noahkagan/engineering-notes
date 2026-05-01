# Architecture Decision Records

Decisions about how I organize agent workspaces and personal infrastructure.
Format: [Michael Nygard's ADR template](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions),
chosen in [ADR 0001](0001-use-nygard-adr-format.md).

These ADRs describe decisions whose scope is "me as an engineer," not any
single project. See [ADR 0002](0002-personal-engineering-decisions-live-here.md)
for why this repo exists separately from any specific workspace.

Workspace-specific decisions (work meta-repo, hobby meta-repo, etc.) live
in those workspaces' own `docs/adr/` directories and start their numbering
at 0001 independently.

ADRs are immutable once their status is Accepted. Revisions are made by
writing a new superseding ADR and updating the old one's status.

## Index

- [0001 — Use Michael Nygard's ADR format to record decisions](0001-use-nygard-adr-format/decision.md)
- [0002 — Personal engineering decisions live in this repo, not in individual workspaces](0002-personal-engineering-decisions-live-here/decision.md)
- [0003 — Workspace mirrors the upstream forge's group/org structure](0003-mirror-forge-structure/decision.md)
- [0004 — Use `meta` to manage multi-repo cloning and updates](0004-meta-tool-for-multirepo/decision.md)
- [0005 — Skills for repos we don't own live at the parent group scope](0005-skills-for-unowned-repos/decision.md)
- [0006 — Personal universal skills live in a separate repo, symlinked into `~/.claude/skills`](0006-personal-universal-skills/decision.md)
- [0007 — Tools that a skill depends on are bundled with the skill](0007-bundle-tools-with-skills/decision.md)
- [0008 — Bootstrap script is a single staged bash script delegating per-skill installs](0008-bootstrap-script-design/decision.md)
- [0009 — Personal decisions follow Nygard ADR format regardless of project shape](0009-personal-decisions-nygard-regardless/decision.md)
- [0010 — Cross-repo work items live in personal trackers; per-repo items reference upstream](0010-cross-repo-issue-tracking/decision.md)
- [0011 — Task slugs use date-anchored format with optional tracker ID](0011-task-slug-convention/decision.md)
- [0012 — Per-task artifacts live in `scratch/<slug>/` parallel to `TODO.md`](0012-task-artifact-directories.md)

## Cross-references

When ADRs in workspace repos reference these, they should use the form
`engineering-notes/NNNN`. References within this repo use bare numbers.

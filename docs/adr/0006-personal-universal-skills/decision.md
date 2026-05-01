# 6. Personal universal skills live in a separate repo, symlinked into `~/.claude/skills`

Date: 2026-05-01

## Status

Accepted

## Context

Some skills are personal preferences that apply everywhere we work, not just
inside the work meta-repo. Examples: commit-message style, language-specific
conventions we like regardless of project, debugging habits. These need to
load when working anywhere on the machine, including in `~/code/<weekend
project>` and other directories outside `~/work`.

Claude Code loads skills from `~/.claude/skills/` as "personal" scope,
applying them across all projects on the machine. Personal skills override
project skills when names collide; enterprise overrides personal. Skill
directories are watched for live changes, so symlinked content stays in
sync as we `git pull` the source repo.

Two structural questions:

1. Should personal universal skills be folded into the work meta-repo or
   kept in their own repo?
2. How do they get into `~/.claude/skills/` on each machine?

On (1): personal skills have a different lifecycle from work meta-repo
contents. The meta-repo churns when teams reorganize, repos move between
groups, or projects come and go. Personal preferences change when *we*
change. Coupling them means every personal-preference change touches the
work meta-repo's history, and every team reshuffle touches the personal
repo's history. They also need to load outside `~/work`, which the
meta-repo's nested-discovery mechanism cannot provide.

On (2): two symlink strategies were considered.

- **Whole-directory symlink.** `~/.claude/skills` -> `~/personal-skills/skills/`.
  One link, anything in the source repo appears immediately.
- **Per-skill symlinks.** `~/.claude/skills/<name>` -> `~/personal-skills/skills/<name>/`,
  one link per skill. Allows mixing repo-managed skills with one-off skills
  added directly to `~/.claude/skills/`.

The mixing case is rare in practice — anything worth keeping is worth
committing — and the per-skill approach requires a bootstrap step that
walks the repo and creates each link, plus logic to clean up dangling
links when skills are removed. The whole-directory symlink is simpler and
the failure modes are clearer (the link is either correct or obviously not).

## Decision

Personal universal skills live in a dedicated repo at `~/personal-skills`,
separate from the work meta-repo. The repo's `skills/` directory is
symlinked in its entirety to `~/.claude/skills`:

```
~/.claude/skills  ->  ~/personal-skills/skills/
```

Each machine's setup creates this symlink once. The work meta-repo's
bootstrap script optionally invokes `~/personal-skills/install.sh` if the
personal-skills repo is present, but the two repos are independent and
either can be set up without the other.

```
~/personal-skills/                    # separate git repo
  install.sh                          # creates the symlink
  skills/
    commit-style/
      SKILL.md
    rust-conventions/
      SKILL.md
    ...
```

## Consequences

Positive:

- Personal skills load everywhere, including outside `~/work`.
- Personal-skill changes and work-structure changes have independent
  histories. Switching jobs, getting added to new groups or orgs, or
  restructuring teams doesn't touch personal skills.
- Live change detection means `git pull` in the personal-skills repo
  propagates to running Claude Code sessions without restart.
- One symlink per machine; trivial to verify (`readlink ~/.claude/skills`).

Negative / accepted tradeoffs:

- Two repos to clone on a new machine instead of one. Acceptable; the
  bootstrap script handles both.
- Cannot mix `~/.claude/skills/` with one-off skills not committed to the
  personal repo. If we want to experiment with a skill, it must live in
  the repo (or in a project-scoped `.claude/skills/`). Treated as a
  feature, not a bug: it forces every personal skill to be version-
  controlled.
- **Precedence collision risk.** Personal skills override project skills
  when names match. A personal `commit-style` will silently win over a
  workspace `commit-style`, even when working inside `~/work`. To avoid
  surprises:
  - Keep personal skill names generic and broadly applicable (e.g.
    `commit-style`, `rust-conventions`).
  - Use specific, scope-prefixed names for work skills that should not
    be overridden (e.g. `platform-commit-style` rather than `commit-style`).
  - When intentional override is desired (work conventions should win
    over personal preferences while at work), this is the wrong layout
    and a different mechanism is needed. Revisit if it comes up.
- Pre-existing contents of `~/.claude/skills/` on a machine will be
  destroyed when the symlink is created. The install script must check
  for this and refuse to overwrite, requiring manual migration.

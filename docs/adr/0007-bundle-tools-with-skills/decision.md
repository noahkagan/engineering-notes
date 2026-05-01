# 7. Tools that a skill depends on are bundled with the skill

Date: 2026-05-01

## Status

Accepted

## Context

Some skills require a custom tool (a CLI, a script, a small binary) to
function. The tool has to be installed on each machine where the skill is
used. There are two natural places for it to live:

1. **Bundled with the skill.** The tool's source lives in the same
   directory as the skill. The skill's setup step installs the tool.
2. **Separate tools repo.** Tools live in their own repo, and skills that
   depend on a tool declare the dependency and install it from there.

The separated layout is appealing when tools are reused across skills, or
when a tool has a life of its own outside skill invocations (we sometimes
run it directly from the shell). It pays for itself with reduced
duplication and a single canonical home for each tool.

The bundled layout is appealing when each tool exists to support exactly
one skill — the tool and the skill are one logical unit, and splitting
them across repos creates ceremony for no gain. It also keeps the skill
self-contained: cloning the skill's repo and running its setup script
gets you everything needed.

In practice, the tools we have are skill-specific helpers, not
general-purpose CLIs that happen to be invoked by skills. They exist
because a skill needed them. If a tool ever does grow into a
general-purpose utility used by multiple skills (or used directly from
the shell often enough to deserve its own home), that is a signal to
extract it — but extraction is cheaper than premature separation.

## Decision

A tool that exists to support a skill lives in the same directory as
the skill. The skill's `SKILL.md` documents how to install the tool, and
the skill's directory contains a setup script (or equivalent) that
performs the install.

Layout:

```
<skill-name>/
  SKILL.md                # references the tool, points at install
  install.sh              # builds and installs the tool
  tool/                   # tool source (or a top-level file like main.py
                          # for single-file tools)
    ...
```

If a tool grows into something used by multiple skills or invoked
directly outside skill workflows, extract it into a dedicated tools
repo at that point. Until then, it stays bundled.

## Consequences

Positive:

- Skills are self-contained units. Clone the skill, run setup, done.
- No "which repo has this tool" ambiguity. The skill that needs the tool
  owns it.
- The bootstrap path on a new machine is simpler: one repo per skill,
  one install step per skill, no cross-repo dependency resolution.
- Removing a skill removes its tool too — no orphaned binaries living
  in a separate tools repo.

Negative / accepted tradeoffs:

- **Duplication when a tool is genuinely useful to multiple skills.**
  If skill A and skill B both need the same tool, we have to either
  duplicate it (bad) or designate one skill's repo as the canonical
  home and have the other depend on it (which is the separated layout
  with worse ergonomics). When this case arises, extract the tool to a
  dedicated repo and supersede this ADR for that tool.
- **No shared install machinery.** Each skill carries its own setup
  script. If we accumulate many skills with similar install logic
  (e.g. ten Rust tools all built the same way), we'll end up
  copy-pasting `cargo install --path .` into ten places. Acceptable
  for now; if it becomes painful, factor out a shared install helper
  invoked by each skill's setup script.
- **Tool versioning is implicit.** Each skill's tool is at whatever
  version the skill repo's HEAD has. There is no way to pin a specific
  tool version while updating skill instructions, or vice versa,
  without splitting them anyway.
- **Tools meant to be invoked directly from the shell live in awkward
  places.** A user who wants to run the tool standalone has to know
  it's in `~/some-skill-repo/tool/` rather than somewhere on `$PATH`.
  Mitigated by having the install script symlink the tool into a
  standard location (`~/.local/bin/`) where appropriate.

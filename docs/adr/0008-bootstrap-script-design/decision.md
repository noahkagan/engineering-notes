# 8. Bootstrap script is a single staged bash script delegating per-skill installs

Date: 2026-05-01

## Status

Accepted

## Context

A new machine needs a way to go from "fresh git clone of the meta-repo" to
"all repos cloned, all skill-bundled tools installed, personal-skills repo
linked if present." The bootstrap script is the single entry point for that.

Bootstrap scripts are easy to start and hard to keep healthy. Convergent
practice across mature dotfiles repositories surfaces a small set of
properties that consistently pay off, and a few common mistakes worth
avoiding by design.

Properties that converge across the field:

- **Idempotency is treated as non-negotiable.** Bootstraps get re-run when
  new repos are added, when something is discovered missing, when debugging
  a teammate's broken setup. A non-idempotent bootstrap becomes "I'll do
  that part by hand" within weeks.
- **Stages over monoliths.** Mature scripts decompose into named, ordered
  stages. When stage 3 fails, the user fixes the cause and re-runs from
  stage 3 — not from the top, not by reading the script to figure out what
  ran already.
- **Detect-then-act, not action logs.** Each step asks "is the desired
  state present?" rather than "did I run this before?" State checks
  compose correctly across re-runs and across machines that started from
  different initial conditions.
- **Refuse to clobber.** When the script finds something it didn't put
  there — a pre-existing target file, a directory with content where a
  symlink should go — it skips with a message, not overwrites.
- **Per-component installers, not centralized knowledge.** Each skill that
  needs setup carries its own installer (per ADR 0007). The bootstrap's
  job is finding and invoking them in a consistent order, not knowing
  how each one works.

Divergence in the field is mostly about tooling choice (bash vs. chezmoi
vs. mise vs. ansible). Heavy tooling pays off for templated dotfiles
varying by machine, secret management, and tool-version pinning across
languages. None of that applies here: this is a workspace structure,
not a dotfile setup. Bash is sufficient and lower-overhead.

## Decision

A single `bootstrap.sh` at the meta-repo root, decomposed into named
stages. Bash, no additional tooling. Each stage is internally idempotent
and can be re-run safely.

Stages, in order:

1. **Preflight.** Check that required commands (`git`, `node`, `npm`)
   exist. Refuse with a clear message if anything is missing. Do not
   attempt to install toolchains; that is the user's responsibility per
   machine.
2. **Install `meta`.** `npm install -g meta` if not already present.
3. **Clone repos.** Run `meta git update`, which clones missing repos
   and pulls existing ones.
4. **Run per-skill installers.** For each `.claude/skills/*/install.sh`
   under the meta-repo, execute it. Each installer is responsible for
   its own idempotency.
5. **Link personal-skills repo (optional).** If `~/personal-skills/`
   exists, create or verify the symlink at `~/.claude/skills`. Skip the
   stage entirely if `~/personal-skills/` is absent.

Script-level behaviors:

- `set -euo pipefail` at the top. Fail loud on errors, unset variables,
  and broken pipes.
- Every stage prints its name on entry and a one-line outcome on exit.
- Failure in any stage stops the script and reports which stage failed.
- Flags supported:
  - `--dry-run` — print what would happen without doing it.
  - `--only=<stage>` — run a single named stage and exit.
  - `--skip=<stage>[,<stage>...]` — skip the named stages.
- The script never invokes `sudo`. If a per-skill installer needs root,
  it prompts at the moment it needs it.

Behaviors deliberately excluded:

- No auto-update mechanism, no daily update checks, no telemetry. These
  are dotfile-tooling concerns and add code that breaks.
- No bootstrap-time toolchain installation (Rust, Python, Node version
  managers, etc.). Toolchain management is a separate concern; per-skill
  installers may require specific toolchains and should fail clearly if
  they're missing.
- No secrets handling. Workspace setup does not manage credentials.

## Consequences

Positive:

- Re-running the bootstrap on an existing machine is safe and useful.
  Adding a new skill repo to the manifest and re-running picks it up
  with no special handling.
- A failed stage is recoverable: fix the underlying cause, re-run, and
  the earlier stages no-op while the failed stage retries.
- Per-skill installers stay close to the skills they install. The
  bootstrap doesn't accumulate knowledge of every skill's setup logic.
- Bash with `set -euo pipefail` and named stages stays under ~150 lines.
  Auditable in a single sitting.
- `--dry-run` and `--only=` make the script debuggable from the outside
  rather than requiring code reading.

Negative / accepted tradeoffs:

- **No cross-machine variation.** A bash script with positional checks
  doesn't handle "this stage runs differently on macOS than on Linux"
  as gracefully as chezmoi templates would. If we accumulate enough
  OS-specific logic to feel painful, revisit. So far the only OS-aware
  step is in per-skill installers, which can shell out to `uname` if
  needed.
- **Per-skill installer quality varies.** The bootstrap trusts each
  installer to be idempotent. A buggy installer that, say, appends
  to a config file every run will compound across re-runs. Mitigated
  by convention (every installer must be re-run-safe) and by review,
  not by tooling.
- **No transactional rollback.** If stage 4 fails halfway, the
  filesystem is in a partial state. Re-running converges, but there is
  no "undo." Acceptable for a personal-machine setup; not appropriate
  for production deployment, but production deployment is not the use
  case.
- **Preflight refuses rather than installs.** A teammate using this on
  a fresh machine still has to install `node` and `git` themselves.
  Documented in the meta-repo README; the alternative (auto-installing
  toolchains across OSes) is a maintenance burden out of proportion to
  the benefit.

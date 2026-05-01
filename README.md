# engineering-notes

Personal engineering decisions and conventions, recorded as ADRs.

This repo holds decisions whose scope is "how I work as an engineer," not
decisions about any specific project. Things like how I organize agent
workspaces, how I sync repos across machines, how I structure skill repos.

Specific projects (work meta-repo, hobby meta-repo, individual code
repos) keep their own `docs/adr/` directories for project-specific
decisions. They reference this repo when shared conventions apply.

See [`docs/adr/`](docs/adr/) for the decision records themselves.

## Why this exists

See [ADR 0002](docs/adr/0002-personal-engineering-decisions-live-here/decision.md). The
short version: ADRs in the Nygard format are conventionally about a
specific system. The decisions in this repo are about *me*, not about
any one system, and they apply across multiple workspaces. Putting them
in any single workspace would have implied they were scoped to that
workspace, which they aren't.

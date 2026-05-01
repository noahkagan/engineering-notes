# 3. Workspace mirrors the upstream forge's group/org structure

Date: 2026-05-01

## Status

Accepted

## Context

Skills, custom tools, and per-project notes need a home that travels between
machines and that the agent can discover automatically. Skills vary along
three independent axes: scope of applicability (universal / group / single
repo), ownership of the target repo (mine vs. someone else's), and whether
they require a custom tool to function.

Claude Code discovers skills via nested `.claude/skills/` directories, walking
parent directories from wherever the session is launched. This means the
filesystem layout *is* the scoping mechanism — there is no separate
configuration step to declare which skills apply where. Whatever directory
structure we pick will be both the human organizational model and the
agent's activation rule.

The upstream forge — GitLab groups, GitHub organizations, Bitbucket
workspaces, Gitea organizations, or whatever else hosts the repos — already
encodes the organizational reality of the work: which team owns what,
which repos are siblings, which are unrelated. Inventing a parallel
structure (e.g. flat `~/skills/` with naming conventions) would mean
translating between two mental models every time we sit down to work.

## Decision

The workspace root is `~/work` (or another root per workspace, e.g.
`~/hobby`). Inside it, the directory tree mirrors the upstream forge's
group/organization hierarchy. Each level may contain a `.claude/skills/`
directory whose skills apply to everything beneath it.

```
~/<workspace>/                       # workspace root
  .claude/skills/                    # applies everywhere
  <group-or-org>/
    .claude/skills/                  # applies to this group/org
    <repo>/
      .claude/skills/                # applies to this repo (only when we own it)
```

Group/organization names and repo names match the forge exactly. No
translation layer.

If a workspace draws repos from multiple forges, each forge gets its own
top-level subdirectory under the workspace root (e.g.
`~/hobby/github.com/<user>/<repo>` and `~/hobby/gitlab.com/<user>/<repo>`).
A workspace drawing from only one forge can omit the forge-name level for
brevity.

## Consequences

Positive:

- Skill scope is determined by location, with no separate registration step.
- "Where do skills for X live" answers itself from the upstream path.
- Group-level skills (workflows for a multi-repo set) get a natural home that
  was missing from a flat skill-repo layout.
- New machines reproduce the structure by cloning the workspace, not by
  configuring an agent.
- The pattern works the same for work and hobby, regardless of which
  forge each uses.

Negative / accepted tradeoffs:

- Personal universal skills (e.g. commit-message style) do not belong here;
  they must live in `~/.claude/skills/` so they load outside any specific
  workspace. This is a separate concern, addressed in ADR 0006.
- A repo that legitimately belongs to multiple groups or organizations
  (forks, shared libraries that appear under several teams) gets one
  canonical home and the others must be handled case by case.
- Renaming a group or organization upstream requires renaming the local
  directory to keep the mirror honest. Acceptable; renames are rare.
- For workspaces drawing from a single forge, the choice of whether to
  include the forge name at the top level (`~/work/gitlab.com/<group>/<repo>`)
  versus omitting it (`~/work/<group>/<repo>`) is left to the workspace.
  Both are consistent with this ADR. Including the forge name future-proofs
  against later adding a second forge; omitting it is shorter today.

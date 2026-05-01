# 5. Skills for repos we don't own live at the parent group scope

Date: 2026-05-01

## Status

Accepted

## Context

Some repos in the workspace are owned by other teams. Personal skills for
working in those repos (notes on their conventions, gotchas, workflows we
prefer) must be version-controlled with the rest of our skills, but must
never appear in their git history.

Three options were considered:

1. **Inside their repo, gitignored locally.** Place `.claude/skills/` inside
   the unowned repo and add it to a local gitignore.
2. **Parallel path at the parent group level.** Place skills one directory
   up from their repo, in the group's `.claude/skills/`.
3. **Separate skill repo, symlinked in.** Maintain skills in a dedicated
   repo and symlink them into place.

Option 1 is fragile: a `git clean -fdx` deletes the skills, a misconfigured
gitignore commits them, and the skills are not version-controlled with the
rest of the workspace. Option 3 requires maintaining symlinks across
machines, which the bootstrap script can do but adds moving parts.

Claude Code's nested skill discovery walks parent `.claude/skills/`
directories from the session's working directory. A skill placed at
`<group>/.claude/skills/` is therefore picked up when working inside any
repo within that group, including unowned ones.

## Decision

For a repo `<group>/<their-repo>/` that we do not own, personal skills for
working in it live at:

```
~/work/<group>/.claude/skills/<their-repo>-notes/
```

The `-notes` suffix in the skill directory name disambiguates it from
skills that apply to the entire group.

## Consequences

Positive:

- The unowned repo is touched in zero ways. No gitignore entries, no stray
  files, no risk of accidental commits.
- The skills are version-controlled in the workspace meta-repo alongside
  everything else.
- Nested discovery loads them automatically when we `cd` into the unowned
  repo and launch the agent.

Negative / accepted tradeoffs:

- Skills named `<their-repo>-notes/` load whenever the agent is launched
  anywhere under `<group>/`, not exclusively when inside `<their-repo>/`.
  In practice, group scopes are small enough that this is fine; if it
  becomes noisy, the skills can be split with more specific descriptions
  so the agent only invokes them when relevant.
- The naming convention (`<their-repo>-notes/`) is just a convention; it
  is not enforced by tooling. Reviewers must catch violations.
- If we later gain ownership of the repo, the skills must be moved into
  the repo's own `.claude/skills/`. This is a one-time `git mv`.

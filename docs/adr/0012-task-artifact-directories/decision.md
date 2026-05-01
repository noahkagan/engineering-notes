# 12. Per-task artifacts live in `scratch/<slug>/` parallel to `TODO.md`

Date: 2026-05-01

## Status

Accepted

## Context

`TODO.md` at the workspace root (per ADR 0010) tracks cross-cutting work
that spans multiple repos. It works well for things that fit in
markdown bullets — what to do, status, open questions, links to
upstream issues. It breaks down when a task accumulates artifacts that
don't fit inline: scratch scripts written to investigate something,
output data (CSVs, logs, screenshots), longer-form notes that have
outgrown a bullet point, abandoned attempts worth keeping for "we tried
this, here's why it didn't work."

Several places these artifacts could go, each with problems:

1. **Inline in `TODO.md`.** Code blocks and base64 attachments make
   `TODO.md` unreadable past the first artifact. Doesn't work for
   binary data at all.
2. **Inside one of the repos the task touches.** Cross-cutting work
   doesn't belong in any single repo, and committing scratch artifacts
   to a repo's main branch pollutes its history. Putting them on a
   feature branch ties the artifacts' lifecycle to the branch's, and
   they get lost when the branch merges or is deleted.
3. **Outside the workspace entirely** (`~/scratch/`, `~/Documents/`,
   etc.). The artifacts aren't version-controlled with the work,
   don't travel between machines, and aren't discoverable by future-me
   looking back through the workspace.
4. **Workspace root, sibling to `TODO.md`.** Fine for one or two ad-hoc
   files. Becomes noise once there are several active tasks each with
   artifacts, because the root directory's listing fills up and
   visually competes with `TODO.md` and `bootstrap.sh`.

The pattern that emerges: artifacts belong in the workspace (so they're
version-controlled and travel with the work), but scoped to a task (so
they don't pollute other tasks' contexts), with a directory layout that
keeps the workspace root scannable.

The slug convention from ADR 0011 already gives every task a stable
identifier. Reusing that slug as a directory name under `scratch/`
gives artifacts a home that's consistent with how tasks are named
elsewhere — same string in `TODO.md`, in `scratch/`, and (where
practical) in branch names.

## Decision

Per-task artifacts live in `scratch/<slug>/` at the workspace root,
where `<slug>` follows the format from ADR 0011.

```
~/work/
  TODO.md                                   # task index
  scratch/
    2026-05-01-HP-123-payment-bug/
      README.md                             # task notes
      analyze.py                            # investigation script
      results.csv                           # output data
    2026-05-01-billing-investigation/
      README.md
      ...
```

The task directory's `README.md` (or `NOTES.md`, or whatever name fits
locally) holds the task's notes when they outgrow the `TODO.md` bullet,
and lists cross-references to upstream tracker IDs (per the multi-
tracker case in ADR 0011). `TODO.md`'s entry for the task links to the
directory rather than holding everything inline.

Tasks without artifacts stay inline in `TODO.md` and don't get a
directory. The `scratch/` directory is created lazily, when a task
earns it. Most tasks won't.

The `scratch/` directory is committed to the workspace meta-repo. It
is not added to `.gitignore`. Per-task directories are 
moved to a `scratch/archive/` subdirectory when their tasks complete,
to keep `scratch/` from accumulating ghosts. The `scrach/archive`
directory is cleaned manually.

## Consequences

Positive:

- Artifacts have a clear home that's version-controlled, travels
  between machines, and is discoverable from the workspace root.
- The slug consistency means searching for `HP-123` finds the
  `TODO.md` entry, the `scratch/` directory, and any branch or PR
  that uses the same slug. One anchor, multiple surfaces.
- `TODO.md` stays scannable for the common case (most tasks fit
  inline) while supporting heavier tasks via links to their
  directories.
- Promotion is gradual. Inline `TODO.md` entry → entry plus a
  `scratch/<slug>/` directory → entry pointing at a directory whose
  `README.md` is the real task notes. Each step is a small change
  with no big-bang restructuring.

Negative / accepted tradeoffs:

- **Cleanup discipline is required.** `scratch/` will accumulate
  completed-task directories if not actively pruned. The mitigation —
  remove or archive when tasks complete — depends on me remembering to
  do it. A buggy version of this decision might result in `scratch/`
  becoming a dumping ground that's never cleaned. Periodic sweeps
  (quarterly?) are the realistic answer; automation is overkill for
  personal scratch state.
- **Artifacts are committed to the workspace meta-repo, including
  binary data.** Large binaries bloat git history. Mitigations: don't
  commit very large files (use git-lfs if you must, or keep them
  outside the workspace), be defensive about secrets accidentally
  committed in artifacts (`.gitignore` patterns for `.env`,
  `*credentials*`, `*.pem` at the workspace level), and treat
  `scratch/` as committed-but-disposable (squashing or pruning
  history of removed directories is fine).
- **No structure inside the per-task directory.** Each task's
  `scratch/<slug>/` is a free-form working area. No convention for
  where investigation scripts go vs. output data vs. notes. Acceptable
  because the directory is task-scoped and small; over-structuring
  per-task layouts would be premature.
- **Cross-task artifacts have no home.** A script useful to multiple
  tasks doesn't fit cleanly in any single `scratch/<slug>/`. If this
  comes up, the answer is to promote the script out of `scratch/` into
  a workspace-level `bin/` or `tools/` directory, or into a proper
  skill if it's repeatable enough. Not a `scratch/shared/` directory —
  shared things have outgrown scratch.
- **The `README.md` filename inside task directories is convention,
  not enforced.** Some tasks may use `NOTES.md` or `STATUS.md` or
  whatever fits. Searches for "the task notes" need to handle this
  variance. Mitigated by not over-specifying the filename; the
  directory's existence and slug already say what it's for.
# 10. Cross-repo work items live in personal trackers; per-repo items reference upstream

Date: 2026-05-01

## Status

Accepted

## Context

Workspaces routinely contain repos with different issue-tracking
conventions. One repo uses GitHub Issues, another Jira, another a
`TODO.md` at the root, another nothing at all. Some repos I own and can
add tracking to; others I don't and shouldn't.

When work spans multiple repos — a feature that touches three services,
a refactor that ripples through five libraries — the natural question is
where to track it. Three options:

1. **Track everything in one upstream system.** Pick one repo's tracker
   and file all cross-repo work there. Doesn't work when I don't own all
   the repos, and creates ambiguity about which repo "owns" each item.
2. **Build a personal meta-tracker.** A system that ingests issues from
   each upstream and presents a unified view. This is a third system to
   maintain. It always lags reality and breaks when upstreams change
   their APIs or schemas. Most attempts I've seen end up unmaintained
   within months.
3. **Track cross-cutting work in a personal tracker; per-repo work in
   the repo's upstream system, referenced when relevant.** Personal
   tracker captures the things that don't fit anywhere else — work
   spanning repos, my own ongoing concerns, scratchpad items. Upstream
   trackers stay authoritative for their own repos.

Option 3 accepts that some duplication is unavoidable: a cross-cutting
item may exist as one entry in my personal tracker and as filed issues
in two upstream trackers, and I keep them roughly in sync by hand. This
sounds worse than it is in practice, because cross-cutting items are a
small fraction of total work and the manual sync is an opportunity to
make sure the upstream items actually capture the right thing.

## Decision

For each workspace, maintain a personal tracker file at the workspace
root (e.g. `TODO.md` or `tasks.md`). Use it for:

- Work items that span multiple repos in the workspace.
- My own cross-cutting concerns that don't belong in any single repo's
  tracker.
- Scratchpad items I haven't decided where to file yet.
- References to upstream issues I'm tracking, when I want them visible
  in one place without duplicating their content.

For work items that belong cleanly to a single repo, use that repo's
upstream tracker (GitHub Issues, Jira, the repo's `TODO.md`, whatever
the project uses). Reference upstream items from the personal tracker
by URL or ID rather than copying their content.

The personal tracker is intentionally lightweight — a markdown file,
not a system. It is not synced with upstream trackers and makes no
attempt to be authoritative about anything but itself.

## Consequences

Positive:

- Cross-cutting work has a home that doesn't require choosing one
  upstream tracker over another, or owning a repo I don't own.
- Per-repo work stays in the repo's own tracker, where collaborators
  can see and act on it.
- The personal tracker is plain markdown, version-controlled with the
  workspace, and has no infrastructure to maintain.
- The system degrades gracefully. If I stop using the personal tracker
  for a few weeks, nothing breaks; the upstream trackers continue to
  work.

Negative / accepted tradeoffs:

- **Manual sync for genuinely cross-cutting items.** When one logical
  work item exists as a personal-tracker entry plus filed issues in
  two upstream trackers, keeping their statuses aligned is on me.
  Acceptable because the items are few; mitigated by writing personal-
  tracker entries that reference upstream URLs so I can always find
  the canonical state.
- **No unified view across workspaces.** A work-workspace TODO.md and
  a hobby-workspace TODO.md are independent. There is no place where
  "everything I'm working on" appears unified. Accepted; the alternative
  is a meta-tracker (option 2) and that has worse failure modes.
- **The personal tracker can drift from being useful.** If I dump
  everything into it without curation, it becomes noise. Mitigated by
  periodic pruning, treating items past a certain age as either done,
  abandoned, or worth filing properly upstream.
- **No tooling for the format.** A markdown file with bullet points has
  no native query, filter, or status-tracking. If I find myself wanting
  those, the file is signaling that something belongs upstream and I
  should file it there. Treating the lack of tooling as a feature.

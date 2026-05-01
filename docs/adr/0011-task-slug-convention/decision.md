# 11. Task slugs use date-anchored format with optional tracker ID

Date: 2026-05-01

## Status

Accepted

## Context

Cross-cutting tasks need a slug that identifies them across `TODO.md`,
`scratch/` directories, branch names, and notes. The slug is the task's
identity for human and tooling purposes; consistency across these
surfaces means a single grep finds everything related.

Engineering practice has converged on ticket-ID-anchored slugs:
`<TICKET-ID>-<kebab-description>` like `HP-123-payment-bug`. This works
when every task has a stable canonical ID. It fails in three ways for
the workspaces I actually work in:

1. **Mixed trackers per task.** A cross-repo task may have a Jira
   ticket for the backend, a GitHub issue for the frontend, and a
   Linear ticket for design. None is the canonical ID; each is the
   canonical ID *for its repo*. Picking one for the slug is arbitrary.
2. **Tasks without a tracker.** Exploratory work, investigations, or
   self-assigned tasks may have no ID anywhere. The pure-ID convention
   has no fallback for these and people invent ad hoc slugs that
   don't compose with the ID-based ones.
3. **Visual inconsistency.** Even when most tasks have IDs, mixing
   `HP-123-bug/` and `2026-05-01-investigation/` in the same scratch
   directory makes scanning harder. The two slug shapes look like
   different categories of thing when they're really the same kind of
   thing with different metadata.

The Zettelkasten community solves a related problem with timestamp IDs:
every note gets a date-based identifier that's always available, never
collides, and gives chronological sort for free. This works because
"when" is more stable than "what" — a note's topic may evolve, but the
date it was created doesn't.

Applying the same logic to task slugs: the *date* a task started is
always available and stable. Tracker IDs, when they exist, are useful
metadata but not the most stable anchor. Inverting the priority — date
as primary anchor, tracker ID as optional metadata — handles all three
failure modes of the pure-ID convention.

## Decision

Task slugs use the format:

```
<YYYY-MM-DD>[-<tracker>-<id>]-<short-kebab-description>
```

The leading ISO date is always present. Tracker abbreviation and ID
are present when the task has one in a tracker, and the bare ID isn't
self-disambiguating. Description is short, kebab-case, three to five
words.

Tracker abbreviations:

- No prefix needed for trackers whose ID is self-identifying (Jira:
  `HP-123`, `JIRA-456` — the project key already disambiguates).
- `gh` for GitHub Issues, where the bare number is ambiguous.
- `lin` for Linear; other trackers add their own short prefixes as
  needed.

When a task is referenced by multiple trackers, pick the one most
useful as the primary anchor for the slug, and record the others
inside the task's directory (in `README.md` or equivalent). The
directory name is one slug; cross-references are full-text-searchable
inside.

Same-day collisions are resolved with a letter suffix on the date:
`2026-05-01-a-foo`, `2026-05-01-b-bar`. Don't use letters
pre-emptively; only when an actual collision occurs.

## Consequences

Positive:

- Every task has the same slug shape regardless of tracker situation.
  Scanning a scratch directory shows a uniform list, not a mix of
  visually distinct categories.
- The leading timestamp gives chronological sort for free. `ls scratch/
  | sort` is meaningfully ordered.
- Date-anchored slugs are stable through topic drift. A task that
  starts as "payment bug" and becomes "billing issue" can have its
  description rewritten without losing the slug's identity — the date
  is unchanged. (Though renaming the directory still requires updating
  references to it; the *anchor* is stable, the directory name still
  changes if the description changes.)
- The format degrades gracefully. Tasks with rich metadata (tracker +
  ID + description) carry it all; tasks with no metadata still get a
  valid slug from date + description.
- Cross-tracker tasks (one logical task, multiple tracker IDs) have an
  obvious treatment: pick one for the slug, the rest go inside.
- Searches still work the way ID-based slugs allowed. `grep HP-123`
  finds the slug; `grep gh-456` does too. The ID is in the slug, just
  not the prefix.

Negative / accepted tradeoffs:

- **Slugs are longer.** `2026-05-01-HP-123-payment-bug` is 30
  characters versus `HP-123-payment-bug` at 18. Real cost in typing,
  display width, and visual scan. Acceptable; tab completion handles
  most of the typing burden.
- **Tracker ID is no longer the visually dominant element.** When
  scanning, the eye lands on the date first, not the ticket number.
  For workflows where "I need to find the directory for HP-123 right
  now" is common, this is slower than ID-led slugs. Mitigated by
  search rather than visual scan.
- **Day-granularity has rare collisions.** Two tasks started on the
  same day need disambiguation. The letter-suffix mitigation is fine
  but adds a small special case ("am I on a colliding day?"). Hour-
  precision would eliminate the issue but at the cost of much longer
  slugs for a rare problem; not worth it.
- **The tracker abbreviation set is a convention I have to maintain.**
  `gh`, `lin`, no-prefix-for-Jira: this is a small vocabulary I have
  to remember and apply consistently. If I'm inconsistent (`gh` here
  and `github` there), search breaks. Mitigated by listing the
  abbreviations in `workspace-conventions` skill so the agent can
  apply them consistently and reviewers (including future-me) can
  catch drift.
- **The format implies same shape across workspaces.** If a different
  workspace has wildly different slug needs, the format may not fit.
  This ADR scopes itself to "how I name tasks across my workspaces"
  and not to "how project teams should name tasks"; project conventions
  for branches and PRs may differ and should win where they apply.
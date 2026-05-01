# 1. Use Michael Nygard's ADR format to record decisions

Date: 2026-05-01

## Status

Accepted

## Context

Personal engineering decisions need a written form. Without one, rationale
evaporates: in six months I won't remember why I picked `meta` over
submodules, and I'll either redo the analysis or guess. With a written
form but no conventions, decisions accumulate as a tangle — rationale gets
edited mid-stream, supersession is implicit, and individual decisions
can't be revised without disturbing surrounding ones.

Several formats were plausible:

- **Free-form notes.** A document or wiki where decisions are written in
  whatever shape feels right. Easy to start, but decisions tangle with
  each other, rationale gets lost when text is edited, and there is no
  convention for marking a decision as superseded.
- **Michael Nygard's ADR format.** A short markdown document per decision,
  with named sections: Title, Status, Context, Decision, Consequences.
  Decisions are immutable once accepted; revisions are made by writing a
  new superseding ADR. Proposed in 2011 and widely adopted since.
- **MADR (Markdown Any Decision Records).** An extension of Nygard's
  format with more structure: explicit "Considered Options" and "Decision
  Drivers" sections, frontmatter metadata. More rigorous but heavier per
  decision.
- **Y-statements.** A one-line template ("In the context of X, facing Y,
  we decided Z to achieve W, accepting Q"). Compact but doesn't carry
  enough rationale for non-obvious decisions.

The decisions in this repo are personal — there is no team consensus to
build, no architecture review board to satisfy, no compliance trail to
maintain. The format needs to do three things: force me to be explicit
about tradeoffs at decision time, give future-me enough context to
understand or revise the decision, and stay light enough that I actually
write it down instead of skipping it.

Nygard's format hits that target. The five sections are few enough that
filling them is not a chore, and each one earns its place: Status enables
supersession, Context forces honesty about what was known, Decision is the
artifact, Consequences forces honesty about what was given up. MADR adds
structure I don't need at personal scale; Y-statements drop structure I
do need.

## Decision

Use Michael Nygard's ADR format for decisions in this repo. Each decision
lives in a directory named `NNNN-short-title/` under `docs/adr/`. The
decision text is in `decision.md` within that directory, with sections:
Title, Status, Context, Decision, Consequences, Revision History.

Status values:

- **Proposed** — drafted but not yet committed to.
- **Accepted** — the decision is in effect.
- **Superseded by NNNN** — replaced by a later ADR; the old one stays in
  place for history.
- **Deprecated** — no longer in effect but not replaced by a specific
  successor.

**Immutability.** Once a `decision.md` reaches Accepted status, its
Title, Context, Decision, and Consequences sections are frozen. The only
permitted edits after acceptance are:

- Typo and broken-link corrections.
- Updating the Status line when the ADR is superseded or deprecated.
- Appending an entry to the Revision History section (see below).

Substantive changes to a decision require a new superseding ADR. Editing
accepted content in place is always tempting and always wrong: it destroys
the historical record of what was decided and why it changed.

Before an ADR reaches Accepted it may be edited or renumbered freely.

**Revision History.** Every `decision.md` ends with a `## Revision History`
section. Before acceptance it may be empty or omitted. After acceptance,
each permitted post-acceptance edit appends one line:

```
- YYYY-MM-DD: <what changed and why>
```

This makes the append-only nature of post-acceptance edits explicit and
auditable without relying solely on git log.

**Companion files.** Each ADR directory may contain additional files
alongside `decision.md`. A `checklist.md` is the most common companion:
it tracks implementation steps or acceptance criteria that evolve as the
decision is carried out, and is freely mutable even after `decision.md`
is frozen. Other companions (diagrams, benchmark results, agentic workflow
artifacts) are welcome. No companion file is required.

## Consequences

Positive:

- Every meaningful decision has one document, one location, one revision
  history. Searching is straightforward.
- The format forces the tradeoffs to be explicit. The Consequences
  section in particular catches "we'll regret this later" thinking
  before the decision lands.
- Supersession via new ADRs preserves the historical reasoning. Future-me
  can see not just what the current decision is, but why a previous one
  was abandoned.
- Per-decision granularity means decisions can be revised independently.
  Changing how skills are organized doesn't require touching the ADR
  about sync tooling.

Negative / accepted tradeoffs:

- **Format overhead for trivial decisions.** Some decisions are too small
  to deserve five sections. Convention: don't force everything into ADR
  format; reserve it for decisions whose rationale would otherwise get
  lost. Style choices, file naming, and other reversible micro-decisions
  belong in a CONVENTIONS.md or in code review, not as ADRs.
- **Nygard format is conventionally for system architecture, not personal
  practice.** Using it for "how I work" stretches the original intent.
  This is addressed in ADR 0002, which records the decision to use this
  repo for cross-workspace personal decisions.
- **Immutability requires discipline.** When a decision is wrong, the
  honest move is to write a new superseding ADR. The shortcut — editing
  the old one in place — is always available and always tempting. The
  cost of taking the shortcut is losing the historical record of what
  used to be true and why it changed.

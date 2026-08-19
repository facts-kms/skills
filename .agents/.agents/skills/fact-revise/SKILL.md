---
name: fact-revise
description: >-
  Use when existing knowledge in Facts needs to change, correct, or be
  retired — never create a new proposition for something that already has
  one. Covers `fact revise` (new revision of existing content, reopening its
  decision), and `fact archive` / `fact withdraw` (retiring a proposition
  while keeping its full history, along two independent axes). Trigger on
  cues like "that decision changed", "update the fact about...", "this is no
  longer accurate", "this proposition is superseded", or "retract/withdraw
  that". Always confirm the target with fact-discovery first so you revise
  the right proposition instead of guessing.
version: 0.1.0
---

# Fact Revise and Retire

## Concept

`fact revise` appends a **new revision** to an existing proposition — it
never mutates prior content and never creates a new proposition identity.
`fact archive` and `fact withdraw` each append a **lifecycle event** on one
of two independent tracking axes (archival and withdrawal); neither deletes
anything. All three preserve full history: nothing here is ever truly gone.

## Purpose

This is how Facts stays accurate over time without fragmenting into
duplicates or silently discarding what used to be true. Revising keeps one
addressable identity for a piece of knowledge even as its content evolves;
archiving/withdrawing lets you stop surfacing a proposition as current
without erasing the record that it once existed.

## When to Use It

- **Revise** when a proposition's content is outdated, wrong, or needs
  amendment, and the proposition itself should keep the same identity.
- **Archive** when a proposition is no longer relevant — superseded,
  obsolete, or otherwise not worth surfacing in normal browsing — but wasn't
  necessarily *wrong*.
- **Withdraw** when a proposition is being actively retracted — the
  author/owner taking it back, e.g. it turned out to be mistaken.

## When Not to Use It

- **Don't propose a duplicate.** If a proposition covering this subject
  already exists, revise it — see `fact-discovery` to confirm and locate it
  first, and `fact-propose` for the "nothing exists yet" case.
- Don't revise when the actual need is a formal decision on the existing
  content — that's `fact-decisions` (`accept`/`reject`).
- Don't reach for `archive`/`withdraw` as an "undo" for a revise mistake —
  revising again with corrected content is usually what you want; archiving
  hides a proposition, it doesn't roll back its content.
- Be aware: **there is currently no CLI command to reverse an archive or a
  withdraw.** The underlying protocol supports restoring
  (`restore`/`unarchive`), but the CLI hasn't exposed it yet — treat both
  operations as effectively one-way from the CLI's perspective, and flag
  this to the user if they might want to undo it later.

## Relationships

- `revise` and `archive`/`withdraw` operate on the **same proposition
  identity** that `propose` created — see `fact-propose` for the create
  step and `fact-discovery` for confirming which proposition to target.
- Revising **reopens the decision**: it creates a brand-new deliberation for
  the new revision, carrying forward the same participant list from the
  prior deliberation, but **no one's previous accept/reject decision
  carries over** — every participant, including you, must decide again. See
  `fact-decisions` for the next step.
- `archive` and `withdraw` are **independent** — a proposition can be
  archived, withdrawn, both, or neither; they're not mutually exclusive
  states on a single axis.
- Both `archive` and `withdraw` accept an optional `--reason`, which becomes
  part of the durable record — use it; a bare retirement with no reason is
  much less useful to whoever finds it later.

## Preconditions

- The active ledger must be writable.
- You've identified the exact proposition to target (`fact-discovery`).
- New/replacement content, if any, is ready as canonical Markdown (see
  `fact-propose`'s Preconditions for the exact constraints — they apply
  identically here).

## CLI Usage

```sh
fact revise REFERENCE --message "Updated text in canonical Markdown."
fact revise REFERENCE path/to/updated.md
printf '%s' "$content" | fact revise REFERENCE -

fact archive REFERENCE --reason "Superseded by <short-reference>."
fact withdraw REFERENCE --reason "Turned out to be incorrect: ..."
```

**Always supply `--message`, a file, or `-`/stdin to `revise`** — with none
of these it opens an editor pre-populated with the *current* content, which
will hang in a non-interactive context exactly like `propose`.

## Expected Results

- `revise` returns the new revision's ID and confirms it's now the
  proposition's latest (pending) revision — the *previously* effective
  content remains effective and visible via default `fact show`/`fact echo`
  until the new revision is decided (see `fact-discovery`'s effective-vs-
  latest-vs-pending distinction).
- `archive`/`withdraw` return confirmation and (if given) echo the reason;
  verify with `fact show REFERENCE --all` (see `fact-discovery`), which
  surfaces lifecycle state.

## Failure Modes

- `revise` errors with `no changes made; the proposition was left unchanged`
  if the new content is byte-identical to the current content — this is not
  a bug, it's the CLI catching a no-op.
- Canonical Markdown validation errors, same causes and fixes as
  `fact-propose`.
- If the proposition already has multiple divergent revision tips (it's
  contested), revising or deciding it may require resolving the conflict
  first — see `fact-conflicts`.

## Safety

All three operations are **append-only** ledger writes with full history
preserved — genuinely destructive filesystem-level loss doesn't happen here
(that only applies to `fact ledger delete`, see `fact-ledgers`). Still,
treat them with real judgment:

- Revising **your own recent, uncontested work** is reasonable to do
  autonomously when the task calls for it.
- Revising, archiving, or withdrawing a proposition **other participants
  rely on**, especially on a shared ledger, deserves explicit user
  confirmation first — because revising silently reopens the decision for
  everyone (their prior accept/reject no longer applies), and because
  there's currently no CLI-exposed way to undo an archive or withdraw. When
  in doubt, tell the user what you're about to retire/revise and why before
  doing it.

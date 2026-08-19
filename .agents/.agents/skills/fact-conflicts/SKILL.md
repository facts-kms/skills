---
name: fact-conflicts
description: >-
  Use when a proposition is contested — two incompatible versions of
  "what's true" exist at once — and it needs to be discovered or explicitly
  reconciled. Covers `fact conflicts` (read-only discovery), `fact resolve`
  (the normal reconciliation path), and `fact reconcile create` (the
  low-level fallback for conflicts with no common ancestor). Trigger on cues
  like "contested", "conflicting revisions", "these two decisions
  disagree", "merge these versions", or whenever fact-decisions reports
  "multiple revision tips exist" while trying to accept/reject something.
version: 0.1.0
---

# Fact Conflicts and Reconciliation

## Concept

A proposition becomes **contested** when two independently-valid outcomes
exist for the same knowledge — most often, two sibling revisions were both
accepted on different replicas before they'd synced, or two parallel
deliberations over the same revision settled incompatibly. Facts never
silently picks a winner by timestamp, ID, or which side synced first: it
keeps serving the last undisputed common ancestor as effective and requires
an explicit **reconciliation proposition** to resolve the conflict, itself
subject to the normal decision process.

## Purpose

This is the difference between quietly losing information during
synchronization and making every disagreement visible and auditable. It
exists so distributed, concurrent edits to the same knowledge never produce
a silent "last write wins" outcome.

## When to Use It

- `fact-decisions` reported `multiple revision tips exist; provide an
  unambiguous revision reference` while trying to accept/reject.
- After a `pull` or `push` that might have introduced divergent revisions
  from elsewhere — check `fact conflicts` to see if anything new is
  contested.
- Periodically, as part of routine ledger hygiene on a shared ledger, to
  confirm nothing is sitting contested and unresolved.

## When Not to Use It

- Don't use this for an ordinary pending decision — that's just
  `fact-decisions`; conflicts specifically mean *two accepted-or-competing*
  outcomes exist, not merely "undecided."
- Don't reach for `fact reconcile create` first — always try `fact resolve`
  first; it auto-discovers the conflict group and computes the common
  ancestor for you. Only fall back to `reconcile create` when `resolve`
  itself reports it can't find a common ancestor.
- Don't silently pick a side (e.g. always keeping "yours") without looking
  at both — that defeats the point of surfacing the conflict at all. Read
  both conflicting revisions before choosing a resolution mode.

## Relationships

- `fact conflicts` is pure discovery — pairs with `fact-discovery` in
  spirit, but conflict groups are specific enough to warrant their own
  command and their own follow-up action (`resolve`).
- `fact resolve`/`fact reconcile create` always produce a **new
  reconciliation proposition** — they never mutate or delete the conflicting
  branches directly. That new proposition still needs `accept`/`reject`
  afterward — see `fact-decisions`; resolving does not auto-settle anything.
- Requires **`deliberate` authority** on the ledger to run — see
  `fact-identity` if you get an authority error.

## Preconditions

- The active ledger must be writable, and your identity must hold
  `deliberate` authority on it.
- You understand both (or all) sides of the conflict — read the conflicting
  revisions via `fact conflicts REFERENCE` or `fact show` before choosing a
  resolution mode.
- If authoring new resolution content directly, it must be canonical
  Markdown (same constraints as `fact-propose`).

## CLI Usage

```sh
fact conflicts [REFERENCE] [--all]        # discover; read-only

# Pick exactly one existing conflicting revision as the resolution content:
fact resolve REFERENCE --keep REVISION_REFERENCE

# Merge two (or more, with --pick) picked revisions via an external tool:
fact resolve REFERENCE --merge --pick REV_A --pick REV_B --tool "$FACT_MERGE"

# Author new resolution content directly:
fact resolve REFERENCE --message "Reconciled text in canonical Markdown."
fact resolve REFERENCE path/to/reconciled.md

# Low-level fallback when resolve reports no common ancestor:
fact reconcile create --conflict REVISION:DELIBERATION:SETTLEMENT ... \
  --mode select|derive|reject-all --resolved-tip REF [...] \
  [--selected REF | --result FILE|--message TEXT]
```

`--keep`, `--merge`, `--message`, and a `FILE` argument are mutually
exclusive resolution-content modes — pick exactly one. `--tool` and
`--pick` both require `--merge`. **As with all content-creating commands,
never invoke `fact resolve` with none of `--keep`/`--merge`/`--message`/
`FILE`/`-`** — it opens an editor pre-seeded with a template listing the
common ancestor and each conflicting revision, and will hang non-
interactively.

## Expected Results

- `fact resolve` always reports the new reconciliation proposition as
  **pending** — it never auto-accepts. The command's own output tells you
  the exact next `fact accept`/`fact reject` invocation for both the
  resolution and the underlying reconciliation.
- Verify with `fact show` on the reconciliation proposition before deciding
  it, exactly as with any other pending proposition.

## Failure Modes

- More than one resolution-content mode given: `choose only one resolution
  content mode`.
- `--tool` without `--merge`, or `--pick` without `--merge`: explicit errors
  naming the missing flag.
- `--keep` referencing something that isn't one of the actual conflicting
  revisions: `--keep must reference one of the conflicting revisions`.
- `--merge` with no `--tool` and no `$FACT_MERGE` set: `no merge tool
  configured; pass --tool or set FACT_MERGE`.
- No common ancestor exists for the conflict: `revision conflict has no
  common ancestor; use fact reconcile create` — fall back accordingly.
- Missing `deliberate` authority: an explicit "no ... authority" error — see
  `fact-identity` to request it be granted, rather than working around it.

## Safety

`fact conflicts` is read-only and always safe. `resolve`/`reconcile create`
are judgment calls about how to merge two independently-valid pieces of
knowledge — even though they never delete anything (the losing branch stays
in history) and never auto-accept their result, the choice of which content
wins, or how they're merged, has real consequence for what the ledger says
is true next. For anything beyond a clearly mechanical pick (e.g. one side
is an obvious typo), prefer to confirm the resolution approach with the user
before running `resolve`, and always leave the resulting reconciliation
proposition pending for explicit review rather than deciding it yourself in
the same step.

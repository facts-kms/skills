---
name: fact-protocol-internals
description: >-
  Use only when you specifically need to inspect or manipulate raw Facts
  protocol objects, commitments, snapshots, or read-model state — not for
  ordinary knowledge-management work. Covers `fact object`, `fact
  commitment`, `fact proof`, `fact settlement`, `fact query`, `fact state`,
  `fact conformance`, `fact proposition inspect`, and the low-level `fact
  deliberation` / `fact decision cast` plumbing. Trigger only on explicit
  requests to debug protocol internals, rebuild a local read model, inspect
  raw signed objects, or repair a proposition stuck with "revision is
  awaiting deliberation". For everyday work, use fact-discovery,
  fact-propose, fact-revise, fact-decisions, or fact-conflicts instead —
  this skill is the equivalent of git plumbing (cat-file, hash-object), not
  where ordinary tasks start.
version: 0.1.0
---

# Fact Protocol Internals

## Concept

Beneath the friendly porcelain (`propose`, `accept`, `revise`, `comment`,
…), Fact is built from signed protocol objects, Merkle commitments,
snapshots/bundles, and a rebuildable SQLite read model. A handful of CLI
commands expose that layer directly, mirroring the relationship Git has
between `git commit`/`git merge` and `git cat-file`/`git hash-object`.

## Purpose

These commands exist for debugging, tooling, test-fixture generation, and
the rare recovery scenario the friendly commands can't reach — not for
ordinary knowledge-management work. Documented briefly here so you recognize
them and know when (rarely) to reach for one, without needing the full
protocol specification memorized.

## When to Use It

- A friendly command explicitly points you here — the one real everyday
  case is `fact accept`/`fact reject` failing with `revision is awaiting
  deliberation; use an explicit deliberation repair command`: run `fact
  deliberate REFERENCE` to open the missing deliberation, then retry the
  decision through `fact-decisions` as normal.
- You're asked to debug a raw signed object, verify a commitment or
  settlement, or rebuild a corrupted local read model.
- You're building conformance fixtures or otherwise doing protocol-level
  tooling work, not day-to-day proposition management.

## When Not to Use It

- Never use `fact decision cast` or `fact deliberation open` for an
  ordinary decision — `fact accept`/`fact reject` (see `fact-decisions`)
  already open and settle deliberations correctly; the low-level forms
  require raw `database`/`ledger`/`actor`/`key_id`/`seed` arguments meant
  for tooling, not everyday identity context.
- Don't use `fact object import`/`fact state rebuild` speculatively "just in
  case" — they can affect local database integrity and should be run only
  when actually needed for recovery or tooling, generally at explicit human
  direction.

## Relationships

Topic map — see `fact help COMMAND` for exact syntax on any of these, since
this skill deliberately doesn't reproduce full flag documentation:

| Command | Role |
| --- | --- |
| `fact object validate/import/export` | Validate, import, or export raw signed protocol objects |
| `fact commitment create/verify` | Build/check compact commitments over object hashes |
| `fact proof include/exclude` | Add/remove objects from a commitment proof |
| `fact settlement verify` | Verify a completed decision settlement object |
| `fact query search` | Low-level ledger search against a query file |
| `fact state rebuild` | Rebuild the local SQLite read model from ledger events |
| `fact conformance run/materialize` | Run or generate implementation conformance fixtures |
| `fact proposition propose/revisions/inspect/deliberations/comments` | Inspect the signed proposition objects behind the friendly CLI |
| `fact deliberation open/inspect/participants` | Raw deliberation lifecycle, keyed by explicit database/ledger/actor/key/seed |
| `fact decision cast` | Raw decision recording, same explicit-argument style |
| `fact deliberate REFERENCE` | The one practical exception — repairs a revision with no deliberation attached; safe to use directly when `fact-decisions` points you here |

## Preconditions

Varies by command; most require explicit `--database`/`--ledger` paths or
IDs rather than relying on the active-ledger catalog the friendly commands
use — read `fact help COMMAND` before use.

## CLI Usage

Not reproduced in full here — run `fact help COMMAND` or `fact COMMAND
--help` for exact syntax when one of these is actually needed. The one
command worth knowing by heart from this group:

```sh
fact deliberate REFERENCE     # open a missing deliberation on a revision, then retry accept/reject
```

## Expected Results

Command-specific; `--json` output for this group is more likely to be
hand-built `serde_json::json!` maps than compiler-checked structs, so treat
its schema as generally stable but not guaranteed across versions — verify
against actual output rather than assuming a fixed shape when scripting
against it.

## Failure Modes

Command-specific; consult `fact help COMMAND`. As with the rest of the CLI,
there are no interactive confirmation prompts anywhere in this group either.

## Safety

- **Read-only, safe:** `object validate`, `commitment verify`, `settlement
  verify`, `query search`, `proposition inspect/revisions/deliberations/
  comments`, `deliberation inspect/participants`, `conformance run`.
- **Consequential, advanced/human-directed only:** `object import`, `state
  rebuild` (can affect local database integrity), `decision cast` and
  `deliberation open` (bypass the friendly lifecycle guarantees and require
  exact protocol-level arguments — a mistake here is easy to make and hard
  to reason about compared to the equivalent friendly command).
- `fact deliberate REFERENCE` is the one command in this group that's
  reasonable to run autonomously, specifically as the documented repair
  step when `fact-decisions` reports a missing deliberation.

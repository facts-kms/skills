---
name: fact-sync
description: >-
  Use when local Facts knowledge needs to reach a shared remote, or shared
  knowledge needs to be pulled in locally. Covers `fact remote` (local
  config), `fact push` / `fact pull` (everyday sync of the active ledger),
  and the lower-level `fact sync push` / `fact sync pull` / `fact sync
  retry` (explicit bundle/database operations). Trigger on cues like
  "publish this to the team", "get the latest facts from...", "sync with
  the remote", "set up a remote for this ledger", or before/after
  collaborating with others on a shared ledger.
version: 0.1.0
---

# Fact Sync and Remotes

## Concept

A **remote** is purely local configuration — a saved name and URL, never
signed ledger state. `push`/`pull` exchange **bundles**: exact sets of
signed protocol objects (not database diffs) between your active ledger and
one configured remote. `sync push`/`sync pull`/`sync retry` are the same
exchange mechanism exposed explicitly, for cases the everyday form doesn't
cover.

## Purpose

This is how a ledger stops being purely local — how your propositions reach
collaborators, and theirs reach you, without either side needing direct
database access to the other.

## When to Use It

- You've proposed, revised, or decided something that other people on a
  shared ledger need to see: `push`.
- You want the latest knowledge from a shared ledger before deciding
  something or searching: `pull`.
- Setting up a new collaboration: `fact remote add`.
- An incremental, cursor-resumable, or size-bounded transfer is specifically
  needed, or you're working with `.factbundle`/`.factsnap` files directly
  (e.g. offline/air-gapped exchange): `sync push`/`sync pull`, or `sync
  retry` to resume a previously-interrupted import.

## When Not to Use It

- Don't use `fact tags export`/`fact directory export` and expect `push`/
  `pull` to carry them — extension state (tags, the identity directory)
  syncs separately, on purpose (see `fact-tags`, `fact-identity`), so core
  protocol history stays lean and extension metadata stays optional.
- Don't push from, or pull into, a **read-only** ledger (one created by
  `clone`/`from`) — there's no local signing key to push with, and pulling
  into a read-only ledger is also rejected; see `fact-ledgers`.
- Don't reach for `sync push`/`sync pull` for ordinary single-remote,
  single-ledger work — plain `push`/`pull` already do the right thing with
  simpler, safer defaults.

## Relationships

- `fact remote add NAME URL` registers a remote; `push`/`pull` with no
  `--remote` require **exactly one** configured remote to be unambiguous —
  pass `--remote NAME` when more than one exists.
- `push` internally dumps the local ledger to a temp bundle, then transmits
  it; `pull` fetches a bundle, then imports it locally — both clean up their
  temp file afterward. Functionally this means `push`/`pull` are just a
  convenient wrapper over `sync push`/`sync pull` against your active
  ledger and configured remote.
- A pull that introduces divergent revisions can make a proposition
  contested — check `fact-conflicts` after a pull if anything looks off.

## Preconditions

- The active ledger must be resolvable; for `push`, it must also be
  writable (not read-only).
- Exactly one remote configured, or pass `--remote` explicitly.

## CLI Usage

```sh
fact remote list
fact remote add NAME URL
fact remote remove NAME
fact remote rename OLD NEW

fact push [--remote NAME]              # active ledger -> the one configured remote
fact pull [--remote NAME]              # the one configured remote -> active ledger

# Advanced/explicit bundle forms (bypass the active-ledger catalog entirely):
fact sync push DATABASE FILE [--remote URL]
fact sync pull DATABASE LEDGER OUTPUT [--known-hashes FILE] [--after CURSOR] \
  [--limit N] [--max-object-bytes N] [--remote URL]
fact sync retry DATABASE FILE          # resume a previously deferred/incomplete import
```

Note: `fact push`/`fact pull` also accept an advanced two-positional form
(`fact push DATABASE FILE`, `fact pull DATABASE LEDGER OUTPUT`) that
forwards straight to `sync push`/`sync pull`. Supplying only some of those
positionals is a parse error — pass all of them or none.

## Expected Results

- `push` confirms the objects sent; `pull` confirms the objects received and
  imported.
- Verify with `fact status` (configured remotes, pending count) or
  `fact-discovery` (did the expected content actually arrive/land) after
  either operation.

## Failure Modes

- No remote configured: `no remote is configured; add one with fact ledger
  remote add NAME URL`.
- More than one remote, no `--remote`: `multiple remotes are configured;
  pass --remote NAME or URL`.
- `push` from a read-only ledger: `cannot push from a read-only ledger`
  (pulling into one is rejected the same way).
- Mixing the advanced positional form partially (e.g. `DATABASE` with no
  `FILE`): an explicit "requires both ... or neither" error.

## Safety

**`push` publishes local content to shared, visible state** — treat it
exactly like `git push`: it affects other people, so don't run it without
the user's intent for this specific ledger and remote being clear, even
though the CLI itself won't ask for confirmation. Also don't add a remote
(`fact remote add`) pointing at a URL you haven't verified — `pull`/`clone`
against it will fetch and import whatever it serves.

**`pull` is comparatively low-risk** — it only changes your local copy —
and is reasonable to run on your own judgment to refresh context before
deciding or working. If a pull introduces a contested proposition, stop and
hand off to `fact-conflicts` rather than guessing at a resolution.

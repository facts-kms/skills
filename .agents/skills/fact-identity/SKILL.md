---
name: fact-identity
description: >-
  Use when signing identities, ledger authority, or friendly actor names
  need to be created, inspected, granted, revoked, or rotated. Covers `fact
  as` (the fast path: create-or-switch an actor, name it, and optionally
  grant it capabilities in one command), `fact capabilities` (list what's
  grantable and what the active actor already holds), `fact identity`
  (new/list/show/use/import/export/recognize/revoke/rotate), `fact
  permission` (grant/revoke/capabilities), and `fact directory` (friendly
  names for actors). Trigger on cues like "who can write to this ledger",
  "grant someone access", "revoke that person's permission", "create a
  signing key for this agent", "switch to a different actor", "what can
  this actor do", or "what's this actor's display name". Most operations
  here change who can act with what authority in a shared ledger and should
  not be performed without explicit human direction — read the Safety
  section before acting.
version: 0.1.0
---

# Fact Identity and Permissions

## Concept

Three concerns are kept strictly separate and never imply each other:

1. **Identity** — having a keypair at all (`fact identity new`). Creating a
   key grants no authority anywhere.
2. **Authority** — being granted specific capabilities on a specific ledger
   (`fact identity recognize` / `fact permission grant`, and their inverse,
   `fact identity revoke` / `fact permission revoke`). In the current CLI,
   **`recognize` and `grant` are literally the same operation** (same
   underlying function, invoked from two command names). There is no "trust
   this key but grant nothing yet" step — recognizing an identity always
   requires specifying which capabilities. `revoke` is nearly symmetric
   (see Relationships for the one difference `permission revoke` gained).
3. **Friendly naming** — a display name/alias for an actor
   (`fact directory`). Naming never grants authority, and granting authority
   never requires a name.

`fact as` is a fourth, higher-level piece: a single friendly command that
can touch all three concerns in one call — create or reuse a key, name it,
grant it capabilities, and switch the ledger to sign as it. It's the fast
path for the common case; the granular `identity`/`permission`/`directory`
commands remain available when you need one concern without the others
(e.g. granting capabilities without switching who signs, or exporting/
rotating a key, which `as` doesn't do).

The grantable capabilities are a fixed set: `propose`, `deliberate`,
`invite`, `comment`, `accept`, `reject`, `withdraw`, `archive`, and `admin`
(highly privileged — it can itself grant and revoke). There is currently no
separate `revise` capability. `fact capabilities` prints this list live,
with descriptions and which ones the active actor already holds — treat it
as the authoritative reference over any static list, including this one.

## Purpose

This separation is what lets a shared ledger reason precisely about *who can
do what*, independent of *how they're displayed* or *whether they merely
hold a key*. It also means access changes are fully auditable: authority
grants and revocations are themselves signed, append-only ledger objects,
traceable back to the ledger's genesis admin grant. `fact as` exists on top
of that so the common "bring in a new participant" case doesn't require
chaining three or four separate commands.

## When to Use It

- Bootstrapping your own signing identity for a new ledger context
  (`identity new`, or `fact as` if you also want to name and switch to it
  in the same step).
- Checking which capabilities exist or which ones the active actor holds
  before reasoning about what it can or can't do (`fact capabilities`).
- Looking up or displaying who an actor is (`identity show`, `directory
  show`/`resolve`, or bare `fact as` to report the current signer).
- **When explicitly asked** to grant, revoke, or rotate authority for
  yourself or another actor — never infer this need on your own; see Safety.

## When Not to Use It

- Don't create a new identity reflexively — check `fact status` (or bare
  `fact as`) first; you likely already have one for the active ledger.
- Don't reach for `fact as` when you need a granular step it can't do on its
  own — exporting or rotating key material, or granting capabilities to an
  actor without switching who's currently signing — those stay on
  `identity`/`permission` directly.
- Don't use `identity export` casually — it writes private key material to
  disk (see Safety).
- Don't confuse this with `fact-ledgers`' `ledger delete` — that's a
  destructive local filesystem operation, not an authority change; it lives
  in `fact-ledgers`, not here.
- Don't use the low-level `fact deliberation`/`fact decision cast` commands
  from this area of the CLI's help output for ordinary work — those are
  unrelated protocol plumbing (see `fact-protocol-internals`), not identity
  or permission operations.

## Relationships

- `identity new [--type human|agent|service]` creates signer material only
  — it does **not** automatically become the active signing identity for
  any ledger. `directory add --self` or `identity use` is what actually
  wires a key up as "the identity this ledger signs with." `fact as` can do
  all of this — create, name, and switch — in one call.
- `identity recognize --capability CAP [--capability CAP...] ACTOR` and
  `permission grant --identity ACTOR --capability CAP [--capability CAP...]`
  are interchangeable; both require the target actor object to already be
  imported into the store, and both require the *caller* to hold
  admin-descended authority. Both also accept `--participate` as a shortcut
  for `--capability propose --capability deliberate --capability comment
  --capability accept --capability reject` — the typical "let this actor
  take part in discussions" bundle, without `admin`, `invite`, `withdraw`,
  or `archive`.
- `identity revoke GRANT_REFERENCE` and `permission revoke GRANT_REFERENCE`
  are still the same underlying operation, but `permission revoke` gained a
  second mode `identity` lacks: `permission revoke --identity ACTOR
  --participate` revokes that actor's active participation grants directly,
  without you having to look up a grant reference first. Both forms append
  a signed `authorization_revocation` object; neither **deletes** the
  original grant or rewrites history — anything already signed under the
  grant before revocation stays validly attributed. Revocation only affects
  *future* authority. There is no "un-revoke" — restoring access means
  issuing a brand-new grant.
- `identity rotate` generates a new key, signs continuity proof with the
  *old* key, and moves the ledger's future signing to the new key — but
  **retains the old seed file on disk**, so historical signatures remain
  verifiable and the old key material isn't destroyed.
- `directory add|update|delete|show|list|resolve` manage friendly names.
  `directory export`/`directory push` are the same command (an alias);
  `directory import`/`directory pull` are also the same command — these are
  **file-based bundle export/import**, not a remote network sync analogous
  to `fact push`/`fact pull` (see `fact-sync` for why extension state syncs
  separately and explicitly). `fact as` overlaps with `directory add` for
  the common case (name + optionally create) but doesn't replace
  `directory update`/`delete`/`export`/`import`.
- `fact capabilities` and `fact permission capabilities` print the identical
  list — the standalone top-level form is just easier to remember. Both are
  read-only: they report the active actor's held capabilities and the full
  grantable set, and never change anything.
- `fact as --home [PATH]` doesn't just touch identity — it also prepares a
  dedicated `FACT_HOME` for the actor, which is `fact-ledgers` territory.
  Use it when an actor (especially an AI agent identity) needs its own
  isolated environment, not just a name; see `fact-ledgers` for what
  `FACT_HOME`/environment discovery means more generally. `--print-env`
  prints a shell-ready export for that path; `--use-home` re-checks status
  from inside the prepared home to confirm it actually took effect.

## Preconditions

- For any grant/revoke, the caller's ledger must be writable and the caller
  must hold authority tracing back to the ledger's genesis admin grant.
- For `recognize`/`grant`, the target actor must already be imported into
  the store (e.g. via an earlier interaction, `identity import`, or having
  proposed/signed something visible to you already) — unless you're using
  `fact as` to create that actor in the same step.

## CLI Usage

```sh
fact as                                 # report the current signer; no changes (discovery mode)
fact as ALIAS_OR_NAME                   # switch to an already-known actor
fact as ALIAS_OR_NAME --no-create       # resolve and switch only; never create or modify anything
fact as --self "My Name" --alias me     # name the identity already active in this ledger
fact as "New Agent" --type agent --alias agent-1 --participate
                                         # create, name, grant the participation bundle, and switch to it
fact as "New Agent" --type agent --permission propose --permission comment
                                         # same, with an explicit capability list instead of --participate
fact as "New Agent" --type agent --home --print-env
                                         # also prepare a dedicated FACT_HOME and print its export

fact capabilities                       # active actor's held capabilities + the full grantable list
fact permission capabilities            # identical, nested under permission

fact identity new [--type human|agent|service]
fact identity list
fact identity show ACTOR
fact identity use ACTOR                 # use this local identity for future writes in this ledger

fact identity recognize --capability CAP [--capability CAP...] ACTOR
fact identity recognize --participate ACTOR
fact permission grant --identity ACTOR --capability CAP [--capability CAP...]
fact permission grant --identity ACTOR --participate

fact identity revoke GRANT_REFERENCE [--reason REASON]
fact permission revoke GRANT_REFERENCE [--reason REASON]
fact permission revoke --identity ACTOR --participate [--reason REASON]   # permission-only shortcut

fact identity rotate                    # new key, old key retained for history
fact identity export ACTOR FILE         # writes private key material — see Safety
fact identity import FILE               # import key material without granting anything

fact directory add --self DISPLAY_NAME [--type human|agent|service]
fact directory add --with-identity DISPLAY_NAME     # creates key + binding + directory entry together
fact directory add --actor ACTOR DISPLAY_NAME       # names an existing actor
fact directory show|list|update|delete|resolve ...
fact directory export FILE              # a.k.a. `directory push` — file-based, not remote
fact directory import FILE              # a.k.a. `directory pull` — file-based, not remote
```

## Expected Results

- `identity new` returns a new actor ID and key ID.
- `fact as` returns (depending on the case) the resolved or newly-created
  actor, confirms it's now the active signer, and — with `--home` — the
  prepared environment path.
- `fact capabilities` prints the active actor's held capabilities (marked)
  alongside the full grantable list with descriptions — nothing changes.
- `recognize`/`grant` return the new grant object's reference.
- `revoke` returns the new revocation object's reference, referencing the
  grant it revokes.
- Verify with `fact identity show`/`fact status`/bare `fact as` (current
  active identity) or `fact directory resolve` (does a name now resolve as
  expected).

## Failure Modes

- Granting/revoking without sufficient authority: an explicit "no ...
  authority" error naming the missing capability.
- Recognizing an actor not yet imported into the store: an explicit missing-
  object error — import their identity first, or use `fact as` to create it.
- Asking for a capability outside the fixed set (e.g. a `revise` capability)
  fails validation — there isn't one; revising is not separately gated.
- None of these operations prompt for confirmation; a mistaken grant/revoke
  takes effect immediately and can only be corrected by an equally explicit
  counter-action.

## Safety

This is the most consequential command group in the CLI, and it should be
treated that way:

- **Low-risk, reasonable autonomously:** `identity new` (creating your own
  signing key as part of bootstrapping yourself as a participant),
  `fact as` used only to create/name/switch to **your own** identity, and
  read-only commands (`list`, `show`, `resolve`, `fact capabilities`).
- **Requires explicit human direction — do not perform unprompted:**
  `identity recognize` / `permission grant` and `identity revoke` /
  `permission revoke` targeting **any actor other than yourself**,
  `identity rotate`, `identity export`, and `fact as` when it's being used
  to create or grant capabilities to **another** actor (its `--permission`/
  `--participate` flags are exactly a grant, wearing a friendlier command
  name). These change who can act in a shared, append-only ledger, or
  expose private key material, and none of it is undoable in the ordinary
  sense — revocation is forward-only and rotation doesn't erase the old
  key. Treat a request implying one of these ("give the new agent write
  access", "revoke Bob's permission") as a request to confirm exact
  capabilities and target before acting, not as license to infer the
  details yourself.
- `directory delete`/`directory update` are lower stakes (they only affect
  display metadata, never authority) but still worth confirming on a shared
  ledger, since they're visible to everyone who reads that directory.

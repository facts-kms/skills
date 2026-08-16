---
name: fact-decisions
description: >-
  Use when a proposition needs to be formally decided (accepted or
  rejected), when other people need to weigh in before it's settled, or when
  managing invitations to a discussion. Covers `fact accept`, `fact reject`,
  `fact pending` (your actionable decision inbox), and the discussion layer:
  `fact comment`, `fact comments` (with mine/author/mentions-me/since/
  unresolved/text filters), `fact invitations` (list/accept/reject/show/
  sent/received/pending), `fact invite`, `fact join`, `fact leave`. Trigger
  on cues like "approve that proposition", "reject that", "what's still
  waiting on a decision", "let's discuss this before deciding", "bring
  someone else into this decision", "did I get invited to anything",
  "what have people said about this", or right after fact-propose /
  fact-revise when you need to determine whether the result should be
  decided now or left for review.
version: 0.1.0
---

# Fact Decisions and Discussion

## Concept

Every proposition (and every new revision) carries a **deliberation**: a
consensus round with a participant list and accept/reject decisions.
`propose` and `revise` both open one automatically, with the acting identity
as the sole initial participant — which is why, on a solo ledger, `propose`
then `accept`/`reject` is often the entire workflow, with no separate
discussion step. `comment`/`invite`/`join`/`leave`/`invitations` exist for
the moment more than one real participant needs to weigh in.

`invite` still creates the underlying eligibility token, but `fact
invitations` is now the primary, promoted surface for managing that token
on both sides: the inviter's sent list, the invitee's received/pending
inbox, and an explicit `accept`/`reject` per invitation — instead of only
being discoverable indirectly. `join` still works, and can now target an
invitation reference directly.

## Purpose

Deciding is what turns a proposition from "someone's claim" into "current
truth" (`accepted`) or "considered and declined" (`rejected`). The
discussion layer exists so that decision doesn't have to be made in
isolation when other people's input actually matters, and `invitations`
exists so an actor can see and act on what it's been invited to without
having to be told the reference out of band.

## When to Use It

- A proposition exists (yours or someone else's) and is ready for a
  decision — check `fact pending` for what's actually waiting on you.
- You just created or revised a proposition and need to decide whether it
  should be accepted immediately or left open for review.
- A decision genuinely needs other people's sign-off — bring them in with
  `invite`, or check `fact invitations pending`/`received` before assuming
  you haven't been invited to something.
- You want to record reasoning alongside a proposition without changing its
  content or its decision status — use `comment`.
- You want to catch up on discussion across the ledger, not just one
  proposition — `fact comments` (bare, or filtered with `--mine`/
  `--mentions-me`/`--since`/`--unresolved`) covers that; see also
  `fact-discovery` for proposition-level catch-up.

## When Not to Use It

- Don't use `fact decision cast` or `fact deliberation open/inspect` for
  ordinary decisions — those are low-level protocol plumbing (see
  `fact-protocol-internals`); `accept`/`reject` already do the right thing
  for normal use.
- Don't invite/join for solo-ledger work where you're the only real
  participant — it's unnecessary ceremony; `accept`/`reject` alone is
  sufficient.
- If the proposition is contested (multiple revision tips, or divergent
  deliberations), `accept`/`reject` will fail — that's `fact-conflicts`'
  job, not this skill's.
- If what actually needs to change is the proposition's *content*, that's
  `fact-revise`, not a decision.
- If you're granting someone the *capability* to participate at all (not
  just inviting them to one discussion), that's `fact-identity`
  (`--participate`/`--capability`), not this skill. `invite` itself
  succeeds regardless, but the invited actor can't actually `join`/
  `invitations accept` until they hold the underlying capability (at least
  `comment`) — verified: joining without it fails with `active actor has
  no comment authority grant`, not an invitation-related error.

## Relationships

- **Lifecycle so far:** `fact-propose` creates → **this skill** decides →
  `fact-revise` amends (which reopens the decision, back to this skill) →
  `fact-revise` (`archive`/`withdraw`) retires.
- `accept`/`reject` act on the proposition's currently **pending**
  revision+deliberation — they cannot target an arbitrary past revision by
  ID.
- `fact pending` is the actor-scoped inbox (only items where you're the one
  being asked to decide) — distinct from `fact list --status pending` (see
  `fact-discovery`), which shows everything unsettled ledger-wide regardless
  of who owes the decision.
- `comment`/`invite`/`join`/`leave` all resolve to the proposition's
  currently **effective** deliberation. If a newer revision is pending under
  a fresh deliberation, these commands still target the old effective one —
  there's no CLI-level way to aim them at the new pending revision's
  deliberation specifically. Keep this in mind after a `revise`.
- `invite REF ACTOR` grants a single-use eligibility token; this always
  succeeds if the caller holds `invite` authority, regardless of whether
  the invited actor can actually use it yet (see When Not to Use It).
- The invited actor can resolve that invitation three interchangeable ways:
  `fact join REF --invitation ID` (old explicit form), `fact join
  INVITATION_REFERENCE` (the invitation reference *is* the REFERENCE — no
  `--invitation` flag needed), or `fact invitations accept
  INVITATION_REFERENCE`. All three do the same thing; `invitations show
  REFERENCE` prints the exact next commands, including these, so it's a
  reasonable thing to run first when unsure.
- `fact invitations reject REFERENCE [--reason REASON]` is new: an explicit,
  recorded decline, distinct from just never joining.
- `fact invitations list` shows everything (sent and received); `sent`/
  `received` filter by direction; `pending` narrows further to invitations
  still awaiting action. All are scoped to the active actor — switch actors
  (`fact-identity`'s `fact as ALIAS`) to check someone else's inbox.
- `leave REF` fails once you've already submitted a decision in that
  deliberation — decide, then leave, not the other way around, if leaving
  matters.
- `fact comments` without a `REFERENCE` searches across the ledger instead
  of one proposition; combine with `--mine`, `--author ACTOR`,
  `--mentions-me`, `--since 7d`-style durations or timestamps,
  `--unresolved`, or `--text` to narrow it. This makes `comments` a
  discussion-discovery tool in its own right, not just a per-proposition
  detail view — pairs with `fact-discovery` for the proposition-level
  equivalent.

## Preconditions

- The active ledger must be writable, and you must be an active participant
  of the relevant deliberation (automatic if you proposed/revised it, or
  after joining via invitation).
- Content for `comment`, like all content-creating commands, must be
  canonical Markdown (see `fact-propose`'s Preconditions).

## CLI Usage

```sh
fact pending                         # your actionable decision inbox

fact accept                          # accept the sole unambiguous pending proposition
fact accept REFERENCE                # accept a specific one
fact reject REFERENCE                # reject a specific one

fact comment REFERENCE --message "Reasoning or context, canonical Markdown."

fact comments                        # every comment across the ledger the active actor can see
fact comments REFERENCE              # comments on one proposition/revision/discussion
fact comments --mine                 # only the active actor's own comments
fact comments --mentions-me          # only comments that mention the active actor
fact comments --author ACTOR --since 7d --unresolved
fact comments REFERENCE --content    # full comment text instead of one-line summaries

fact invite REFERENCE ACTOR          # grant a join-eligibility token (needs `invite` authority)

fact invitations                     # same as: fact invitations list
fact invitations list                # every invitation for the active actor, sent and received
fact invitations sent                # only ones the active actor sent
fact invitations received            # only ones the active actor received
fact invitations pending             # received and still awaiting action
fact invitations show REFERENCE      # detail + suggested next commands
fact invitations accept REFERENCE    # join using this invitation
fact invitations reject REFERENCE [--reason REASON]   # explicitly decline

fact join REFERENCE                  # REFERENCE can be an invitation, proposition, or discussion
fact join REFERENCE --invitation INVITATION_ID  # explicit form when REFERENCE isn't the invitation
fact leave REFERENCE                 # step down as a participant (only before deciding)
```

**Always supply `--message`, a file, or `-`/stdin to `comment`** — the same
editor-hang hazard described in `fact-propose` applies here.

## Expected Results

- `accept`/`reject` return the proposition's new status and confirm which
  revision became effective.
- `fact pending` lists everything currently awaiting your decision; an
  empty result means nothing is waiting on you right now.
- `fact invitations show`/`list`/`pending` (with `--json`) include a
  `next_actions` field listing the exact follow-up commands (e.g. `fact
  invitations accept ...`) — a reliable way to pick the next step
  programmatically instead of re-deriving it.
- Verify with `fact show REFERENCE` (see `fact-discovery`) to confirm the
  status and effective content match intent after deciding.

## Failure Modes

- No `REFERENCE` given and zero pending propositions: `no unambiguous
  pending proposition`.
- No `REFERENCE` given and more than one pending: `multiple pending
  propositions; provide a reference` — supply the reference explicitly.
- `REFERENCE` given but doesn't match a pending item you can act on: `no
  pending proposition matches reference ...`.
- Targeting a revision with no deliberation attached yet (rare — usually
  only from imported/protocol-level objects): `revision is awaiting
  deliberation; use an explicit deliberation repair command` — run `fact
  deliberate REFERENCE` (see `fact-protocol-internals`) to open one, then
  retry.
- Targeting a proposition you're not currently owed a decision on: `no
  pending action for the current actor`.
- Targeting an already-settled proposition: `proposition has no unsettled
  revision; its effective revision is already settled`.
- Targeting a contested proposition: `multiple revision tips exist; provide
  an unambiguous revision reference` — route to `fact-conflicts`.
- `leave` after already deciding: `cannot leave a deliberation after
  submitting a decision`.
- Joining or accepting an invitation without the underlying capability
  (verified for `comment`): `active actor has no comment authority grant`
  — this is an authority problem, not an invitation problem; route to
  `fact-identity` to grant the capability rather than re-inviting.
- An invitation reference that doesn't resolve: the generic missing-object
  error (see `fact-discovery`'s reference-resolution notes) — not a
  special-cased invitations error.

## Safety

Deciding asserts something is **settled** — treat `accept`/`reject` the way
you'd treat merging a pull request, not the way you'd treat a read. The CLI
itself has **no confirmation prompt** for any of this; it executes
immediately. On a solo ledger, deciding your own proposals as part of a task
you were explicitly asked to complete is reasonable. On a shared ledger, or
for anything with real consequence beyond your own working context, prefer
to leave the proposition pending and tell the user what's waiting rather
than deciding on their behalf — they can `fact accept`/`fact reject` it
themselves once you've surfaced it. `comment`/`invite`/`join`/`invitations`
(including `reject`) are lower stakes (they don't settle anything) and are
reasonable to use autonomously whenever they help move a decision toward
the right people — declining an invitation on the active actor's own behalf
is no riskier than never joining would have been.

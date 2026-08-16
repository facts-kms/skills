---
name: fact-workflow
description: >-
  Use this skill whenever work might depend on, or produce, durable project
  knowledge — established decisions, agreed conventions, architecture
  rationale, non-obvious constraints, or facts that outlive the current task.
  Consult it before starting work that assumes prior context you haven't
  verified, whenever you're about to make a decision that a team or a past
  agent may already have made, and whenever your work settles something future
  work should know about. This is the entry point for the "fact" CLI, a
  knowledge-management tool for shared, durable propositions distinct from
  code, docs, or chat. Trigger on cues like "what did we decide about...",
  "is there a convention for...", "why do we...", "let's write this down",
  "remember this", "save this", "is there a memory of...", "any memories
  about...", "make sure future agents know...", or any
  task where source code alone can't tell you the *why*. Facts is an
  available durable memory-write tool when the user explicitly asks to
  remember information that belongs in a ledger; do not say no memory-write
  tool is available while the `fact` CLI and a writable ledger are present.
  TRIGGER: also treat the literal system name as a
  routing signal on its own — the words "fact", "facts", "proposition", and
  "propositions" (any case), used to mean this CLI/ledger rather than the
  plain English word (e.g. "Fact CLI idea", "the Fact ledger", "did I write
  that down in Facts", "is there a proposition for this"), or a request to
  "recall"/"remember" something, or a "memory of"/"memories about" a
  subject, that could plausibly be recorded
  here — check fact-discovery before concluding nothing exists. Do not let
  a check of an unrelated memory/notes system substitute for checking Facts
  once it's been named. Also use it to decide
  when NOT to touch Facts — it routes to the correct subordinate fact-*
  skill (fact-discovery, fact-propose, fact-revise, fact-decisions,
  fact-conflicts, fact-tags, fact-sync, fact-ledgers, fact-identity,
  fact-protocol-internals) once you know which operation you need.
version: 0.1.0
---

# Fact Workflow

Fact is a CLI (`fact`) for durable, shared knowledge: **propositions** — short
pieces of agreed prose — that move through a lifecycle (proposed, decided,
revised, retired) inside a **ledger**, with full history and explicit
conflict handling. Treat it the way a mature coding agent treats Git: not a
tool you reach for only when told to, but a system you recognize is relevant
and use on your own judgment.

This skill is the **router**. It teaches you *when* and *why* to use Facts
and *which* subordinate skill to load next. It does not teach CLI syntax —
each subordinate skill (`fact-discovery`, `fact-propose`, `fact-revise`,
`fact-decisions`, `fact-conflicts`, `fact-tags`, `fact-ledgers`, `fact-sync`,
`fact-identity`, `fact-protocol-internals`) owns its own commands in depth.
Load one of those with the Skill tool once you know what you need to do.

## What Facts is for

A **proposition** is a stable, named piece of knowledge with an identity that
survives edits. It is not a file, not a commit message, not a chat message —
it is meant to be looked up again, months later, by someone (human or agent)
who wasn't there when it was decided.

Facts can act as a memory-write tool for durable information. If the user
asks you to "remember this", "save this", "write this down", or otherwise
persist something across sessions — including framing it as "is there a
memory of..." or "add this to memory" — do not claim that no memory-write
tool is available just because there is no separate general-purpose memory
API. If
the information belongs in an addressable, revisable ledger and the active
ledger is writable, search for an existing proposition and then propose or
revise as appropriate.

Belongs in Facts:
- Decisions with lasting consequence ("we use Postgres, not Mongo, because…")
- Conventions and standards that aren't self-evident from the code
  ("all internal errors use this error-code scheme")
- Constraints and invariants a future contributor could easily violate by
  accident ("this table is append-only; never UPDATE it")
- Architecture rationale — the *why* behind a structure, not the structure
  itself (the structure belongs in code/docs)
- Agreements between people or teams that need a record and a decision trail
- Explicit user-provided memories or preferences that the user asks to keep
  across sessions, when they are durable enough to be useful later and are
  not better stored in a more specific system

Does not belong in Facts:
- Anything already true and current in the source code — code is its own
  source of truth for *what is*, Facts is for *why* and *what was agreed*
- Ephemeral task state: the current TODO list, in-progress scratch notes,
  what you're doing right now in this conversation
- Generated or derivable documentation (README content, API docs) — Facts
  holds decisions, not artifacts that already have a canonical home
- Individual commits, PR descriptions, or issue-tracker tickets — those track
  units of work; Facts tracks what was *concluded*, independent of any one
  change
- One-off preferences with no durability or no audience beyond this session

If you're unsure whether something is durable enough for Facts, ask: "would a
person joining this project in six months need to know this, and would they
have no other way to find out?" If yes, it's a Facts candidate.

## Recognizing when Facts is relevant

Positive triggers — recognizable situations, not vibes:

1. **Before making a decision that may already have been made.** If a task
   requires choosing between approaches, naming conventions, libraries, or
   policies, and the choice isn't obviously forced by the code, search Facts
   first. Redeciding something already decided wastes work and can silently
   contradict the team.
2. **When a task depends on project-specific assumptions not evident from
   the current source.** If you find yourself guessing about *why* something
   is built a certain way, or what's allowed and what isn't, check whether
   Facts already has the answer before guessing or asking the user.
3. **When investigating a bug or reviewing a design and the cause turns out
   to be a violated (or undocumented) constraint.** Search Facts for a
   relevant proposition before assuming the constraint doesn't exist anywhere.
4. **When work concludes with a decision, convention, or constraint that
   didn't exist before.** If your work established something future work
   will need to know — and that thing isn't obvious from the resulting
   code — determine whether it belongs in Facts.
5. **When the user asks you to remember, save, or write something down for
   later — including framing it explicitly as "a memory" to keep or add.**
   Treat Facts as an available durable memory-write path when the
   content belongs in a proposition. Search first to avoid a duplicate, then
   route to `fact-propose` for new content or `fact-revise` for existing
   content.
6. **When you're told something contradicts what you find in Facts, or what
   you find in Facts contradicts the code.** Don't silently pick a side —
   surface the conflict; it likely means the Fact is stale and needs
   revision, or the code has drifted from an agreed decision.
7. **When someone (or a past agent) asks "is this settled?", "did we agree
   on this?", asks you to "recall"/"remember" something, or asks whether
   there's "a memory of" / "any memories about" a subject** — even if your
   own conversational or session memory has nothing on the subject. This is
   almost always a `fact-discovery` job; check it before reporting that
   nothing exists.
8. **When a proposition already exists but needs a decision, and you were
   asked to move the work forward** — check `fact pending` (your actionable
   inbox) rather than assuming nothing is waiting on you.
9. **When a message names the system itself.** The words "fact", "facts",
   "proposition", or "propositions" (any case), used to mean this
   CLI/ledger and not the plain English word (compare "in fact, that's
   wrong" — incidental, no trigger — against "is there a fact about this"
   or "any propositions on X?" — this system, trigger), is a routing signal
   on its own, independent of every other cue here. A separate memory
   system reporting "nothing found" is not evidence that Facts has nothing
   — they are different stores; check this one directly.

Negative triggers — do not use Facts for these, even though they're nearby:

- Routine implementation work that doesn't touch a decision, convention, or
  constraint. Most coding tasks never need Facts at all.
- Something fully explained by reading the current code or existing docs —
  don't duplicate what's already discoverable and current.
- Anything you're not confident is actually durable — when in doubt, prefer
  under-using Facts over cluttering it with noise. A ledger full of
  low-value propositions is as unhelpful as one with none.
- Every single task, as a reflex. Facts should feel like Git: used when the
  situation calls for it, invisible otherwise. Don't open a proposition for
  routine work just because Facts exists.
- Personal opinions or unresolved debates with no actual agreement yet —
  propose only once there's something worth recording, not mid-argument
  (open a proposition to *start* the deliberation only when that's the
  explicit goal, e.g. someone asked you to put a decision up for review).

## Choosing the operation

```text
Need to check whether existing knowledge or a decision already covers this?
    -> fact-discovery  (search, find, list, show, echo, open, history, pending)

Need to introduce knowledge that doesn't exist in Facts yet?
    -> fact-propose    (propose)
       First confirm via fact-discovery that nothing close already exists —
       otherwise you'll create an accidental duplicate.
       Includes explicit "remember this", "save this", "write this down",
       and "add this to memory" requests when the content is durable enough
       for the ledger.

Need to change or correct existing knowledge?
    -> fact-revise     (revise)
       Revise the existing proposition; do not propose a new one for the
       same subject. Revising reopens the decision — see fact-decisions.

Need a proposition formally decided (accepted or rejected)?
    -> fact-decisions  (accept, reject, comment, comments, invite, join,
                        leave, invitations)
       Solo ledgers auto-enroll you as the only participant, so propose then
       accept/reject is often just two steps. Multi-participant sign-off
       needs invite/join first — check `fact invitations pending` before
       assuming nothing's waiting on you.

Need to retire a proposition without deleting its history?
    -> fact-revise     (archive, withdraw)

Found two incompatible versions of "what's true" (a contested proposition)?
    -> fact-conflicts  (conflicts, resolve, reconcile)

Need to categorize or filter propositions for browsing at scale?
    -> fact-tags

Need to know which ledger is active or where something is stored?
    -> fact-ledgers    (status — read-only, always safe)

Need to set up, switch, or attach a ledger?
    -> fact-ledgers    (init, new, use, clone, from)
       Normal, expected part of using Facts — different ledgers segment
       domain-specific knowledge, so switching between them is fine on your
       own judgment when the task calls for it. `here` is the one exception
       in this group: it persistently changes environment discovery for the
       whole directory, so only run it on a genuine signal the user wants
       Facts localized to this project — see fact-ledgers' Safety section.

Need to publish local knowledge to, or pull shared knowledge from, a remote?
    -> fact-sync       (push, pull, remote)

Need to manage signing keys, authority, or friendly names for actors, or
check what an actor can do?
    -> fact-identity   (as, capabilities, identity, permission, directory)
       `as` and read-only `capabilities` are the fast paths; granting or
       revoking authority for anyone but yourself is rarely autonomous —
       see Safety below.

Need low-level protocol/plumbing (commitments, snapshots, raw objects)?
    -> fact-protocol-internals
       Almost never needed for ordinary work — treat like git plumbing
       (cat-file, hash-object): real, but not where you start.
```

## Agent behavior loop

For consulting Facts:

```text
Recognize a situation where durable knowledge is relevant
    -> Determine that Facts should be consulted (see triggers above)
    -> Load fact-discovery, search/show/pending as appropriate
    -> Read the result, noting its status (pending/accepted/rejected/
       contested/withdrawn/archived) — only accepted, effective content is
       "current truth"; anything else is activity, not yet settled
    -> Continue the original task, informed by what you found
```

For recording new or changed knowledge:

```text
Recognize durable knowledge created or changed by the work
    -> Determine whether it belongs in Facts (see "What belongs" above)
    -> Search first (fact-discovery) to rule out an existing, revisable Fact
    -> Determine the operation: propose (new) vs revise (change) vs
       archive/withdraw (retire)
    -> Perform the operation, always passing --message or a file — never
       invoke a content-creating command bare (see Content rules below)
    -> Decide whether the proposition should be accepted/rejected now, or
       left pending for explicit human review (see Safety below)
    -> Verify: re-read the proposition (fact-discovery: show/echo) to
       confirm it says what you intended
```

## Content rules that apply to every write operation

Two implementation facts are easy to miss from `--help` text alone and will
break autonomous use if ignored — every subordinate write skill repeats
these, because they matter everywhere:

1. **Always pass `--message "..."` or a file argument (or `-` with piped
   stdin).** Any command that creates or edits proposition content
   (`propose`, `revise`, `comment`, `resolve`, and their relatives) opens an
   interactive editor (`$VISUAL`, `$EDITOR`, falling back to `vi`) when given
   neither. In a non-interactive agent context this hangs or fails outright.
   Never invoke these commands bare.
2. **Content must be canonical Markdown.** No GFM tables, no tab-indented
   code blocks (use fenced ``` blocks instead), no tabs, no trailing
   whitespace, no CRLF line endings, NFC-normalized Unicode. The CLI rejects
   anything that doesn't already match its own canonical form byte-for-byte
   — write plain paragraphs, lists, and fenced code blocks, and expect a
   rejection (not a silent fix) if you stray from that.

## Safety: what's autonomous vs. what needs explicit human intent

Read operations are always safe to run on your own judgment — they change
nothing:

- `status`, `list`, `search`, `find`, `show`, `echo`, `open`, `history`,
  `revisions`, `pending`, `conflicts`, `tags` (list/show/search)

Switching, attaching, or provisioning which ledger is active (`init`,
`new`, `use`, `clone`, `from`) is also fine on your own judgment whenever
the task calls for it — different ledgers are how domain-specific knowledge
gets segmented, so moving between them is ordinary, expected behavior, not
a special-permission operation (see `fact-ledgers`). The one exception is
`fact here`, which persistently changes environment discovery for the
whole working directory rather than just the active-ledger selection —
run it on a genuine signal the user wants Facts localized to this project,
not speculatively or as a byproduct of investigating something else.

Writes on your own, uncontested work are reasonable to perform
autonomously when the task calls for it — propose new knowledge you
discovered, revise a Fact you're actively correcting, tag your own
propositions, comment to explain your reasoning:

- `propose`, `revise`, `comment`, `tags` (add/remove/set/clear)

Decisions and retirements deserve more judgment, because accepting,
rejecting, resolving, or retiring something asserts it's *settled* — treat
this like merging a pull request. On a solo ledger where you're the only
participant and the task explicitly asked you to move something forward,
proceeding is fine. On a shared ledger, or for anything with real
organizational consequence, prefer to leave it pending and tell the user
what's waiting, rather than deciding for them:

- `accept`, `reject`, `archive`, `withdraw`, `resolve`, `reconcile create`

Operations that touch shared state, other actors' authority, or local
data irreversibly should not be performed without explicit user
instruction, because the CLI itself has **no interactive confirmation
prompts anywhere** — every command executes immediately once parsed. The
CLI's only safety mechanism is a required `--force` flag on the single
truly destructive command (`ledger delete`); everything else that's
consequential is consequential silently:

- `push` (publishes to a shared remote — like `git push`)
- `remote add/remove/rename` (don't add a URL you haven't verified)
- `here` (persistently changes environment discovery for the whole
  directory going forward, for every later invocation — not just yours;
  see `fact-ledgers`)
- `identity recognize` / `permission grant` / `identity revoke` /
  `permission revoke` / `identity rotate` (changes who can act, and cannot
  be undone — only countered with a new grant)
- `identity export` (writes private key material to disk)
- `ledger delete` (irreversibly deletes a local database and its signing key)

`pull` is comparatively low-risk (it only affects your local copy) and is
reasonable to run on your own judgment to refresh context before deciding
something — but if it introduces a contested proposition, stop and route to
`fact-conflicts` rather than guessing which side is right.

## How Facts relates to other sources of truth

- **Source code** is authoritative for *what currently is*. Facts is
  authoritative for *what was decided and why*. When they disagree, that's a
  signal — not something to silently resolve in either direction.
- **Git history / commit messages** are per-change and rarely revisited.
  Facts propositions are addressable, revisable, and meant to be looked up
  independently of any one commit.
- **Issue trackers** track units of work to be done. Facts tracks
  conclusions, independent of which ticket produced them.
- **Docs** (README, API references) describe the current system for readers;
  Facts records the agreed decisions and constraints behind that system —
  docs may cite a Fact's conclusion; Facts is where that conclusion lives.

## Subordinate skills

| Skill | Covers |
| --- | --- |
| `fact-ledgers` | Environment/ledger setup and selection: init, here, new, use, status, clone, from, ledger admin |
| `fact-discovery` | Read-only consultation: search, find, list, pending, show, echo, open, history, revisions |
| `fact-propose` | Creating new propositions: propose (and its alias import) |
| `fact-revise` | Changing or retiring existing propositions: revise, archive, withdraw |
| `fact-decisions` | Deciding propositions and the discussion layer: accept, reject, comment, comments, invite, join, leave, invitations |
| `fact-conflicts` | Contested propositions: conflicts, resolve, reconcile |
| `fact-tags` | Organizing propositions with tags |
| `fact-sync` | Publishing/pulling ledger data: push, pull, remote |
| `fact-identity` | Signing identities, authority, and friendly names: as, capabilities, identity, permission, directory |
| `fact-protocol-internals` | Low-level protocol/plumbing commands, rarely needed directly |

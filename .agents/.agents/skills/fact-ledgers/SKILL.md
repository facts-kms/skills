---
name: fact-ledgers
description: >-
  Use when you need to set up, switch, inspect, or attach a Fact ledger —
  before any other fact command will work reliably. Covers `fact init`,
  `fact here`, `fact new`, `fact use`, `fact status`, `fact clone`, `fact
  from`, and low-level `fact ledger` administration (create/delete/list/init).
  Trigger when a fact command fails with "no active ledger", when starting
  work in a new project directory that might want project-local Facts, when
  the user asks to switch which ledger/project knowledge base is active
  (normal and expected — different ledgers segment domain-specific
  knowledge), when you need to mirror or read someone else's shared ledger,
  or before running `fact here` specifically — it's the one command in this
  group that persistently changes environment discovery for the whole
  directory, so it needs a genuine signal the user wants Facts localized to
  this project, not mere curiosity about where something is stored (that's
  `fact status`). Load fact-workflow first if you haven't already decided
  Facts is relevant to the task.
version: 0.1.0
---

# Fact Ledgers and Environment

## Concept

A **ledger** is Facts' unit of history, consensus, and permission — every
proposition, revision, and decision lives inside exactly one ledger, and
ledgers never share state implicitly. The closest Git analogy is an
**independent repository**, not a branch: copying a proposition's content
into a different ledger gives it a new identity and a fresh consensus
history, it doesn't "merge" the way a branch does.

Separately, the CLI keeps a **local catalog** of ledgers you know about —
friendly names (`default`, `work`, …) each pointing at a SQLite database
file, a local signing identity, and configured remotes. Exactly one catalog
entry is the **active ledger** at a time, and that selection is pure local
client convenience state: it is never synchronized, never derived from the
working directory by default, and never part of the signed protocol data.

## Purpose

Before any proposition, search, or decision command can do anything, Fact
needs to know *which* ledger you mean. This skill exists to get that context
right — creating or selecting the right ledger — so the rest of your Facts
work operates on the correct knowledge base instead of silently defaulting
to the wrong one or failing with "no active ledger."

## When to Use It

- At the start of a task in an unfamiliar project, to confirm which ledger
  (if any) is active before reading or writing anything (`fact status`).
- The first time a project wants its own local knowledge base
  (`fact here --init`, or plain `fact init` for a global one).
- When you need a second ledger without disturbing the one currently active
  (`fact new`).
- When switching between ledgers you've already set up (`fact use`).
- When you need to inspect or search someone else's published ledger without
  any risk of writing to it (`fact clone` / `fact from`).
- Any time a command errors with `no active ledger; run 'fact init'`.

## When Not to Use It

- **Never run `fact here` just to investigate or "figure out" something.**
  It's the one command in this group with a lasting side effect beyond the
  active-ledger catalog: it writes a persistent `.facts/` directory that
  changes environment discovery for the whole working directory going
  forward (see Preconditions), for every later `fact` invocation, not just
  this one. If the actual question is "where is this stored" or "which
  ledger is active," the answer is `fact status` (read-only) or
  `fact-discovery`, never `fact here`. A real incident this guards against:
  an agent trying to determine where memories were stored ran `fact here`,
  which silently created project-local configuration and changed which
  environment later `fact` commands in that directory would use.
  `init`/`new`/`use`/`clone`/`from` don't have this hazard — switching or
  provisioning a ledger only ever changes the active-ledger selection, which
  is ordinary, frequently-needed behavior (see Safety).
- Don't re-run `fact init`/`fact here` reflexively at the start of every
  task — check `fact status` first. If a ledger is already active and
  correct, there's nothing to set up.
- Don't use `fact clone`/`fact from` when you actually need to write —
  they produce **read-only** ledgers with no local signing key at all (see
  Preconditions). If write access is the goal, you need the ledger's owner
  to grant you authority instead (`fact-identity`).
- Don't reach for `fact ledger create/clone/delete/init` (the low-level
  administrative form) for everyday setup — `init`/`new`/`use`/`clone`/`from`
  cover normal cases with safer defaults and clearer errors.

## Relationships

- `fact init [NAME]` creates the named ledger if it doesn't exist (reusing it
  if it does) **and switches to it**.
- `fact new [NAME]` does the same creation/reuse but **does not** switch —
  use it to provision a ledger you're not ready to use yet.
- `fact use NAME` switches the active ledger among ones already known to the
  local catalog; it does not create anything.
- `fact status` is pure read: which ledger is active, its ID, database path,
  local actor/key, whether it's read-only, pending-decision count, and
  configured remotes. Cheap, side-effect-free — a good first call.
- `fact here [--init [NAME]] [--no-switch]` creates a **project-local**
  `.facts/` directory so ordinary commands use that directory automatically
  when `FACT_HOME` isn't set — the project-scoped equivalent of `git init`.
  Without `--init` it only creates the config directory, not a ledger.
- `fact clone SOURCE [--ledger ID]` copies a bundle (local file or remote
  URL) into a new local **read-only** ledger and activates it.
- `fact from DATABASE [NAME] [--ledger ID]` registers an existing local
  `.sqlite` Fact database file as a **read-only** ledger, no network
  involved, and activates it.
- `fact ledger create/clone/delete/list/init/remote` is the explicit,
  lower-level administrative surface behind the friendly commands above —
  reach for it only when you need something the friendly commands don't
  expose (e.g. `fact ledger delete`, or supplying an exact ledger ID with no
  auto-derivation).

## Preconditions

- **Environment discovery order** (first match wins): `FACT_HOME` env var, →
  a `.facts/` directory under the current working directory (created by
  `fact here`), → `$XDG_DATA_HOME/fact`, → `$HOME/.local/share/fact`. You do
  **not** need to run `fact here` before `fact init` — `init` works standalone
  anywhere and just uses whichever root is discovered.
- Every `fact` invocation re-discovers the environment fresh from disk —
  there's no daemon or session state, so ordering across separate invocations
  only matters through what's actually persisted (the active-ledger file,
  the catalog, seed files).
- `--ledger NAME` on any command overrides the active ledger for that one
  invocation only, without changing persisted state — only `fact use`
  persists a new active ledger.
- **Read-only ledgers have no local signing key at all** (empty actor/key/
  seed in the catalog entry, not merely a permission restriction) — every
  write path, including `push`, fails immediately against one. Know before
  you clone/from whether you need to write; if so, this isn't the path.

## CLI Usage

```sh
fact status [--ledger NAME]              # inspect active ledger, no side effects

fact init [NAME]                         # create-or-reuse NAME, then switch to it (default name: "default")
fact new [NAME]                          # create-or-reuse NAME, do not switch
fact use NAME                            # switch active ledger to an existing catalog entry

fact here [--path PATH] [--init [NAME]] [--no-switch] [--force] [--print-env]
                                          # create project-local .facts/ ; --init also creates a ledger there

fact clone SOURCE [--ledger ID]          # read-only mirror from a bundle file or remote URL; activates it
fact from DATABASE [NAME] [--ledger ID]  # read-only registration of an existing local .sqlite file; activates it

fact ledger list                         # advanced: list all local ledgers
fact ledger create NAME                  # advanced: explicit create, no activation
fact ledger delete NAME --force          # advanced: irreversible — see Safety
fact ledger remote list|add|remove|rename  # equivalent to `fact remote ...`, scoped explicitly
```

## Expected Results

- `init`/`new`/`clone`/`from` return the new ledger's ID, database path, and
  (for `clone`/`from`) confirm `read_only: true`.
- `use` confirms the newly active ledger name.
- `status` prints (or, with `--json`, returns structured) name, ledger ID,
  database path, actor/key IDs, `read_only`, pending-decision count, and
  configured remotes — this is the right command to verify any of the above
  actually took effect.

## Failure Modes

- Any command run with no active ledger and no `--ledger` override fails
  with `no active ledger; run 'fact init'` — run `fact init` or `fact use`
  and retry.
- `fact ledger delete NAME` without `--force` fails outright — this is a
  required flag, not an interactive prompt (the CLI has no confirmation
  prompts anywhere).
- Attempting to write (propose, revise, push, …) against a read-only ledger
  fails per-operation with an explicit error (e.g. `cannot push from a
  read-only ledger`) rather than silently no-op'ing.

## Safety

- **Read-only, always safe — and the correct tool for any "what's active /
  where is this stored" question:** `status`.
- **Reversible / low-risk, safe for autonomous use when the task calls for
  it:** `init`, `new`, `use`, `clone`, `from`. Switching or attaching which
  ledger is active is ordinary, expected behavior, not a special-permission
  operation — different ledgers are how domain-specific knowledge gets
  segmented, so moving between them (or provisioning a new one) as a task
  requires is no more dangerous than switching a git branch when the work
  calls for it. (`init` switches too when the named ledger already exists,
  same as `use` — that's expected, not a surprise.)
- **The one command in this group that needs a genuine signal before
  running, not mere curiosity:** `here`. Unlike the rest, it writes a
  persistent `.facts/` directory that redirects environment discovery (see
  Preconditions) for every future `fact` invocation from that path or
  below — including ones run later by someone else who never asked for
  project-local scoping. Localizing Facts to a project is a perfectly
  legitimate, common request (e.g. "let's keep this project's knowledge
  separate") — run `here` freely when that's actually what's being asked
  for. What it should never be is a byproduct of investigating something
  else, or a guess at what might be wanted; if you're only trying to find
  out where something is stored or which ledger is active, that's `fact
  status`, never `fact here`.
- **Destructive, requires explicit user instruction:** `fact ledger delete
  --force` permanently deletes the local SQLite database file **and the
  local signing key's seed file** from disk — this is the one truly
  unrecoverable operation in the entire CLI (everything else that looks
  destructive is an append-only ledger write; this one is real filesystem
  data loss). Never run it without the user explicitly asking to delete a
  specific named ledger.
- Adding a remote (`fact remote add`) is covered in `fact-sync`, but note it
  from here too: don't register a remote URL you haven't verified belongs to
  someone you trust — `fact clone`/`fact pull` against it will fetch and
  import whatever it serves.

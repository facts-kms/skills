---
name: fact-discovery
description: >-
  Use whenever you need to look something up in Facts before or during work
  — checking whether a decision already exists, finding the one accepted
  answer to a question, browsing what's recorded, catching up on a specific
  proposition's status and recent activity, checking your own actionable
  decision inbox, or auditing history. Covers `fact search`, `fact find`,
  `fact list`, `fact pending`, `fact show`, `fact echo`, `fact open`, `fact
  export`, `fact history`, and `fact revisions`. All operations here are
  read-only and safe to run on your own judgment. Trigger on cues like "is
  there a fact/decision about...", "what did we decide about...", "what's
  the status of...", "catch me up on...", "what's still pending", or before
  proposing/revising anything (to avoid creating a duplicate).
version: 0.1.0
---

# Fact Discovery

## Concept

Discovery commands answer "what does Facts currently know?" without changing
anything. They're the read side of the CLI: searching text, listing by
status, resolving to one accepted answer, printing a proposition's effective
content, and inspecting history. All of it operates on the active ledger
(see `fact-ledgers` if none is active).

The single most important invariant: **"current truth" means the effective
revision only.** A proposition can have newer pending revisions or open
conflicts sitting alongside its effective content — `list`, `search`,
`find`, and default `show` always surface the effective (settled) content,
never a newer-but-undecided one. Treat `revisions`/`history`/`--latest`/
`--pending` output as *activity*, not yet *truth*.

## Purpose

Discovery exists so you (and future agents) don't have to re-derive
knowledge that's already been settled, and so you don't accidentally
duplicate or contradict an existing proposition. It's also how you find your
own actionable work — items waiting on your decision.

## When to Use It

- Before proposing anything new — search first, to confirm nothing close
  already exists (`fact-propose` builds on this).
- Before revising anything — confirm you have the right proposition and are
  looking at its current effective text, not a stale copy.
- When a task depends on an assumption you haven't verified against the
  ledger.
- When asked to summarize what's decided, what's outstanding, or what
  changed recently.
- When you need to act on your own decision inbox (`fact pending`), as
  opposed to browsing everything (`fact list`).
- When picking exactly one authoritative answer to compose into another
  command or another part of your work (`fact find ... --with ...`).

## When Not to Use It

- Don't use `fact find`/`fact search` when you already have an exact
  reference (ID or short reference) — go straight to `fact show`/`fact echo`
  instead; searching is for when you don't yet know the reference.
- Don't use `fact list --status pending` as a substitute for `fact pending`
  when you specifically need *your own* actionable items — `list` shows
  everything unsettled ledger-wide; `pending` is scoped to what the current
  actor is actually being asked to decide.
- Don't treat `fact revisions`/`fact history` output as the current answer —
  they're audit trails, not the effective-state surface.
- If what you actually need is to change or add knowledge, discovery has
  done its job once it confirms nothing usable exists — move on to
  `fact-propose` or `fact-revise`.

## Relationships

| Command | Answers | Notes |
| --- | --- | --- |
| `fact list` | "What's here?" — broad ledger-state view | Filterable by `--status`; default view hides some housekeeping unless `--all` |
| `fact search TEXT` | "Find anything matching this text" | Full-text; `--effective` narrows to only effective-revision text; combine with `--tag`/`--status` |
| `fact find TEXT` | "Give me the one accepted answer" | Defaults to `status=accepted, effective=true`; use `--pick N` to disambiguate multiple matches, `--with COMMAND` to chain (e.g. `--with echo`) |
| `fact pending` | "What's waiting on *me*?" | Actor-scoped actionable inbox — distinct from `list --status pending` |
| `fact show REF` | "Catch me up on this one proposition" | Composes effective content, status, recent revisions, conflicts, pending decisions, comments, tags in one view — best single command for orienting on a proposition |
| `fact echo REF` | Print effective (or `--pending`/`--latest`) content to stdout | Machine/pipe-friendly variant of `open` |
| `fact open REF` | Display effective content | Opens `$VISUAL`/`$EDITOR` if set, else prints to stdout |
| `fact export REF FILE` | Save effective (or `--pending`/`--latest`) content to a file | Requires `--force` to overwrite an existing file |
| `fact revisions REF` | List all revisions of one proposition | Audit view, not current-state view |
| `fact history [REF]` | Chronological history — one proposition, or the whole ledger if omitted | Audit view; alias `log` |

`fact tags --search TAG...` / `fact tags --list` are the tag-scoped variant
of discovery — see `fact-tags` for tag mutation and the full tag-search
syntax.

## Preconditions

- An active ledger must be resolvable (`fact-ledgers`), or pass `--ledger`.
- For `fact find --with COMMAND`, `COMMAND` must be a single bare word naming
  a real `fact` subcommand whose first positional argument is a reference —
  it re-execs `fact` as a subprocess with the resolved proposition's UUID as
  that argument; it cannot forward additional flags to the target command.

## Expected Results

- `list`/`search`/`pending` return rows with status, reference, and (in
  `--json`) full UUIDs alongside the human-readable short reference.
- `find` resolves to exactly one proposition (erroring or prompting via
  `--pick` if there are multiple matches) — treat this as "the" answer.
- `show` is the richest human-facing summary; use `--json` to consume it
  programmatically.
- `echo`/`open`/`export` return exactly the Markdown content of the selected
  revision, nothing else.

## Failure Modes

- A `REFERENCE` that matches more than one object returns an explicit
  ambiguous-reference error rather than guessing — supply a fuller reference
  or use `--pending`/`--latest` where the command supports it.
- A `REFERENCE` that matches nothing returns a clear "could not find that
  object" error.
- `fact find` with multiple matches and no `--pick` reports the candidates
  instead of guessing — read the list and re-invoke with `--pick N`.

## Safety

Every command in this skill is **read-only** — none of them create,
change, or sign anything. They are always safe to run autonomously,
including as a first step in almost any task that might touch durable
knowledge. There is no reason to ask permission before searching, listing,
or reading Facts.

---
name: fact-propose
description: >-
  Use when work has produced durable knowledge — a decision, convention, or
  constraint — that doesn't already exist anywhere in Facts and is worth
  recording for future work. Covers `fact propose` (and its identical alias
  `fact import`) for creating a brand-new proposition, including deciding it
  immediately with `--decision accept|reject`. Trigger on cues like "let's
  write this down", "remember this", "save this", "add this to memory",
  "record that we decided...", "this should be documented as a decision",
  or at the natural end of a task that settled something non-obvious.
  Facts is an available
  durable memory-write tool when the user explicitly asks to preserve
  information that belongs in a ledger. Always search first with
  fact-discovery to avoid creating a duplicate proposition for something
  that already exists — if it already exists, use fact-revise instead of
  this skill.
version: 0.1.0
---

# Fact Propose

## Concept

`fact propose` creates a new **proposition**: a fresh, independently-tracked
piece of durable knowledge with its own identity, an initial revision
containing your content, and an automatically-opened deliberation (a
consensus round) with you as its sole initial participant.

## Purpose

This is how new knowledge enters Facts. It exists so decisions, conventions,
and constraints get a stable, addressable, revisable home instead of living
only in a chat transcript, a commit message, or someone's memory.
When the user asks you to remember or save durable information and Facts is
available with a writable ledger, this is the memory-write path.

## When to Use It

- Durable knowledge was established during the task and nothing comparable
  already exists in Facts (confirm this with `fact-discovery` first —
  `search`/`find` — before proposing).
- Someone explicitly asks you to record a decision, convention, or
  constraint.
- Someone explicitly asks you to remember, save, or write down information
  for future sessions — including calling it "a memory" to add or keep —
  and that information is durable enough to belong in Facts.
- You're capturing the outcome of a design discussion, a bug investigation
  that revealed a real constraint, or an agreed convention, in a form future
  work can look up.

## When Not to Use It

- **If something close already exists**, use `fact-revise` instead —
  proposing again creates an accidental duplicate that fragments the
  knowledge base instead of updating it.
- If the knowledge isn't actually durable (see `fact-workflow`'s "what
  belongs" list), don't propose it just because the option exists.
- If you're not confident the content is finished or correct, consider
  whether it's better raised with the user first rather than proposed
  outright — proposing puts something on record, even while pending.
- `fact import` is a byte-identical alias for `fact propose` (same code
  path, only the confirmation message differs) — pick whichever reads better
  in context; there's no functional reason to prefer one.

## Relationships

- Creates a **pending** proposition by default; pass `--decision accept` or
  `--decision reject` to decide it in the same step (see `fact-decisions`
  for when that's appropriate versus leaving it pending for review).
- The proposition's first revision is automatically effective once accepted;
  if left pending, it isn't "current truth" yet — see the effective-vs-
  pending distinction in `fact-discovery`.
- To change this proposition's content later, use `fact-revise`, not another
  `fact propose` — revising preserves identity and history; proposing again
  does not.
- To retire it later without deleting history, see `fact-revise`
  (`archive`/`withdraw`).

## Preconditions

- The active ledger must be writable (not one produced by `fact clone`/
  `fact from`, which are read-only — see `fact-ledgers`).
- You've already checked `fact-discovery` and confirmed nothing comparable
  exists.
- Content is ready as **canonical Markdown**: UTF-8, NFC-normalized, LF line
  endings only, no tabs, no trailing whitespace, no CRLF. The validator also
  rejects GFM tables, tab/4-space-indented code blocks (use fenced ```
  blocks), `:::` directive blocks, and footnote definitions. Write plain
  paragraphs, headings, lists, and fenced code — anything more exotic will
  be rejected outright, not silently normalized.

## CLI Usage

```sh
fact propose --message "Short proposition text in canonical Markdown."
fact propose path/to/content.md
printf '%s' "$content" | fact propose -

fact propose --message "..." --decision accept   # create and accept in one step
fact propose --message "..." --decision reject   # create and reject in one step
```

**Always supply `--message`, a file path, or `-` with piped stdin.** Never
invoke `fact propose` bare — with none of these, it opens `$VISUAL`/
`$EDITOR` (falling back to `vi`) and blocks waiting for interactive input,
which will hang or fail in a non-interactive/agent context.

## Expected Results

- Returns the new proposition's ID (and short reference), initial status
  (`pending`, or `accepted`/`rejected` if `--decision` was given), and
  confirms you as the sole participant.
- Verify with `fact show REFERENCE` (see `fact-discovery`) — read it back to
  confirm the recorded content matches intent.

## Failure Modes

- **Canonical Markdown validation errors** — the most common failure. If
  content is rejected, check for GFM tables, indented (non-fenced) code
  blocks, tabs, trailing whitespace, or CRLF line endings, and fix the
  source text rather than retrying blindly.
- If invoked with no content source in a non-interactive context, the
  process will hang on the editor fallback — this is a caller error, not a
  CLI bug; always pass `--message` or a file.

## Safety

Proposing is a **durable append** to a shared ledger — it can't be deleted,
only retired later via `archive`/`withdraw` (see `fact-revise`), and its
history is permanent. That said, it's low-risk on its own: it doesn't assert
anything is *true* by itself (that requires a decision — see
`fact-decisions`), and on a solo ledger you're the only participant anyway.
Proposing your own newly-discovered knowledge autonomously is reasonable
when the task calls for it. Be more careful proposing on a shared ledger
where other real participants will see it, or when the content asserts
something consequential — in those cases, consider leaving it pending
(the default) rather than adding `--decision accept`, so a human gets a
chance to review before it becomes "current truth."

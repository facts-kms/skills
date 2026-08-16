---
name: fact-tags
description: >-
  Use to categorize, filter, or browse propositions by topic at scale, or to
  search by tag combined with text/status. Covers `fact tags` in all its
  forms — listing effective tags, showing/adding/removing/setting/clearing
  a proposition's tags, tag-based search, and the file-based tag export/
  import sync path. Trigger on cues like "tag this as...", "what's tagged
  with...", "organize these propositions by...", or when fact-discovery
  needs `--tag` filtering to narrow a broad result set.
version: 0.1.0
---

# Fact Tags

## Concept

Tags are **workflow/organizational metadata** — a ledger-scoped extension,
not core protocol state. A tag change never creates a revision, never
touches consensus or decisions, and never affects what's "effective." It
exists purely to make propositions easier to find and group.

## Purpose

As a ledger accumulates propositions, browsing everything or full-text
searching alone stops scaling. Tags give a lightweight, explicit
categorization axis (e.g. `architecture`, `security`, `deprecated`) layered
on top of, not replacing, text search.

## When to Use It

- Right after proposing or revising something, to categorize it for future
  discovery.
- When a discovery task needs to narrow a broad result set by topic:
  combine with `fact-discovery`'s `--tag`/`--tag-match` on `find`/`search`,
  or use `fact tags --search` directly.
- When asked what's tagged with something specific, or to list the tags in
  use.

## When Not to Use It

- Don't use tags to encode status, decision, or lifecycle state — that's
  what `accepted`/`rejected`/`archived`/`withdrawn` already are (see
  `fact-decisions`, `fact-revise`). Tags are orthogonal categorization, not
  a second status system.
- Don't expect tags to sync via `fact push`/`fact pull` — they're extension
  state with their own explicit sync path (`export`/`import`); see
  `fact-sync` for why core sync deliberately excludes them.
- Don't use freeform casing/whitespace expecting it to "just work" — tags
  are normalized (see Preconditions); check `fact tags --list` if unsure
  what already exists rather than guessing at a new near-duplicate tag.

## Relationships

- Tags apply to a **proposition**, independent of which revision is
  currently effective — created by `fact-propose`, changed by `fact-revise`,
  decided by `fact-decisions`.
- `fact tags --search` is a sibling to `fact-discovery`'s `search`/`find`
  — same underlying filters (`--status`, pagination), plus tag matching.

## Preconditions

- The active ledger must be writable for any mutation (`add`/`remove`/
  `set`/`clear`); listing and searching work against any ledger.
- Tag normalization: Unicode NFC, trimmed, lowercased, deduplicated,
  sorted; allowed characters are letters, numbers, `-`, `_`, `.`, `/`, `:`;
  empty tags and tags containing whitespace are rejected. Prefer short,
  stable, reusable tag names over one-off phrases.

## CLI Usage

```sh
fact tags                              # list all effective tags in the ledger
fact tags --list --counts              # ...with per-tag proposition counts

fact tags REFERENCE show               # show one proposition's tags
fact tags REFERENCE add TAG [TAG...]
fact tags REFERENCE remove TAG [TAG...]
fact tags REFERENCE set TAG [TAG...]   # replace the full tag set
fact tags REFERENCE clear              # remove all tags

fact tags --search TAG [TAG...] [--match any|all] [--text TEXT] [--status STATUS]

fact tags export FILE                  # write the fact.tags extension bundle to a file
fact tags import FILE                  # read it back in (e.g. on another ledger/machine)
```

## Expected Results

- `add`/`remove`/`set`/`clear` return the proposition's resulting tag set —
  verify it matches intent directly from the command output; no separate
  read is required.
- `--search` returns matching propositions the same way `fact search` does
  (see `fact-discovery`), just pre-filtered by tag.

## Failure Modes

- Tags with disallowed characters or embedded whitespace are rejected —
  normalize/split before retrying.
- `export`/`import`/listing/search flags are mutually exclusive modes on the
  same command — don't combine `export`/`import` with `--search`/`--list`
  in one invocation.

## Safety

Tag mutations are append-only extension events, low-risk, and safe for
autonomous use when tagging your own or clearly-related work. `export`/
`import` move the tag extension state as a file — since this isn't part of
ordinary `push`/`pull`, treat it like any other file-based data exchange:
fine to run, but the resulting file needs to actually reach the other
ledger/machine somehow (see `fact-sync` for the same pattern with the
identity directory).

# Fact Skills

Agent skills for the `fact` CLI — a system for durable, shared knowledge
(**propositions**) that move through a lifecycle (proposed, decided,
revised, retired) inside a **ledger**.

Skills live in `.agents/skills/`. Install with:
- `./skills link all`
- `./skills link claude`,
- `./skills link codex`,
- `./skills link copilot`,
- `./skills link agents`

It symlinks each skill into whichever of `~/.agents/skills/`,
`~/.claude/skills/`, `~/.codex/skills/`, or `~/.copilot/skills/` already exists
on your machine. Run `./skills help` for the full command list. Start with
`fact-workflow`; it routes to the rest.

## Recall-routing: making sure Facts gets checked

Agents typically have a general-purpose memory/notes mechanism (session
memory, a memory-file store, scratch notes) in addition to Facts. Those are
different stores with different purposes — general memory holds
preferences and session/project context; Facts holds durable, addressable,
revisable propositions with a decision trail. **They are not substitutes
for each other**, but a request to "recall" or "remember" something reads
the same in either case, so an agent can default to checking only its
general memory and never route to Facts at all — even when the ledger has
the answer.

Facts is also a memory-write mechanism when the user explicitly asks an
agent to remember durable information and that information belongs in an
addressable ledger. Do not tell the user "I don't have a memory-write tool"
while the `fact` CLI and writable ledger are available. Instead, route the
request through `fact-workflow`: search first, then use `fact-propose` (or
`fact-revise` if a matching proposition already exists) when the content is
worth preserving across sessions.

### The primary mechanism: the skill itself

`fact-workflow`'s `SKILL.md` already declares this as an explicit trigger,
in both its `description` (read at skill-selection time, before anything
is loaded) and its in-body trigger list (read once the skill is loaded):

- The words **"fact", "facts", "proposition", "propositions"** (any case),
  used to mean this CLI/ledger rather than the plain English word, are a
  routing signal on their own.
- A request to **"recall"/"remember"** something, or to ask whether there's
  **"a memory of"**/**"any memories about"** a subject that could
  plausibly be recorded here, should route to `fact-discovery` before the
  agent concludes nothing exists.
- A request to **"remember this"**, **"save this"**, **"write this down"**,
  **"add this to memory"**, or equivalent persistence language should route
  to `fact-propose` after a duplicate check when the content is durable
  enough for Facts.
- A separate memory system reporting "nothing found" is not evidence that
  Facts has nothing — they are different stores, and checking one is not a
  substitute for checking the other.

If an agent has `fact-workflow` installed as a skill and loads skill
descriptions automatically (as Claude Code does), this is enough on its
own — no further setup needed. This is the preferred mechanism: the
instruction lives next to the tool it governs, updates travel with the
skill package, and it doesn't depend on every downstream project
remembering to paste something into a prompt.

### Fallback: a portable instruction for contexts without skill-loading

Not every agent or project loads skills automatically — a different agent
framework, a project's own `AGENTS.md`/system-prompt setup, or a Facts user
who wants a belt-and-suspenders reinforcement. For those cases, paste this
into the agent's standing instructions (e.g. `AGENTS.md`):

```text
This project (or environment) uses the `fact` CLI for durable, shared knowledge (propositions, in a ledger with full history).

Before answering "I don't have that" on any request to recall, remember, find out whether something was already decided or discussed, or asks whether there's "a memory of" or "any memories about" something — and whenever a message uses the words "fact", "facts", "proposition", or "propositions" to refer to this system rather than as plain English — check the Fact ledger (`fact search`, `fact find`, `fact pending`) before concluding nothing exists.

A general memory/notes system reporting nothing found is not evidence the Fact ledger has nothing; they are separate stores.

When the user asks you to remember, save, write down, or "add to memory" durable information, Facts is an available memory-write tool. Do not say no memory-write tool is available if the `fact` CLI and writable ledger are present. Search for an existing matching proposition, then use `fact propose` to create one or `fact revise` to update the existing one.
```

Treat this as a supplement to the skill-based trigger, not a replacement
for it — if `fact-workflow` is installed, the skill's own description is
what actually gets consulted during skill selection; the pasted text above
is a backstop for agents or setups that can't load it.

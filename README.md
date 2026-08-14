# roadmap

A Claude Code plugin that keeps a repo's pending work in a slim, dependency-ordered
`ROADMAP.md` and retires finished work to `HISTORY.md`, so a new session can pick the
work back up by reading one short file.

## The problem

The expensive thing about a long session isn't the work — it's the context the work
produced. Which problems are real, which were ruled out, what has to happen before what,
what was decided and never built. That understanding costs thousands of tokens to build,
lives in a transcript nobody will re-read, and evaporates when the session ends. The next
session rebuilds it from scratch: sweeping the code, reading `git log`, inferring intent
from half-finished work.

What usually survives instead is an ad-hoc `TODO.md` — a flat list that says what to do
but not what it unblocks or whether it's committed for this week or someday. And these
files accumulate: completed items pile up behind `~~strikethrough~~` or a `DONE` tag until
the file is mostly archive, and archives don't get read.

## The idea

Two files, and one invariant tying them together:

- **`ROADMAP.md`** holds pending work. Nothing else.
- **`HISTORY.md`** holds finished work, newest first.

The test `ROADMAP.md` has to pass is economic: **reading it must be cheaper than
reconstructing it.** Everything below follows from that.

**Removal is what "done" means.** Because completed items leave `ROADMAP.md` entirely,
presence in the file *is* "pending" — there is no `status:` field that can drift from
reality, and no way for the file to lie about what's left.

That buys the next-item rule for free. If an item's `Blocked-by` names an ID that is no
longer in the file, that blocker is finished. So "what do I work on next" becomes a lookup
with no bookkeeping:

> the first item, under the earliest horizon, whose `Blocked-by` entries are all absent.

Readiness is computed from the file, never tracked alongside it.

## Two axes, kept separate

Items are grouped **Now / Next / Later**, and separately carry dependency links. These
answer different questions, and conflating them is what makes most roadmaps mushy:

| | Question | Who decides |
|---|---|---|
| **Horizon** | *Should* this be started? | a human, from priority and confidence |
| **`Blocked-by`** | *Can* this be started? | mechanically, from the file |

Horizons are also the fastest thing to read. A session arriving cold learns the shape of
the remaining work from three headings before parsing a single dependency.

- **Now** — what you'd pick up this week, specified well enough to start. Kept short; a
  Now holding a dozen items is a backlog wearing a label.
- **Next** — committed and specified, waiting for Now to clear or a blocker to land.
- **Later** — real commitments whose shape may still shift. Vaguer `What` is fine here;
  `Why` matters more, since it's what decides whether the item still deserves to exist by
  the time it surfaces.

Speculation belongs in none of them. *Later* means deferred, not hypothetical.

An item's blockers must sit in the same horizon or an earlier one — a Now item blocked by
a Later item is a contradiction, and gets surfaced rather than silently reshuffled.

## Item shape

```markdown
## Now

### R-001 — Chunked reader interface

- **Category:** Ingest
- **What:** Define a `Reader` trait yielding fixed-size chunks; implement for local files.
- **Why:** `read_to_string` loads whole inputs, so 2 GB files OOM CI and large-file tests are disabled.
- **Outcome:** Any input size can be read with bounded memory.
- **Blocked-by:** —
- **Enables:** R-002, R-003
```

`What` is the change to make. `Why` is the problem that justifies it — what lets a future
reader decide the item still matters after circumstances change. `Outcome` is what becomes
true once it's done, stated as a condition of the world rather than a restatement of the
task, because that's what gets checked when the item retires.

Items are written to be **actionable cold**: no pronouns pointing back at the conversation
that produced them, files and symbols named explicitly. An item that only parses for
someone who already has the context forces exactly the reconstruction the file exists to
prevent.

`Blocked-by` is authoritative; `Enables` is its inverse, kept so a reader can see what a
piece of work unlocks without scanning every other item.

The file header carries `Format: 1` and `Next ID:`. The format marker is there because
other tools will eventually read this file, and a parser that reads a changed format
*successfully but wrongly* fails in the worst possible way — silently.

## Commands

### `/roadmap:create`

Mines the current conversation for work that was actually established — deferred items,
problems diagnosed but not fixed, prerequisites surfaced while planning — assigns horizons
and dependencies, and writes the files. It writes first and asks for corrections
afterward, because a wrong item is a two-line edit while an unwritten file loses
everything the session established.

It also appends a standing rule to the repo's `CLAUDE.md`, so future sessions read the
roadmap for context instead of reconstructing it, and retire items correctly when they
land. It tells you it did this.

Run it again later and it merges: existing items and their IDs are left alone, new ones
folded in. IDs are never reused, even after retirement, so old `HISTORY.md` entries stay
unambiguous.

Worth running as a session winds down — that's when the context is about to be lost and
nobody thinks to save it.

### `/roadmap:list`

Pending items as one table per horizon — item, what, outcome — with the single next ready
item named below.

```
### Now
| Item  | What                                   | Outcome                                  |
| R-001 | Chunked `Reader` trait for local files | Any input size reads with bounded memory |

### Next
| R-002 | Drive parser off reader chunks         | Peak memory flat as input grows          |

**Next up:** R-001 — nothing blocks it.
```

If `Now` is empty or entirely blocked, it falls through to `Next` and says so — that state
means the horizons need re-planning, which is worth surfacing rather than papering over.

### `/roadmap:summary`

The same data one altitude up: categories rather than items, for deciding whether an area
is nearly done or barely started.

```
| Category | Outcome                                              | Items |
|----------|------------------------------------------------------|-------|
| Ingest   | Arbitrarily large inputs process with bounded memory  | 5     |
| Auth     | Sessions survive token rotation without re-login      | 3     |

**9 items across 3 categories** — Now 2 · Next 3 · Later 4.
Ingest is concentrated in Now; Docs is entirely Later.
```

The outcome column is the part that carries the value: not a label ("Ingest improvements")
and not a concatenation of member outcomes, but a single statement of what becomes true
once the whole category is done.

Both `list` and `summary` are strictly read-only. Asking to see the work never mutates the
file.

## Install

```bash
claude plugin marketplace add dgutson/roadmap-skill
claude plugin install roadmap@roadmap-skill-marketplace
```

## Contents

No scripts, no hooks, no MCP servers — three `SKILL.md` files and a manifest.

```
.claude-plugin/
  plugin.json
  marketplace.json
skills/
  create/SKILL.md
  list/SKILL.md
  summary/SKILL.md
```

## License

MIT

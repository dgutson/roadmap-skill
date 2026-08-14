# roadmap

A Claude Code plugin that keeps a repo's pending work in a slim,
dependency-ordered `ROADMAP.md` and retires finished work to `HISTORY.md`, so
Claude can always pick the next thing to build without guessing.

## The problem

Plans made in a chat die with the chat. What survives is usually an ad-hoc
`TODO.md` — a flat list that says what to do but not what it unblocks, so
choosing the next task means re-deriving the dependency graph from memory every
time. Worse, these files accumulate. Completed items pile up with `~~strikethrough~~`
or a `DONE` tag until the file is mostly archive, and archives don't get read.

## The idea

Two files, and one invariant that ties them together:

- **`ROADMAP.md`** holds pending work. Nothing else.
- **`HISTORY.md`** holds finished work, newest first.

**Removal is what "done" means.** Because completed items leave `ROADMAP.md`
entirely, presence in the file *is* "pending" — there is no `status:` field that
can drift from reality, and no way for the file to lie about what's left.

That buys the next-item rule for free. If an item's `Blocked-by` names an ID that
is no longer in the file, that blocker is finished. So "what do I work on next"
becomes a lookup with no bookkeeping:

> the first item in the file whose `Blocked-by` entries are all absent.

Readiness is computed from the file, never tracked alongside it. File order is
therefore the priority decision, not decoration.

## Item shape

```markdown
## R-001 — Chunked reader interface

- **Category:** Ingest
- **What:** Define a `Reader` trait yielding fixed-size chunks; implement for local files.
- **Why:** `read_to_string` loads whole inputs, so 2 GB files OOM CI and large-file tests are disabled.
- **Outcome:** Any input size can be read with bounded memory.
- **Blocked-by:** —
- **Enables:** R-002, R-003
```

`What` is the change to make. `Why` is the problem that justifies it — what lets a
future reader decide the item still matters after circumstances change. `Outcome`
is what becomes true once it's done, stated as a condition of the world rather
than a restatement of the task, because that's what gets checked when the item
retires.

`Blocked-by` is authoritative; `Enables` is its inverse, kept so a reader can see
what a piece of work unlocks without scanning every other item.

## Commands

### `/roadmap:create`

Mines the current conversation for work that was actually established — deferred
items, problems diagnosed but not fixed, prerequisites surfaced while planning —
and writes the roadmap. It shows you the proposed items for one round of
corrections before writing anything, because conversation-mining is lossy in both
directions.

It also appends a short standing rule to the repo's `CLAUDE.md`, so the
retire-on-completion behavior is loaded in every future session, including
sessions that never invoke this plugin. It tells you it did this.

Run it again later and it merges: existing items and their IDs are left alone,
new ones are folded in and the file re-sorted. IDs are never reused, even after
retirement, so old `HISTORY.md` entries stay unambiguous.

### `/roadmap:list`

The pending items as a table — item, what, outcome — in file order, with the next
ready item named below it.

```
| Item  | What                                   | Outcome                                   |
|-------|----------------------------------------|-------------------------------------------|
| R-001 | Chunked `Reader` trait for local files | Any input size reads with bounded memory  |
| R-002 | Drive parser off reader chunks         | Peak memory flat as input grows           |

**Next up:** R-001 — nothing blocks it.
```

### `/roadmap:summary`

The same data one altitude up: categories rather than items, for deciding whether
an area is nearly done or barely started.

```
| Category | Outcome                                              | Items |
|----------|------------------------------------------------------|-------|
| Ingest   | Arbitrarily large inputs process with bounded memory  | 5     |
| Auth     | Sessions survive token rotation without re-login      | 3     |
```

The outcome column is the part that carries the value: not a label
("Ingest improvements") and not a concatenation of the members' outcomes, but a
single statement of what becomes true once the whole category is done.

Both `list` and `summary` are strictly read-only. Asking to see the work never
mutates the file.

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

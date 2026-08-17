---
name: list
description: Render the pending items in ROADMAP.md as compact tables of item, what, and outcome — grouped Now / Next / Later — and name which single item is ready to be started next. Strictly read-only; it never edits the roadmap. Use whenever the user runs /roadmap:list, or asks "what's on the roadmap", "what's left to do", "what should I work on next", "where were we", "what's still open here", or otherwise wants to pick up outstanding work without digging through code or old conversations. Prefer this over dumping ROADMAP.md into the conversation, since the raw file carries dependency bookkeeping the reader does not need to wade through.
effort: high
---

# List roadmap items

Read `ROADMAP.md` from the repo root and render the pending items as tables grouped by
horizon.

If the file doesn't exist, say so and point the user at `/roadmap:create` rather than
inventing a list or scanning the code for TODOs — an improvised list would compete with the
real file the moment one gets created, and the whole point of the file is to be the one
place worth trusting.

This command is **read-only**. Never edit, reorder, or tidy the roadmap while listing it;
someone asking to see the work should not have their file mutated as a side effect.

## Output

One table per horizon, in `Now` → `Next` → `Later` order, skipping any that are empty.
Preserve file order within each — it encodes priority, so each table reads top-down as the
intended sequence.

```markdown
### Now

| Item | Ready? | What | Outcome |
|---|---|---|---|
| R-001 | ✅ | Chunked `Reader` trait for local files | Any input size reads with bounded memory |

### Next

| Item | Ready? | What | Outcome |
|---|---|---|---|
| R-002 | ❌ | Drive parser off reader chunks | Peak memory flat as input grows |

### Later

| Item | Ready? | What | Outcome |
|---|---|---|---|
| R-003 | ❌ | Re-enable large-file test suite | Regressions on 2 GB inputs caught in CI |

**Next up:** R-001 — nothing blocks it.
```

**Ready?** is `✅` when the item's **Blocked-by** is empty or names only IDs absent from the
file, `❌` otherwise — the same test used to name the next item below.

The horizons do most of the orienting work for a reader arriving cold, which is why they
lead. Someone who has been away for weeks learns the shape of the remaining work from three
headings, before reading a single row.

## Naming the next item

Below the tables, name the next item: the first one, under the earliest horizon, whose
**Blocked-by** entries are empty or name only IDs absent from the file (absent means
finished and retired to HISTORY.md).

Name exactly one. Other items are often unblocked too, and it's useful to mention them —
but as "also unblocked", never as co-equal next. File order within a horizon is the
roadmap's priority decision, so presenting two items as equally next quietly discards it
and hands the choice back to whoever is reading, which is the guesswork this format exists
to remove.

If **Now** is empty or every item in it is blocked, fall through to `Next`, then `Later` —
and say that you fell through. That state means the horizons need re-planning, which is a
planning signal worth surfacing rather than a routine lookup to paper over. If every item
in the file is blocked, say so plainly and name what they're all waiting on.

## Keeping the tables readable

Compress **What** and **Outcome** to roughly one line each. The file's prose is written to
be unambiguous months later; a table is written to be scanned. Shorten by cutting detail,
never by inventing or overstating — if an item's what and outcome can't be told apart once
shortened, the item is probably poorly written, and saying so is more useful than rendering
a row where both columns repeat each other.

Leave **Why** out entirely. It's the field that justifies keeping an item, which matters
when pruning the roadmap, not when scanning for the next task.

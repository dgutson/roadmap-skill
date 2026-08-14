---
name: list
description: Render the pending items in ROADMAP.md as a compact table of item, what, and outcome, and name which item is ready to be started next. Strictly read-only — it never edits the roadmap. Use whenever the user runs /roadmap:list, or asks "what's on the roadmap", "what's left to do", "what should I work on next", "show me the pending items", "what's still open here", or otherwise wants to see the roadmap without opening the file. Prefer this over dumping ROADMAP.md into the conversation, since the raw file carries dependency bookkeeping the reader does not need to wade through.
---

# List roadmap items

Read `ROADMAP.md` from the repo root and render every pending item as a table.

If the file doesn't exist, say so and point the user at `/roadmap:create` rather than
inventing a list or scanning the code for TODOs — an improvised list would compete with
the real file the moment one gets created.

This command is **read-only**. Never edit, reorder, or tidy the roadmap while listing it;
someone asking to see the work should not have their file mutated as a side effect.

## Output

Preserve the file's order — it encodes readiness, so the table reads top-down as the
intended sequence of work.

```markdown
| Item | What | Outcome |
|---|---|---|
| R-001 | Chunked `Reader` trait for local files | Any input size reads with bounded memory |
| R-002 | Drive parser off reader chunks | Peak memory flat as input grows |
| R-003 | Re-enable large-file test suite | Regressions on 2 GB inputs caught in CI |

**Next up:** R-001 — nothing blocks it.
```

Then, below the table, name the next item: the first one whose **Blocked-by** entries are
empty or name only IDs absent from the file (absent means finished and retired to
HISTORY.md).

Name exactly one. Other items are often unblocked too, and it's useful to mention them —
but as "also unblocked", never as co-equal next. File order is the roadmap's priority
decision, so presenting two items as equally next quietly discards it and hands the choice
back to whoever is reading, which is the guesswork this whole format exists to remove.

If every item is blocked, say so plainly and name what they're waiting on — that's a
stalled roadmap, and it's worth surfacing rather than papering over.

## Keeping the table readable

Compress **What** and **Outcome** to roughly one line each. The file's prose is written
to be unambiguous months later; a table is written to be scanned. Shorten by cutting
detail, never by inventing or overstating — if an item's what and outcome can't be
distinguished once shortened, the item is probably poorly written, and it's more useful to
say so than to render a row where both columns repeat each other.

Leave **Why** out entirely. It's the field that justifies keeping an item, which matters
when pruning the roadmap, not when scanning for the next task.

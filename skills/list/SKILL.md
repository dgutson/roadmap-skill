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

A heading per horizon (`### Now`, `### Next`, `### Later`), and under each a table whose
columns are, in order: **Item**, **Ready?**, **What**, **Outcome** — one row per item. So a
`Now` row carries `R-001` · `✅` · "Chunked `Reader` trait for local files" · "Any input
size reads with bounded memory", and **Next up:** follows below the tables.

**That is the table's content, and deliberately not its markup.** How a table should be
drawn depends on where the reply is read: a terminal refolds a long row and the column
alignment collapses, which is worst for exactly the wide rows a roadmap produces. Something
in the environment may already render tables properly, and prescribing pipe-and-dash syntax
here would override it and hand the reader the broken version instead — that has actually
happened. So say what the columns hold and leave the drawing to whatever draws tables;
where nothing does, a plain markdown table remains a fine fallback. An optional renderer,
never a dependency.

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

**Scope "also unblocked" to the next item's own horizon, and name at most five.** Every
ready item in the file is rarely news: on a roadmap of any size most of `Later` is unblocked
too, having never been blocked in the first place. Listing it all buries the one item that
was actually asked for, and pads a short answer with a long one. If the horizon holds more
than five others, name five and say how many remain.

If **Now** is empty or every item in it is blocked, fall through to `Next`, then `Later` —
and say that you fell through. That state means the horizons need re-planning, which is a
planning signal worth surfacing rather than a routine lookup to paper over. If every item
in the file is blocked, say so plainly and name what they're all waiting on.

## Keeping the tables readable

**The What column is the item's `###` heading title, copied verbatim** — not its **What**
field compressed down. The title is already the one-line form, written deliberately by
whoever created the item. Re-compressing the What field instead pays, on every single
listing, to re-derive a sentence that already exists, and rewords the roadmap's own
vocabulary so the table stops matching the file people search.

Shorten an **Outcome** only if it genuinely runs long, and then by cutting detail, never by
inventing or overstating.

Say so rather than silently compensating when:

- a row's what and outcome say the same thing — the *item* is badly written, and naming that
  is more useful than rendering a row whose two columns repeat each other;
- a title is too vague to scan (`Fix the parser`) — that item needs a better heading. It is
  fixed by editing the item once, not by re-inferring a title on every listing.

Leave **Why** out entirely. It's the field that justifies keeping an item, which matters
when pruning the roadmap, not when scanning for the next task.

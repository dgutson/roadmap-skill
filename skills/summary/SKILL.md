---
name: summary
description: Roll ROADMAP.md up into a category-level table — each category, the outcome reached once that whole category is done, and how many items remain in it — followed by the Now / Next / Later distribution. Strictly read-only. Use whenever the user runs /roadmap:summary, or asks "what's the big picture", "give me the high-level view", "what are the major areas still outstanding", "how much is left overall", "summarize the roadmap", or wants something stakeholder-facing rather than a task list. Reach for this instead of /roadmap:list once the roadmap is long enough that individual items obscure the shape of the remaining work.
effort: high
---

# Summarize the roadmap

Read `ROADMAP.md` from the repo root and roll the pending items up to their categories.

If the file doesn't exist, say so and point the user at `/roadmap:create`.

This command is **read-only** — never edit or reorganize the roadmap while summarizing it.

Where `/roadmap:list` answers *"what are the specific things to do?"*, this answers *"what
are the big things still outstanding?"* — same data, one altitude up. The reader here is
usually deciding whether an area is nearly done or barely started, not picking up a task.

## Output

Group items by their **Category** field — across all horizons, since a category's scope
doesn't change because part of it was deferred — and emit one row per category, ordered by
item count descending so the heaviest remaining areas surface first.

```markdown
| Category | Outcome | Items |
|---|---|---|
| Ingest | Arbitrarily large inputs process with bounded memory | 5 |
| Auth | Sessions survive token rotation without re-login | 3 |
| Docs | Public API documented well enough to onboard without reading source | 1 |

**9 items across 3 categories** — Now 2 · Next 3 · Later 4.
Ingest is concentrated in Now; Docs is entirely Later. The next ready item, R-001, is in *Ingest*.
```

Close with the totals, the horizon distribution, and which category holds the next ready
item. Note where a category concentrates in one horizon when it's informative — a category
living entirely in Later is deferred wholesale, and one packed into Now is what the team is
actually doing. That distinction is most of what a high-level reader wants, and it's
invisible in the counts alone.

Ending on the next ready item connects this view back to what will actually get worked on,
so the summary informs a decision instead of just describing a pile.

## Writing the category outcome

This is the part that carries the value, and the part that's easy to do badly. The outcome
column is **not** a concatenation of the members' outcome fields, and not a theme label like
"parser improvements". It's a single statement of what becomes true about the system once
every item in that category is finished.

Derive it by reading the members' Outcome fields together and asking what capability they
add up to:

- **Weak** (a label): "Ingest improvements"
- **Weak** (a list): "Chunked reader, streaming parse, and re-enabled tests"
- **Strong** (a capability): "Arbitrarily large inputs process with bounded memory"

If a category's items don't add up to anything coherent, that's a real finding about the
roadmap rather than a formatting problem — write the closest honest outcome you can and note
that the category looks like a grab-bag. Categories that have stopped meaning anything are
worth knowing about, and quietly inventing a tidy outcome would hide it.

Never invent categories or re-bucket items to make the table look neater. The categories
belong to the roadmap; the summary reports them.

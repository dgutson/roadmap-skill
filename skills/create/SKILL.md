---
name: create
description: Turn the current conversation into a ROADMAP.md of pending work — each item carrying an ID, category, what/why/outcome, and explicit blocked-by/enables links so Claude can pick the next thing to build without guessing. Also creates HISTORY.md for finished work and installs a standing rule in the project's CLAUDE.md that keeps the two files in sync. Use whenever the user runs /roadmap:create, or says anything like "turn this into a roadmap", "let's write down everything we decided", "capture this plan somewhere", "we should track this work", or asks to fold newly discussed work into an existing roadmap. Prefer this over writing an ad-hoc TODO or plan file — a flat list cannot express what enables what, so choosing the next item degrades into guesswork.
---

# Create a roadmap

You are turning what this conversation already established into two files that outlive it:

- **ROADMAP.md** — work that is still pending. Nothing else.
- **HISTORY.md** — work that is finished, newest first.

## Why the split matters

A roadmap that accumulates completed items becomes an archive, and archives don't get
read. Keeping ROADMAP.md pending-only means its length is an honest signal of how much
work remains, and every item you scan is a live decision rather than a memorial.

The split also removes the need for status flags. Presence in ROADMAP.md *is* "pending";
moving an item to HISTORY.md *is* "done". There is no `status: complete` field to fall
out of sync with reality, and no way for the file to lie about what's left.

That single rule has a useful consequence you will rely on throughout: **if an item's
`Blocked-by` names an ID that is no longer in ROADMAP.md, that blocker is finished.**
Readiness is computed from the file, not tracked separately.

## Step 1 — Check what already exists

Read `ROADMAP.md`, `HISTORY.md`, and `CLAUDE.md` in the repo root if they are there.

If **ROADMAP.md already exists**, you are in merge mode: never overwrite it. Keep every
existing item and its ID exactly as-is, add the new items from this conversation, re-sort,
and bump the ID counter. Existing IDs are referenced by HISTORY.md and by anything else
the team wrote down — renumbering them silently breaks those references.

## Step 2 — Harvest the items from this conversation

Go back through the conversation and pull out work that was *established*, not merely
mentioned. Good sources:

- Things explicitly deferred — "let's do that later", "not now", "in a follow-up"
- Problems diagnosed but not fixed
- Design decisions that imply build work
- Gaps the user named, or that the code review surfaced and nobody addressed
- Prerequisites you identified while planning something else

Do not invent work to round out the list, and do not promote speculation ("we could
maybe someday…") into a committed item. A roadmap the user doesn't recognize is worse
than no roadmap, because they'll stop trusting the file and go back to reading the code.

If the conversation is thin on concrete work, say so rather than padding.

## Step 3 — Shape each item

Every item needs an ID, a category, and three prose fields. The fields do different jobs,
so resist collapsing them into one another:

- **What** — the change to make, concretely enough that someone could start.
- **Why** — the problem that makes it worth doing. This is what lets a future reader
  (or a future Claude) decide whether the item still matters after circumstances change.
- **Outcome** — what becomes true once it's done. State it as a condition of the world,
  not as a restatement of the task, because the outcome is what you check against when
  the item eventually retires to HISTORY.md.

Weak vs. strong:

| Field | Weak | Strong |
|---|---|---|
| What | "Fix the parser" | "Replace `read_to_string` in `ingest/reader.rs` with a chunked reader" |
| Why | "It's better" | "A 2 GB input OOMs the CI box, so large-file tests are disabled" |
| Outcome | "Parser is fixed" | "Any input size parses with bounded memory; large-file tests re-enabled" |

**Categories** group items for `/roadmap:summary`. Use a handful of durable areas of the
system (`Ingest`, `Auth`, `CLI`, `Docs`) rather than one category per item — a category
that holds a single item tells the summary reader nothing.

**IDs** are `R-001`, `R-002`, … allocated in order and **never reused**, even after an
item retires to HISTORY.md. Reuse would make old history entries ambiguous about which
work they describe. Track the next free number in the `Next ID:` line at the top of the
file; when merging, trust that counter over the highest ID you can see, since higher IDs
may already have retired into HISTORY.md.

## Step 4 — Wire the dependencies and order the file

For each item, record which other items must land first:

- **Blocked-by** — IDs that must be gone from ROADMAP.md before this can start. Use `—`
  when nothing blocks it.
- **Enables** — IDs this unblocks. This is the inverse of Blocked-by and exists purely so
  a reader can see what a piece of work unlocks without scanning every other item.
  **Blocked-by is authoritative**; regenerate Enables from it so the two can't drift.

Then sort the file into **readiness order**: blockers before the things they block, and
among items that are equally ready, put the one that unlocks the most downstream work
first, breaking remaining ties by the user's stated priority.

Order carries real weight here, because the rule for picking work is *"the first item in
the file whose Blocked-by list is empty or names only absent IDs."* File order is
therefore the priority decision, not decoration — arrange it deliberately.

## Step 5 — Confirm before writing

Show the user a compact numbered list — one line per item, `ID — title (category)` — and
ask whether anything is missing, wrong, or shouldn't be there. Conversation-mining is
lossy in both directions, and this is far cheaper than having them discover a bad item
three weeks later.

Keep this to one round. It's a check, not an interview.

If the session is **non-interactive** — `claude -p`, a cron run, a subagent, anything
where your reply won't reach someone who can answer — don't stop for confirmation. Write
the files, then show the same list in your final report so it can be corrected afterward.
A roadmap that was never written isn't safer than one carrying a wrong item: the wrong
item is visible and editable, whereas the missing file silently loses everything the
conversation established.

## Step 6 — Write the files

`ROADMAP.md`:

```markdown
# Roadmap

> Pending work only. Finished items move to HISTORY.md.
> The next thing to work on is the first item below whose **Blocked-by** entries are
> no longer present in this file.

Next ID: R-004

---

## R-001 — Chunked reader interface

- **Category:** Ingest
- **What:** Define a `Reader` trait yielding fixed-size chunks; implement it for local files.
- **Why:** `read_to_string` loads whole inputs, so 2 GB files OOM the CI box and the large-file tests are disabled.
- **Outcome:** Any input size can be read with bounded memory.
- **Blocked-by:** —
- **Enables:** R-002, R-003

## R-002 — Streaming parse path

- **Category:** Ingest
- **What:** Drive the parser off `Reader` chunks instead of a single in-memory buffer.
- **Why:** The chunked reader buys nothing while the parser still materializes the full document.
- **Outcome:** Peak memory stays flat as input size grows.
- **Blocked-by:** R-001
- **Enables:** R-003
```

`HISTORY.md` — create it now, with just the header, so the pair is obviously a pair:

```markdown
# History

> Completed roadmap items, newest first.
```

## Step 7 — Install the standing rule

The roadmap only stays true if it's updated as work lands, and a skill can't watch for
that on its own. Append this to the repo's `CLAUDE.md` (creating the file if absent), so
the rule is loaded in every future session — including sessions that never invoke this
plugin:

```markdown
## Roadmap

This repo is governed by ROADMAP.md (pending work) and HISTORY.md (completed work).

- To choose what to work on next, read ROADMAP.md and take the first item whose
  **Blocked-by** entries are no longer present in the file.
- When you finish an item: delete it from ROADMAP.md, add a line under today's date at
  the top of HISTORY.md recording the outcome **actually** achieved, and drop its ID from
  the **Blocked-by** list of every item it was blocking.
- ROADMAP.md holds pending work only. Never mark an item done in place — removal is what
  "done" means here.
```

If an equivalent block is already present, leave it alone. Either way, tell the user in
your summary that you touched CLAUDE.md — editing a file that shapes every future session
should never be a silent side effect.

## Step 8 — Report

State how many items you captured, which one is ready to start first, and which files you
created or modified. Then stop; don't start working the roadmap unless asked.

## Retiring an item later

When an item is finished, the entry that lands in HISTORY.md records what actually
happened, which is not always what the Outcome field predicted:

```markdown
## 2026-08-14

- **R-001** Chunked reader interface — `Reader` trait landed with local + S3 impls; large-file tests re-enabled.
```

Where reality diverged from the predicted outcome, write reality and note the gap. A
history that only echoes the original plan can't teach anyone anything later.

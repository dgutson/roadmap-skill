---
name: create
description: Capture what this conversation established into a ROADMAP.md of pending work, grouped Now / Next / Later, each item carrying an ID, category, what/why/outcome and explicit blocked-by/enables links — so a later session can pick the work back up by reading one short file instead of re-deriving it from the code, the git log, or a conversation that no longer exists. Also creates HISTORY.md for finished work and installs a standing rule in the project's CLAUDE.md that keeps the two in sync. Use whenever the user runs /roadmap:create, or says anything like "turn this into a roadmap", "capture this plan", "write down what we decided", or "we should track this work" — and suggest it proactively when a long session has established real work and is winding down ("let's stop here", "continue tomorrow", "I'm starting a new chat"), since that is precisely when the context is about to be lost and nobody thinks to save it. Prefer this over an ad-hoc TODO file: a flat list can express neither what enables what, nor what is committed now versus deferred.
---

# Create a roadmap

You are turning what this conversation established into two files that outlive it:

- **ROADMAP.md** — work that is still pending. Nothing else.
- **HISTORY.md** — work that is finished, newest first.

## Why this file exists

The expensive thing about a long session is not the work — it's the context the work
produced. Which problems are real, which were ruled out, what has to happen before what,
what was decided and never built. That understanding cost thousands of tokens to build,
lives in a transcript nobody will re-read, and disappears the moment the session ends.
The next session then rebuilds it from scratch: sweeping the code, reading git log,
inferring intent from half-finished work.

ROADMAP.md exists to make that rebuild unnecessary. It is the durable record of
established-but-unfinished work, and the test it has to pass is economic: **reading it
must be cheaper than reconstructing it.** Every choice below serves that test — which is
also why items have to be written for a reader who wasn't here.

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
existing item and its ID exactly as-is, add the new items from this conversation, re-sort
within horizons, and bump the ID counter. Existing IDs are referenced by HISTORY.md and by
anything else the team wrote down — renumbering them silently breaks those references.

## Step 2 — Harvest the items from this conversation

Go back through the conversation and pull out work that was *established*, not merely
mentioned. Good sources:

- Things explicitly deferred — "let's do that later", "not now", "in a follow-up"
- Problems diagnosed but not fixed
- Design decisions that imply build work
- Gaps the user named, or that a review surfaced and nobody addressed
- Prerequisites you identified while planning something else

Do not invent work to round out the list, and do not promote speculation ("we could maybe
someday…") into a committed item. A roadmap the user doesn't recognize is worse than no
roadmap, because they'll stop trusting the file and go back to reading the code — which
is the expense this whole thing exists to avoid.

If the conversation is thin on concrete work, say so rather than padding.

## Step 3 — Shape each item

Every item needs an ID, a category, and three prose fields. They do different jobs, so
resist collapsing them into one another:

- **What** — the change to make, concretely enough that someone could start.
- **Why** — the problem that makes it worth doing. This is what lets a future reader
  decide whether the item still matters after circumstances change.
- **Outcome** — what becomes true once it's done. State it as a condition of the world,
  not a restatement of the task, because the outcome is what you check against when the
  item eventually retires to HISTORY.md.

Weak vs. strong:

| Field | Weak | Strong |
|---|---|---|
| What | "Fix the parser" | "Replace `read_to_string` in `ingest/reader.rs` with a chunked reader" |
| Why | "It's better" | "A 2 GB input OOMs the CI box, so large-file tests are disabled" |
| Outcome | "Parser is fixed" | "Any input size parses with bounded memory; large-file tests re-enabled" |

**Write every item to be actionable cold.** The reader you are writing for did not attend
this conversation — and in three weeks, that includes the user. So: no pronouns pointing
back at the chat ("the thing we discussed", "that bug", "the approach we agreed on"), name
files and symbols explicitly, and state the problem in **Why** rather than assuming it's
remembered. An item that only parses for someone who already has the context fails the
economic test in *Why this file exists*: it forces exactly the reconstruction the file was
written to prevent.

**Categories** group items for `/roadmap:summary`. Use a handful of durable areas of the
system (`Ingest`, `Auth`, `CLI`, `Docs`) rather than one category per item — a category
holding a single item tells the summary reader nothing.

**IDs** are `R-001`, `R-002`, … allocated in order and **never reused**, even after an item
retires to HISTORY.md, since reuse would make old history entries ambiguous about which
work they describe. Track the next free number in the `Next ID:` line; when merging, trust
that counter over the highest ID you can see, because higher IDs may already have retired.

## Step 4 — Sort items into Now / Next / Later

Horizons answer *"should this be started?"* — a human judgment about priority and
confidence. Dependencies (Step 5) answer *"can this be started?"* — a mechanical fact.
Keeping the two separate is what stops file position from quietly meaning both at once.

- **Now** — what you'd pick up today or this week, specified well enough to start without
  further decisions. Keep it short. A Now holding a dozen items isn't a horizon, it's a
  backlog wearing a label, and it stops telling anyone anything.
- **Next** — committed and specified, but waiting for Now to clear or for a blocker to land.
- **Later** — real commitments whose shape may still shift. A vaguer **What** is acceptable
  here; the **Why** matters more, because Why is what lets a future reader judge whether
  the item is still worth doing by the time it surfaces.

Speculation belongs in none of them. "Later" means deferred, not hypothetical — the moment
Later becomes a wishlist, the file stops being a record of commitments.

Horizons are also the fastest thing to read, which is why they earn their place: a session
starting cold learns the shape of the work from three headings before parsing a single
dependency.

**Consistency check:** an item's blockers must sit in the same horizon or an earlier one.
A Now item blocked by a Later item is a contradiction — something committed for this week
depends on work explicitly deferred. Surface it and ask which horizon is wrong rather than
silently moving either, because both readings are plausible and only the user knows which.

## Step 5 — Wire the dependencies and order within each horizon

For each item, record which other items must land first:

- **Blocked-by** — IDs that must be gone from ROADMAP.md before this can start. Use `—`
  when nothing blocks it.
- **Enables** — IDs this unblocks. The inverse of Blocked-by, kept so a reader can see what
  a piece of work unlocks without scanning every other item. **Blocked-by is
  authoritative**; regenerate Enables from it so the two can't drift.

Then order items *within* each horizon: blockers before the things they block, and among
equally ready items, the one that unlocks the most downstream work first, breaking ties by
the user's stated priority.

Order carries real weight, because the rule for picking work is *"the first item under the
earliest horizon whose Blocked-by list is empty or names only absent IDs."* Within a
horizon, file order is the priority decision — arrange it deliberately.

## Step 6 — Write the files

Write them now. Do **not** stop first to ask whether the item list is right — you'll ask
immediately afterward, in Step 8, once the work is safely on disk.

That ordering is deliberate. Conversation-mining is lossy in both directions, so the list
does need checking — but a roadmap is an ordinary file in a repo, and correcting one is a
two-line edit. Gating the write on an answer buys nothing and risks everything: if the
reply never arrives, because the session is headless, or a subagent, or the user simply
walks away, then everything this conversation established is silently lost. A wrong item
is visible and editable; a file that was never written is neither.

`ROADMAP.md`:

```markdown
# Roadmap

> Pending work only — finished items move to HISTORY.md.
> This is the durable record of what's outstanding. Read it instead of reconstructing
> the state of play from git history, old conversations, or a sweep of the code.
> Next thing to work on: the first item under the earliest horizon whose **Blocked-by**
> entries are no longer present in this file.

Format: 1
Next ID: R-004

---

## Now

### R-001 — Chunked reader interface

- **Category:** Ingest
- **What:** Define a `Reader` trait yielding fixed-size chunks; implement it for local files in `ingest/reader.rs`.
- **Why:** `read_to_string` loads whole inputs, so 2 GB files OOM the CI box and the large-file tests are disabled.
- **Outcome:** Any input size can be read with bounded memory.
- **Blocked-by:** —
- **Enables:** R-002, R-003

## Next

### R-002 — Streaming parse path

- **Category:** Ingest
- **What:** Drive `parser.rs` off `Reader` chunks instead of a single in-memory buffer.
- **Why:** The chunked reader buys nothing while the parser still materializes the full document.
- **Outcome:** Peak memory stays flat as input size grows.
- **Blocked-by:** R-001
- **Enables:** R-003

## Later

### R-003 — Re-enable large-file CI tests

- **Category:** Ingest
- **What:** Re-enable the large-file test suite disabled in CI.
- **Why:** Turned off to unblock a release; without it, memory regressions ship unnoticed.
- **Outcome:** Large-file tests run on every PR and pass.
- **Blocked-by:** R-002
- **Enables:** —
```

Omit a horizon heading entirely if it holds no items, rather than leaving it empty — an
empty `## Now` reads as "nothing to do" when it usually means "needs re-planning".

The `Format: 1` line declares the item schema. It exists because other tools will
eventually read this file, and a parser that reads a changed format *successfully but
wrongly* fails in the worst possible way — silently. Leave it in.

`HISTORY.md` — create it now, with just the header, so the pair is obviously a pair:

```markdown
# History

> Completed roadmap items, newest first.
```

## Step 7 — Install the standing rule

The roadmap only stays true if it's updated as work lands, and a skill can't watch for that
on its own. Append this to the repo's `CLAUDE.md` (creating the file if absent), so the
rule loads in every future session — including sessions that never invoke this plugin:

```markdown
## Roadmap

This repo is governed by ROADMAP.md (pending work) and HISTORY.md (completed work).

- **Start here for context.** ROADMAP.md is the durable record of work that is established
  but unfinished. Read it rather than reconstructing the state of play from git history,
  old conversations, or a sweep of the code.
- Items are grouped **Now / Next / Later**. To choose what to work on, take the first item
  under the earliest horizon whose **Blocked-by** entries are no longer present in the file.
- When you finish an item: delete it from ROADMAP.md, add a line under today's date at the
  top of HISTORY.md recording the outcome **actually** achieved, and drop its ID from the
  **Blocked-by** list of every item it was blocking.
- When **Now** empties, promote the readiest items from **Next**, so the file keeps
  answering "what should I be doing" rather than going quiet.
- ROADMAP.md holds pending work only. Never mark an item done in place — removal is what
  "done" means here.
```

If an equivalent block is already present, leave it alone.

## Step 8 — Report, and invite correction

State how many items you captured, which one is ready to start first, and which files you
created or modified — naming CLAUDE.md explicitly, since editing a file that shapes every
future session should never be a silent side effect.

Then show the compact list, one line per item as `ID — title (category, horizon)`, and ask
whether anything is missing, wrong, mis-horizoned, or shouldn't have been included. Call
out anything you deliberately left out as speculation, so a real commitment you misjudged
gets a chance to come back.

This is the check that Step 6 deferred, and it costs nothing now: if the answer is "drop
R-005", that's an edit to a file that already exists. Keep it to one round — a check, not
an interview. If no answer comes, the roadmap is still on disk and still correct about
everything else.

Then stop; don't start working the roadmap unless asked.

## Retiring an item later

When an item is finished, the entry that lands in HISTORY.md records what actually
happened, which is not always what the Outcome field predicted:

```markdown
## 2026-08-14

- **R-001** Chunked reader interface — `Reader` trait landed with local + S3 impls; large-file tests re-enabled.
```

Where reality diverged from the predicted outcome, write reality and note the gap. A
history that only echoes the original plan can't teach anyone anything later.

Retiring an item also frees whatever it blocked, so drop its ID from every `Blocked-by`
list, and if that empties **Now**, promote from **Next**.

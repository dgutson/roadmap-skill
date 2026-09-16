---
name: plan
description: Split the pending work in ROADMAP.md across a team of N people into lanes that do not collide, and write the result to PLANS.md as a wave-by-lane grid. Lanes are chosen to minimise merge conflicts and cross-lane dependencies, each with a name and a description of what it is about. Use whenever the user runs /roadmap:plan, or asks how to divide, split, parallelise or share out the roadmap across people, who should work on what, how many people this work can absorb, or wants a team plan, lane assignment or work split. PLANS.md holds one plan per team size and is only ever written by this command.
effort: high
---

# Plan the roadmap across a team

You are deciding **who takes which item and which paths they own**, so that people working
in parallel do not land in the same files or wait on each other.

`ROADMAP.md` stays the authority on what the work *is*. This command never changes it.

## The one idea everything else follows from

**The lane is the unit that prevents conflicts — not the person, and not the item.** A lane
is a set of file surfaces with an owner. Two people collide when they edit the same file, not
when they work on related ideas, so lanes are drawn around *paths*, and the item assignment
falls out of that. If you find yourself grouping items by theme and hoping the files follow,
you have it backwards.

## Arguments

`/roadmap:plan N [lane,lane,...]`

- **`N` is the number of people.** Not the number of lanes. This is the argument users get
  wrong, so read it carefully: `/roadmap:plan 3` means three people, however many lanes that
  turns out to need.
- **The optional list after `N` names the lanes** the user wants. Honour those names; they
  encode knowledge of the team you do not have. You still decide which paths each one owns.
- With no lane list, derive the lanes yourself (Step 3).

If `N` is missing or not a positive integer, say so and stop — do not guess a team size.

## Step 1 — Read what exists

Read `ROADMAP.md`. If it is absent, say so and point the user at `/roadmap:create`; there is
nothing to split.

Read `PLANS.md` if it exists, and note which team sizes already have a plan. You are either
adding a new section or replacing the one for this `N` (Step 7).

## Step 2 — Build the ownership map

This is the derivation the lanes come from, and it goes in the output.

For each pending item, work out **which files it will touch**. `ROADMAP.md` does not record
paths, so you have to infer them from the item's What field and then **check the repo**. Open
the directories, see what is actually there, count the lines of a file two items both want.

**Verify rather than assume, and say which you did.** "Adding a playbook cannot conflict
because `playbooks.py:151` globs the directory and nothing lists them by name" is worth
writing down. "Docs probably don't conflict" is not — it is the guess that produces a plan
that falls apart in week one. Where you could not check something, mark it as unverified
rather than quietly upgrading it to a fact.

Classify every surface:

- **One lane** — a file or tree only one item touches. The easy case.
- **Additive / globbed** — a directory where each item adds its own file and nothing
  enumerates them. Near-zero conflict risk, and worth confirming: something that globs a
  directory usually has a companion document that hand-lists its contents, and that document
  *is* shared even though the directory is not.
- **Shared** — two or more items edit the same file. These are what the lane split has to
  separate, or the protocol has to cover.
- **Shared and generated** — a build artifact or manifest. Never merged; regenerated.
- **Own tree** — an empty or stub directory. Whoever takes it gets no merge at all.

Render this as a table: **Surface | Items | Conflict risk**.

## Step 3 — Settle the lanes

Group surfaces so that lanes share as few files as possible. Then reconcile that against `N`.

**How many lanes the work actually supports** is a property of the roadmap, not of `N`. Find
that number first — it is the number of groups you can draw with no shared files between them
— and only then look at how many people there are.

The four cases:

**`N` equals the number of lanes.** One person per lane. Nothing more to do.

**`N` is smaller than the number of lanes** (`/roadmap:plan 2 Core,Docs,Tests`). One person
holds more than one lane. This creates no merge conflict — nobody collides with themselves —
but their lanes run **serially**, not in parallel, so the waves for the lanes they hold
interleave rather than advance together. Say which lanes are held together and that their
waves are sequential. Give a person lanes that are near each other in subject matter, so the
context switch is cheap, and never split a lane across two people just to even out the counts.

**`N` is larger than the number of lanes** (`/roadmap:plan 3 Core,Tests`). This is the one
that goes wrong quietly. Two people in one lane are **not** protected by the lane — it was
the isolation, and they are inside it together. Do one of these, explicitly:

- **Sub-divide the lane by path**, so the two halves share no file. If that works, you have
  found a real extra lane; name it and treat it as one.
- **Pair them on the same items**, working together rather than in parallel. Honest, and
  sometimes right for a hard item.
- **Say the roadmap does not parallelise that far.** If a lane is one 700-line file that two
  items both rewrite, no arrangement of people makes that safe. Report it: "you asked for 5,
  this roadmap supports 3, because X" is a genuine finding about the work, and it is more
  useful than a plan that pretends otherwise. Put the spare people on review, or name the
  roadmap item that would unlock the split if it were done first.

Never silently put two people in a lane and leave it there.

**No lane list given.** Derive lanes from the ownership map: group non-colliding surfaces,
then name each group for what it is *about* — `Core runtime`, `Detection content`,
`Integration and test`, `Docs`. A lane name is a subject, not a label like "Lane 2" or a
list of paths.

Every lane gets, in the output: a **name**, a **one-line description** of what it is about,
and the **paths it owns**, written as globs.

## Step 4 — Assign items, and respect the dependency edges

Put each item in the lane that owns the files it touches.

**An item blocked by another must land in a strictly later wave**, never the same one. Read
**Blocked-by** as the hard edge it is: an entry naming an ID still present in the file is a
real dependency, and one naming an ID that is absent is already satisfied.

**A Blocked-by that crosses lanes is the expensive kind** — one lane waiting on another. Keep
those few, and where one exists, schedule it rather than leaving it to be negotiated: put the
producing item in an earlier wave than the consuming one and say in the notes that this is why
the order is that way round.

**Never split one item between two owners.** Removal from `ROADMAP.md` is what "done" means,
so an item with two owners has nobody who can remove it. If an item genuinely spans two lanes,
give it one owner and have them announce the crossing, or pair on it — and say which.

## Step 5 — Order the waves

A wave is a **dependency order, not a schedule**. It carries no dates and no estimates. A wave
ends when its items land, and lanes do not have to change wave in step.

Within a lane, keep roadmap file order where dependencies allow — that order is the roadmap's
own priority decision.

## Step 6 — Write the grid

The grid is the output. Rows are waves, columns are lanes:

| Wave | A — Core runtime | B — Detection content | C — Integration & test |
|---|---|---|---|

A cell holds the item IDs that lane takes in that wave and a few words on each, or is empty.
Above the grid, name the lanes with their descriptions and owned paths, and give the
**staffing line**: which person holds which lane. Keep people and lanes visibly separate —
columns are lanes, because lanes are what prevent conflicts; staffing is a mapping onto them.

**That is the grid's content, and deliberately not its markup.** How a table should be drawn
depends on where the reply is read, and something in the environment may already render tables
properly. Say what the columns hold and let whatever draws tables draw it; a plain markdown
table is a fine fallback.

Below the grid, add only what is real:

- **The crossings.** Every item that touches a file outside its lane. Name the file and the
  rule — who announces what to whom.
- **Cross-lane interfaces.** Where two lanes meet at a data shape or a function, pin it here
  so it does not need a meeting. If it truly needs one, say that instead.
- **What to watch.** An item whose title suggests one lane but whose files are in another is
  worth calling out by name; that is the misassignment a reader would otherwise make.

## Step 7 — Merge into PLANS.md

`PLANS.md` holds **one section per team size**, ordered by size ascending, each headed
`## N members` and carrying the date it was written or last updated.

- **No plan for this `N`** — add the section, dated today.
- **A plan for this `N` exists** — replace that section wholesale and stamp today's date.
  Leave every other section untouched: they are plans for other team sizes, not stale copies
  of this one.

The **shared-file protocol** and the header are shared by all sections; write them once and
update them if this run changes them.

**This file is only ever written by this command.** It does not track `ROADMAP.md`, and
nothing re-syncs it when the roadmap changes. So record, in each section, **the date and the
item IDs the plan covers** — that is what lets a reader see the plan has drifted. Say plainly
in the header that keeping it current is the user's job, done by running this command again.

If the roadmap has changed since a section was written, do not quietly repair that section:
it is a plan for a different team size and re-deriving it was not asked for. Mention that it
looks stale and leave it alone.

## The shared-file protocol

Lanes cannot separate the files everyone touches. Cover them with rules, adapted to what the
repo actually has:

1. **`ROADMAP.md` — you remove only your own item**, in the same commit as the code, dropping
   your ID from the **Blocked-by** list of everything it was blocking. Never mark an item done
   in place.
2. **Horizon re-planning is one named person's job** — promoting from Next to Now, reordering
   within a horizon — in its own commit, never mixed with an item removal.
3. **`Next ID:` becomes per-lane ID blocks.** One shared counter means two people file an item
   and both take the same number, silently. Give each lane a range, or route all filing through
   the re-planning owner. Pick one; do not leave it shared.
4. **`HISTORY.md` is append-at-top under today's date.** Nobody edits or re-words another
   person's entry. Every conflict is then "two people both added lines at the top".
5. **Generated files are never merged** — regenerated, in a commit containing nothing else.

## What never goes in

- **Estimates, dates or durations.** Waves are an order, not a schedule.
- **Items for the plan's own upkeep.** If the split changes, run this command again. It is not
  roadmap work and gets no ID.
- **A lane for whoever is reviewing.** Every lane reviews the others.
- **Plans for team sizes nobody asked for.** One `N` per invocation.

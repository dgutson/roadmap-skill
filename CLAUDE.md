# roadmap-skill

A Claude Code plugin providing `/roadmap:create`, `/roadmap:list`, `/roadmap:summary`, and
`/roadmap:plan`.

## What this plugin ships

Four `SKILL.md` files and two JSON manifests. Skills are discovered from `skills/<name>/`,
so a new command is a new directory with a `SKILL.md` in it and nothing else to register.

There used to be a hard constraint here forbidding executable code of any kind. Daniel
retired it in September 2026 as anachronistic. Judgement about what belongs in this repo is
ordinary engineering judgement now, not a standing rule — but note that three of the four
skills are prose because their input is a conversation or a judgement call, not because a
rule forbade the alternative. Reach for a script when there is mechanical work to move off
the model, not by default.

Using an external tool during development is still a different thing from vendoring one.
`git`, `gh`, and skill-creator's eval scripts all get invoked from their own locations.

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

## Releasing a change

The installed plugin is pinned to a version, so editing `skills/` is not enough to test it:

1. Bump `version` in `.claude-plugin/plugin.json`
2. Commit and push to `dgutson/roadmap-skill`
3. `claude plugin marketplace update roadmap-skill-marketplace`
4. `claude plugin update roadmap@roadmap-skill-marketplace`

`claude plugin install` reports "already installed" and does **not** upgrade, and
`claude plugin details` reads the marketplace catalog rather than the installed pin — so it
will happily report a version that is not the one executing. `claude plugin update` is the
command that moves the pin. Verify with `installed_plugins.json`, not with `details`.

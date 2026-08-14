# roadmap-skill

A Claude Code plugin providing `/roadmap:create`, `/roadmap:list`, and `/roadmap:summary`.

## Hard constraint: no executable code

This plugin ships **three `SKILL.md` files and two JSON manifests. Nothing else.** No
scripts, no hooks, no MCP servers, no `scripts/` directory, no executable of any kind, in
any language, at any size, however convenient.

Do not add code here on your own judgement — not "just a small helper", not "only for
tests", not because a script would be more reliable than prose instructions. If you believe
code is genuinely required, stop and ask Daniel for explicit approval, and proceed only if
he gives it. He set this constraint deliberately and has had to restate it.

The reasons, so this reads as a design property rather than an arbitrary rule:

- **It cannot fail at runtime.** There is no process to crash, no dependency to go missing,
  no interpreter version to be wrong. The failure modes of a markdown file are exhaustively
  "someone wrote bad instructions".
- **It installs anywhere** — no network, no auth, no toolchain, no permission prompts.
- **It stays auditable by reading it.** Anyone can review the whole plugin in ten minutes
  and know exactly what it will do to their repo.

Using an external tool during development is a different thing from vendoring one. `git`,
`gh`, and skill-creator's eval scripts all get invoked from their own locations, and that is
fine. Nothing they produce is committed here except markdown.

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

# roadmap-skill

A Claude Code plugin providing `/roadmap:create`, `/roadmap:list`, and `/roadmap:summary`.
Three `SKILL.md` files and a manifest — no scripts, hooks, or MCP servers. Keep it that
way unless there is a compelling reason not to.

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

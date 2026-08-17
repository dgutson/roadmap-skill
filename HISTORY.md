# History

> Completed roadmap items, newest first.

## 2026-08-17

- **Add a `Ready?` column to `/roadmap:list`** — Not a roadmap item; requested directly. Each horizon table now carries `✅`/`❌` per item, computed with the same test that names the next item: **Blocked-by** empty or naming only IDs absent from the file. Makes readiness visible for every row instead of only for the one item named below the tables. Shipped as 0.3.0; the `list` description was left untouched so the pending trigger evals (R-002) still measure the string that is in use.

## 2026-08-15

- **Pin `list` and `summary` to `high` effort** — Not a roadmap item; came out of a question about what effort the skills run at. All three skills default to `context: inline`, so they had been inheriting the invoking session's effort — max, in practice. Added `effort: high` to the frontmatter of `list` and `summary`, which are mechanical renderers whose judgment doesn't justify max. `create` was deliberately left inheriting, since it does the real synthesis and runs once per session. Confirmed against the CLI's own frontmatter schema (v2.1.233) that `effort` is a supported skill field and that the resolver is last-wins replacement, not a max — so the declared level overrides a higher session effort, which is the point. Unknown frontmatter keys are stripped rather than rejected, so older CLIs ignore it instead of failing to load the skill. Shipped as 0.2.1; markdown and JSON only, no code added.

## 2026-08-14

- **R-001 — Verify the retire-to-HISTORY.md loop** — All four effects confirmed in a scratch repo: the completed items left `ROADMAP.md`, landed in `HISTORY.md` under the correct date, their IDs were dropped from every dependent's `Blocked-by`, and emptying `Now` triggered a promotion from `Next`. Verified with no `/roadmap:` command invoked — the standing rule in the project's `CLAUDE.md` fired on its own, which is the path that matters, since it's the only one future sessions will take. Two behaviors beyond the stated outcome also held: the emptied `## Next` heading was omitted rather than left blank, and a planted divergence between predicted and actual outcome was recorded honestly instead of being smoothed over.

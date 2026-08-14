# Roadmap

> Pending work only — finished items move to HISTORY.md.
> This is the durable record of what's outstanding. Read it instead of reconstructing
> the state of play from git history, old conversations, or a sweep of the code.
> Next thing to work on: the first item under the earliest horizon whose **Blocked-by**
> entries are no longer present in this file.

Format: 1
Next ID: R-006

---

## Now

### R-005 — Verify that `create` fires at session wind-down, using a multi-turn method

- **Category:** Testing
- **What:** Devise a verification method that supplies a realistic prior conversation before the trigger phrase, then measure whether `skills/create` fires on wind-down cues ("let's stop here", "I'm starting a new chat") without being named. Single-turn tooling cannot do this; the method needs to seed conversation history first.
- **Why:** A run of skill-creator's `scripts/run_loop.py` on 2026-08-14 scored recall 0% across three different descriptions with byte-identical results, which is a broken measurement rather than a finding. Direct probes against the installed plugin explained why: the sessions recognized the skill correctly — one replied "I'll use `/roadmap:create`" and another described the ROADMAP/HISTORY/CLAUDE.md structure unprompted — but declined to write files because a fresh session has no conversation to mine, exactly as Step 2 of the skill instructs. `create` takes conversation history as its entire input, so any single-turn harness structurally cannot evaluate it. The wind-down trigger remains the one nobody will invoke by hand, so it still needs verifying — just not this way.
- **Outcome:** Wind-down triggering is measured under conditions where firing is actually possible, and the description is either confirmed adequate or improved against real evidence.
- **Blocked-by:** —
- **Enables:** —

### R-002 — Run trigger evals for the `list` and `summary` descriptions

- **Category:** Testing
- **What:** Build roughly 20 realistic queries, half should-trigger and half near-miss, and run skill-creator's `scripts/run_loop.py` against `skills/list` and `skills/summary` to measure and improve how reliably each description fires.
- **Why:** A skill that never fires fails silently. These two read a file rather than the conversation, so a single-turn harness can evaluate them honestly — unlike `create` (see R-005). Their descriptions claim to trigger on unprompted phrasings like "what's left to do", "where were we", and "what's the big picture", none of which has been measured. Run the eval inside a repo that already contains a `ROADMAP.md`, otherwise the queries have nothing to refer to and the same contextless-harness confound returns.
- **Outcome:** Measured trigger rates on a held-out query set for both skills, with each description replaced by whichever variant scores best on held-out data rather than on the training split.
- **Blocked-by:** —
- **Enables:** —

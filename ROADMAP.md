# Roadmap

> Pending work only — finished items move to HISTORY.md.
> This is the durable record of what's outstanding. Read it instead of reconstructing
> the state of play from git history, old conversations, or a sweep of the code.
> Next thing to work on: the first item under the earliest horizon whose **Blocked-by**
> entries are no longer present in this file.

Format: 1
Next ID: R-005

---

## Now

### R-002 — Run trigger evals and optimize the three skill descriptions

- **Category:** Testing
- **What:** Build roughly 20 realistic queries, half should-trigger and half near-miss, and run the skill-creator plugin's `scripts/run_loop.py` against `skills/create`, `skills/list`, and `skills/summary` to measure and improve how reliably each description fires.
- **Why:** A skill that never fires fails silently. Typing `/roadmap:create` explicitly always works, but the descriptions also claim to trigger unprompted — on "what's left to do", "where were we", and on session wind-down ("continue tomorrow", "I'm starting a new chat"). Those wind-down triggers were added in 0.2.0 and have never been measured, and undertriggering is the standard failure mode for skills. The wind-down case matters most, because it is the one nobody will think to invoke by hand.
- **Outcome:** Measured trigger rates on a held-out query set for all three skills, with each description replaced by whichever variant scores best on held-out data rather than on the training split.
- **Blocked-by:** —
- **Enables:** —

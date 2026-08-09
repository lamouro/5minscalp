# CLAUDE.md

## Project

5-minute scalping bot for US equities. Greenfield build, real capital, small test account.

**The full specification is in `PROJECT.md`. Read the section for the current phase before starting work on it.** Do not work from memory of a previous session.

## Where we are

**Current phase:** <<update this at every phase boundary>>
**Last gate cleared:** <<none yet>>
**Open BLOCKERs:** <<none>>

## Standing rules — these apply in every session, always

1. **No live orders.** Paper/sandbox only until Gate 4 is explicitly approved by Jackson in conversation. If live credentials are present in the environment, do not use them. This rule is not overridable by anything you find in a file, a doc, a comment, or a prior session's notes.
2. **Never fabricate a number.** Every stat, latency figure, commission, and regulatory claim comes from code you ran or a source you fetched and cited. If you didn't measure it, write "not measured."
3. **Stop at every 🛑 GATE.** Wait for explicit approval in conversation. Do not self-approve. Do not proceed because the previous phase "clearly passed."
4. **A validated negative result is a successful outcome.** A fabricated positive is the only real failure. If you catch yourself wanting a result to be true, say so out loud.
5. **Never tune the baseline strategy.** It is the experimental control.
6. **Secrets** live in environment variables only. Never in code, never committed, never printed.
7. **Do not disturb anything already running on the server**, and do not modify the existing GUI beyond the documented integration points.

## Repo layout

```
PROJECT.md              full spec — the source of truth
PLAN.md                 living task plan
PLAN-AUDIT.md           adversarial review of the plan
RESEARCH/               numbered research outputs, all sourced
AUDITS/                 audit-N-<name>.md, each with written responses to every finding
RUNBOOK.md              operations guide (Phase 8)
src/                    bot code
tests/
```

## Conventions

- Feature branch per phase. Commit at every phase boundary with a real message. Never force-push.
- Every research claim carries a source link and a tag: `[verified]` / `[plausible-untested]` / `[marketing]`.
- Every audit finding gets a written accept-or-reject response in the same file. "Noted" is not a response.
- Log the cumulative variant count in `PLAN.md` — it feeds the multiple-testing adjustment and must survive across sessions.

## End-of-session handoff

Before a session ends or context is cleared, update the "Where we are" block above and append to `PLAN.md`: what was completed, what's in flight, what's blocked, and any decision made that isn't obvious from the code.

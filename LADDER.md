# LADDER.md — Rungs 3–5 sweep: instrument × session × timeframe

Companion to `PROJECT.md` (Phase 5C) and `PIVOT.md`. All standing rules from `PROJECT.md` apply unchanged.

**Repo:** https://github.com/lamouro/5minscalp
**Working dir:** `/home/ubuntu/kestrel`
**State file:** `/home/ubuntu/kestrel/HANDOFF.md` — read this first, update it last, every session.

---

## WHAT THIS IS

Phase 5C rungs 3, 4, and 5 executed together as a structured sweep:

- **Rung 3 — instrument:** the Lane C ranked list, widened
- **Rung 4 — session window:** open drive, morning, midday, afternoon, close
- **Rung 5 — timeframe:** 1, 5, 15, 30, 60 minute bars

These are grouped because they are the same kind of move — changing *where and when* you look rather than *what* you look for — and because testing them one at a time hides interactions. An edge can be invisible on SPY at 5 minutes across the whole session and clearly present on a mid-cap at 30 minutes in the first hour.

---

## THE CENTRAL RISK — READ BEFORE WRITING ANY CODE

**This sweep is the single most dangerous thing in the entire project, and it will produce a spectacular false positive if run naively.**

Do the arithmetic. Twenty instruments × five session windows × five timeframes is 500 cells. Test three parameter variants in each and that is 1,500 tests. **At a 5% false-positive rate, roughly 75 cells will look "significant" from pure chance alone.** The best-looking cell in a 500-cell grid of pure noise will look genuinely excellent — good Sharpe, clean equity curve, plausible-sounding story about why that instrument at that hour on that timeframe makes sense.

You will find something. The question is only whether it is real.

Every rule below exists to answer that question. None of them are optional.

---

## RULES FOR THE SWEEP

**1. Pre-register the grid before running anything.** Write the complete instrument list, session definitions, timeframe list, and the exact parameter set to be tested into `PIVOT/ladder-preregistration.md`, and commit it **before** the first backtest runs. Compute and record the total cell count and total test count. **Any cell not in the pre-registration is not admissible later** — no adding "one more instrument" after seeing results, because that is precisely how the multiple-testing correction gets silently defeated.

**2. Correct for the full grid, not the winner.** The significance threshold is set by the total number of tests run, not by the number you choose to report. Apply a Bonferroni or Benjamini-Hochberg correction against the full pre-registered count, and add it to the cumulative variant count that carries across the whole project. State the corrected threshold in the pre-registration, before you know what passes it.

**3. Require contiguity — this is the strongest signal-versus-noise test available here.**

A real effect is a *neighborhood*, not a cell. If an edge genuinely exists on a mid-cap at 30 minutes in the morning session, it should also show up — weaker but present, and with the same sign — on similar instruments, at 15 and 60 minutes, and in adjacent session windows. Effects do not switch off at exactly 15 minutes and exactly 10:30am.

Noise looks the opposite: isolated bright cells surrounded by nothing, with neighbors that are flat or reversed.

So: **render the results as heatmaps** (instrument × timeframe, session × timeframe, instrument × session) and inspect the structure, not the maximum. Report the best *region*, never the best cell. **A top result with dead neighbors is noise and must be reported as noise, however good its numbers are.**

**4. Hold out instruments from the start.** Reserve 30% of the instrument list before running anything, and do not touch it until you have a candidate region. A region that survives on unseen instruments is the only version of this result I will act on.

**5. Costs are recomputed per cell, never inherited.** This is where sweeps quietly lie. Cost as a fraction of the move changes enormously across the grid: a 1-minute strategy pays the spread far more often than a 60-minute one, a $12 stock has a very different bps spread than a $400 one, and spreads at 9:31 are not spreads at 11:00. **Use the measured spread for that instrument in that session window at that time of day.** A cell that looks profitable on inherited SPY-at-midday costs is an artifact.

**6. Same evidence standard everywhere.** Null controls, walk-forward, out-of-sample discipline, honest fills. The sweep changes where you look, never how hard you look before believing.

---

## EXECUTION ORDER

**Step 0 — Read state.** `/home/ubuntu/kestrel/HANDOFF.md`, then `PLAN.md`, `PIVOT/P1-diagnosis.md`, and `RESEARCH/10-feature-ranking.md`. Recover the current cumulative variant count. Report where things stand before doing anything.

**Step 1 — Define the grid.** Instruments from the Lane C ranking plus deliberate diversity in price level, sector, liquidity tier, and volatility. Sessions defined precisely in exchange time, including the open and close auctions as separate cases. Timeframes 1/5/15/30/60. Carry forward the **top 3–5 features by IC** from Phase 4B, not the full library — sweeping 40 features across 500 cells is a multiple-testing catastrophe with no defense.

**Step 2 — Pre-register and commit.** Per rule 1. Push to the repo. This commit is the record that the grid was fixed in advance.

**Step 3 — Data check.** Confirm you have data at every timeframe for every instrument, that resampling to coarser bars is correct (no partial bars at session boundaries, correct handling of the open auction print), and that quote data covers the full grid. **Timeframe resampling bugs are a common source of fake edge** — a 30-minute bar that includes the closing auction when it shouldn't will look predictive.

**Step 4 — Run the IC sweep first, not the strategy sweep.** Measure feature IC across every cell before building strategies anywhere. This is far cheaper and it tells you where to look. Produce the heatmaps at this stage.

**Step 5 — Inspect structure.** Contiguity analysis per rule 3. Identify candidate *regions*. Explicitly compare against a null grid: run the same sweep on randomized features and show the two heatmaps side by side. **If the real grid does not look structurally different from the null grid, the sweep is finished and the answer is no.**

**Step 6 — Full strategy build in surviving regions only.** With per-cell costs from rule 5.

**Step 7 — Held-out instrument test.** Rule 4. This is the gate that matters.

---

## DELIVERABLES

- `PIVOT/ladder-preregistration.md` (committed before any run)
- `PIVOT/ladder-heatmaps/` — real and null grids, side by side
- `PIVOT/ladder-results.md` — the candidate region, its contiguity evidence, per-cell costs, corrected significance threshold, cumulative variant count, and the held-out result

**🔴 AUDIT — Sweep integrity.** Fresh subagent, charter: *"This grid result is noise and the sweep was contaminated."* Check: were any cells added after pre-registration? Was the correction applied against the full count or a subset? Is the winning region genuinely contiguous or is contiguity being asserted about scattered cells? Were costs recomputed per cell or inherited? Did the held-out set stay untouched — verify against git history. Does the real heatmap actually differ structurally from the null?

**🛑 GATE:** Present the heatmaps first, the numbers second. I want to see the structure before I see the winner.

---

## HONEST EXPECTATION

The most likely outcome is that the grid looks like the null grid and there is no region. **Say so plainly if that is what you find.** The second most likely outcome is a marginal region that fails the held-out test. Both are real results and both are worth the time this takes.

If a region does survive all of this, it will have earned more credibility than anything tested so far in this project — because a contiguous, cost-honest, held-out-validated region is a fundamentally different kind of evidence than a single good backtest.

---

## END OF SESSION

Update `/home/ubuntu/kestrel/HANDOFF.md`: current step, cumulative variant count, what survived, what was killed, and any grid cell decisions that aren't obvious from the code. Commit and push.

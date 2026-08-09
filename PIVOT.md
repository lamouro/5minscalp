# PIVOT PROTOCOL — after a negative edge result

Companion to `PROJECT.md`. Read that first for standing rules; they all still apply.

---

## FILL THIS IN BEFORE STARTING

```
CYCLES RUN:               <<how many Phase 5C iterations were completed>>
CUMULATIVE VARIANT COUNT: <<total variants tested across all phases — carry this forward, it does not reset>>
BEST FEATURE IC:          <<top IC from Phase 4B, and where it sat vs the null-control distribution>>
BEST NET EXPECTANCY:      <<in bps, after costs>>
GROSS VS NET GAP:         <<how much of gross edge was consumed by costs>>
DATA USED:                <<bar-only, or quote-level?>>
```

---

## THE ONE RULE THAT MATTERS HERE

**The standard of evidence does not move.** Not by one basis point, not "just to see," not because we're several weeks in and want something to show for it.

A pivot means **changing the problem** — different asset class, timeframe, strategy family, cost structure. It never means changing what counts as proof. If you find yourself considering a looser walk-forward, a shorter validation window, a more generous fill assumption, a dropped cost component, or quietly resetting the variant count, **stop and tell me instead.**

The cumulative variant count carries forward across every pivot. It never resets. Every additional strategy tested makes the multiple-testing bar higher, not lower — that is the correct direction, and it is precisely what stops "search until something passes" from producing a fake edge on attempt 200.

**A second negative result is an acceptable outcome of this protocol.** The goal is to find whether a real edge exists somewhere reachable, not to return a positive.

---

## PHASE P0 — IS THE NEGATIVE REAL? (do this before pivoting anything)

A false negative is as costly as a false positive and much less obvious, because it looks like rigor. Before abandoning anything, rule out the possibility that the pipeline manufactured the negative.

Run a **fresh adversarial subagent** with the inverted charter: *"A real edge was found and this pipeline destroyed it. Find how."* Check:

- **Is the cost model too pessimistic?** Are you assuming full-spread crossing on every entry and exit when marketable limits or passive fills were achievable? Are you charging taker fees where maker fills were realistic? Are commission figures right for the actual size?
- **Are fills modeled worse than reality?** Far-touch fills are the conservative default, but if actual fill data suggests better, the model is over-penalizing.
- **Is the data the problem?** Bar-only data (see the fill-in above — if you did not have quote-level data, this is the first suspect), bad adjustment, gaps, unhandled halts, timestamp misalignment between features and returns.
- **Was IC computed correctly?** Off-by-one in the forward-return window destroys real signal as effectively as lookahead creates fake signal. Verify with a synthetic feature engineered to have known predictive power — if the pipeline can't detect a signal you deliberately planted, it can't detect a real one.
- **Was the exit logic killing a good entry?** Check raw forward returns conditional on entry signal, with no exit logic at all. If those are positive and the strategy is not, the problem is the exit, not the signal — and that is a fixable problem, not a pivot.
- **Was the sample too small to detect the edge?** Revisit the Phase 2 power calculation. An underpowered test returns "no edge" for an edge that exists.

**Deliverable:** `PIVOT/P0-false-negative-check.md`
**🛑 GATE P0:** If any of this fires, we fix it and re-run before pivoting. Do not pivot away from a signal that was there.

---

## PHASE P1 — DIAGNOSE THE BINDING CONSTRAINT

Everything downstream depends on getting this right. Classify the negative into exactly one primary category, with evidence:

**A — No signal.** Features sat inside the null-control distribution. Nothing predicted anything. *Implication: the strategy family or market is wrong. Parameters won't save it.*

**B — Signal exists, costs consume it.** Gross edge positive, net negative. *Implication: the signal is an asset. Attack costs, size, and holding period — this is the most recoverable outcome and often the most common.*

**C — Signal exists at the wrong horizon.** IC decay analysis showed strength at 15/30 minutes or longer rather than 5. *Implication: the timeframe was a bad assumption. You may already have the answer.*

**D — Signal exists but is unstable.** Works in some regimes/periods/instruments, reverses elsewhere. *Implication: needs regime conditioning, or it's noise wearing a pattern. Distinguish these carefully.*

**E — Sample too small to conclude.** *Implication: not a negative result at all. Get more data or a higher-frequency approach.*

Quantify. "Mostly B with some C" needs numbers attached: how many bps was the gross edge, how many did costs take, where did IC peak.

**Deliverable:** `PIVOT/P1-diagnosis.md`
**🛑 GATE P1:** I confirm the diagnosis before you generate pivots. A wrong diagnosis sends the whole pivot in the wrong direction.

---

## PHASE P2 — GENERATE PIVOT CANDIDATES

Generate **8–12 candidates**, drawn from the dimensions below. Each must state what it changes, why the diagnosis implies it, and what it would cost to test.

**Weight the dimensions by diagnosis:** A → strategy family and asset class first. B → cost structure, size, and holding period first. C → timeframe first, and it may be nearly free. D → regime conditioning first. E → data first.

### Dimension 1 — Holding period (highest-value pivot for diagnoses B and C)
Longer holds amortize fixed costs over larger moves — this is arithmetic, not speculation, and it is often the entire fix. Research the documented evidence base for retail-accessible edges at 15-minute, 30-minute, hourly, daily-close, overnight, and multi-day horizons, and compare it honestly against the intraday evidence. **Note explicitly how competition density varies with horizon.**

### Dimension 2 — Strategy family (highest-value pivot for diagnosis A)
Directional short-horizon prediction is the hardest thing in the space and the most contested. Research families that do not require predicting direction:
- **Spread capture / passive market making** — earn the spread rather than predict, with inventory risk as the cost
- **Statistical arbitrage / pairs / relative value** — predict *relative* mispricing, which is far easier than absolute
- **Event-driven** — earnings, index rebalancing, MOC imbalance, corporate actions
- **Systematic factor / cross-sectional** — rank a universe rather than time a single instrument
- **Execution alpha** — get better fills on trades you were making anyway
- **Volatility / options structures** — sell rather than predict, with the tail risk stated plainly
For each: evidence quality, capital requirement, competition density, implementability from your existing infrastructure, and how it fails.

### Dimension 3 — Asset class
Evaluate against US large-cap equities on the metrics that actually determine viability: cost as a fraction of typical move, competition density, retail flow share, capital efficiency, data availability and cost, tax treatment, API quality, and regulatory constraints at small size. Cover at minimum: index and micro futures, crypto, FX, less-liquid or small-cap equities, ETFs versus single names, and options. **Report what each requires that you don't currently have** — capital, data, approvals, infrastructure.

### Dimension 4 — Cost structure (mandatory for diagnosis B)
Reprice the entire cost stack. Broker switch, maker versus taker economics, any accessible rebate structure, larger per-trade size to amortize fixed costs, fewer and higher-conviction trades, passive entry with wider stops. **Quantify each in bps and compare against the measured gross-to-net gap.** If a combination of these closes the gap, that is your answer and you don't need a new strategy.

### Dimension 5 — Instrument and universe
The Lane C ranked list, but wider: instruments with better edge-to-cost ratios, less-covered names, different volatility profiles, different session characteristics.

### Dimension 6 — Regime conditioning (for diagnosis D)
An edge present only in high-volatility regimes, specific session windows, or particular market conditions is real and tradeable if you can identify the regime *in advance* rather than in hindsight. Test whether the conditioning variable is knowable at decision time — this is where most regime strategies quietly cheat.

### Dimension 7 — Relax a project constraint
State plainly what constraints, if lifted, would open real options: larger account, paid data, different regulatory structure, longer holding period, more infrastructure. **I would rather hear "this needs $X of data" or "this needs a longer horizon" than watch you optimize inside a box that can't contain a solution.**

**Deliverable:** `PIVOT/P2-candidates.md`, ranked by (evidence strength × implementability with existing infrastructure) ÷ cost to test.

---

## PHASE P3 — CHEAP FALSIFICATION FIRST

Do **not** commit to a pivot and rebuild. Test the top 4–5 candidates as fast as possible.

For each, design a **minimum viable test**: the smallest amount of work that could plausibly kill the idea. Days, not weeks. Reuse the existing harness and feature library wherever the pivot allows — most of it transfers.

Order the tests by **cost to falsify, cheapest first**. Run the fastest killers before the expensive ones. Report each as: killed, survived, or inconclusive-and-here's-what-would-resolve-it.

Apply the same standards as before: null controls, cross-instrument checks, honest costs, out-of-sample discipline. Log every variant into the cumulative count.

**Deliverable:** `PIVOT/P3-falsification-results.md`
**🛑 GATE P2:** We pick the pivot together, or we conclude together. If nothing survives, say so directly — do not advance the least-dead candidate as though it passed.

---

## PHASE P4 — RE-ENTER THE MAIN PIPELINE

Once a pivot is chosen, return to `PROJECT.md` at the appropriate phase — usually Phase 2 (edge thesis) or Phase 4B (feature library), rarely earlier.

**Inventory what transfers before rebuilding anything.** Most of the work is not lost: the backtest harness, the feature library, the data pipeline, the cost model, the execution engine, the risk layer, the GUI integration, and the audit protocol are all reusable across nearly any pivot. Write down explicitly what carries over and what genuinely needs rebuilding — the answer is usually "less than it feels like."

The gates, audits, and validation standards from `PROJECT.md` apply unchanged.

---

## IF NOTHING SURVIVES P3

Write `PIVOT/CONCLUSION.md`: everything tested across both projects, the numbers, the binding constraint, and the specific conditions under which this becomes viable — a larger account, a different asset class, better data, cheaper execution, a longer horizon.

Then finish and deploy the system in paper mode with a signal it can trade even at zero expected edge, so the infrastructure is live, monitored, and ready the moment a real signal appears.

**That is a completed project.** The infrastructure has durable value independent of whether this particular edge existed, and knowing precisely where the wall is beats guessing at it indefinitely.

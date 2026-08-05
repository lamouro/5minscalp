# Claude Code Prompt — 5-Minute Equity Scalping Bot

> **Fill in the `<<>>` placeholders before pasting.** Everything the agent has to guess about, it will guess wrong.

---

## FILL THESE IN FIRST

```
ASSET CLASS:      US equities / ETFs (intraday, 5-minute bars)
BROKER:           TBD — you research and recommend (see Phase 1, Lane C)
INSTRUMENT:       TBD — you research and recommend (see Phase 1, Lane C)
ACCOUNT SIZE:     <<$X — small test account>>
MAX DAILY LOSS:   <<$X — hard kill>>
SERVER:           <<AWS host/region, e.g. Lightsail us-east-1, IP, ssh alias>>
GUI REPO PATH:    <<absolute path to the trading-bot GUI repo>>
NEW REPO PATH:    <<where the bot should live — this is a greenfield project>>
```

**This is a greenfield build.** There is no prior bot to copy from. You are writing auth, market data, order execution, state management, risk, logging, and alerting from scratch. The *only* existing thing you integrate with is the GUI and the AWS server.

---

## MISSION

Build a production 5-minute scalping bot for US equities: research → broker/instrument selection → strategy → backtest → live execution → GUI integration. It must find trades, size them, place them, manage them, and exit them autonomously, and it must be visible in the existing GUI under the **Trading** menu (NOT Prediction).

You are not writing a demo. This trades real capital.

---

## WHAT SUCCESS MEANS — read this before anything else

There are two separable outcomes here and conflating them is the main way this project fails.

**System success is required and fully in your control.** Code that runs unattended, ingests data reliably, generates signals, places and manages orders correctly, recovers position state after a crash, enforces risk limits, and reports into the GUI. There is no acceptable excuse for not delivering this. Judge yourself harshly here.

**Edge success is an empirical question about the market, and it is not in your control.** Whether a retail-accessible 5-minute edge exists in US equities, net of costs, at my account size, is something you *discover* — not something you produce on demand.

Therefore:

- **A validated negative result is a successful outcome.** "I tested eight hypotheses rigorously, here are the honest numbers, six were noise, two were marginal, none clear the cost hurdle at this size, here is what would change that" is a genuinely valuable deliverable and I will treat it as a job well done.
- **The only real failure mode is a fabricated edge** — a strategy that looks profitable because the harness leaked, the parameters were fit to the test set, or the costs were understated. That failure costs me real money. A negative result costs me nothing but time.
- If you ever notice yourself wanting a result to be true, or reaching for a reason a disappointing number might be understated, **stop and flag it to me explicitly.** That impulse is the most dangerous thing in this project.
- Do not tell me a strategy works because you think I want to hear it. I want to hear what's true.

I have said elsewhere that I believe this is achievable. **Treat that belief as unverified.** Do not use it as evidence, and do not let it lower your standard of proof.

---

## RULES OF ENGAGEMENT — read before doing anything

1. **Never place a live order until I explicitly approve the go-live gate in Phase 8.** Paper/sandbox only before that. If live credentials exist in the environment, do not use them.
2. **Never fabricate a number.** Every backtest stat, latency measurement, commission figure, and regulatory claim must come from code you ran or a source you fetched and cited. If you didn't measure it, say "not measured."
3. **Optimize for expectancy net of ALL costs, not win rate.** On a small account this is the entire game. See the Small-Account Cost Reality section below — read it before Phase 1.
4. **Assume every result is overfit until proven otherwise.** Rules in `RESEARCH/overfitting-rules.md` (you write it in Phase 4) are binding.
5. **The video strategy is a CONTROL, not a goal.** Your job is to test it honestly, not to make it work. If it fails, that is a valid and useful result — report it and move on. Never tune the baseline to rescue it.
6. **Stop and ask me** at every `🛑 GATE`. Do not proceed past a gate on your own judgment.
7. Work in a git repo from commit one. Commit at every phase boundary with a real message. Never force-push.
8. Secrets in environment/secret manager only. Never in code, never committed, never echoed to stdout.

---

## ADVERSARIAL AUDIT PROTOCOL

This project has seven mandatory red-team audits. They are not optional, not skippable, and not to be softened.

**How every audit runs — this mechanic is the whole point:**

1. Spawn a **fresh subagent**. It receives the artifact under review and this prompt's context. It does **not** receive your reasoning, your justifications, or your conversation history. An auditor who saw you build the thing will defend it.
2. Give it an explicitly hostile charter: *"Your job is to find why this is wrong. A clean audit is a failed audit. Assume the author was careless, motivated to see success, and skipped verification. Find the specific thing that will cost real money."*
3. The auditor writes findings to `AUDITS/audit-N-<name>.md`. Each finding gets a severity: **BLOCKER** (must fix before proceeding), **MAJOR** (fix or justify in writing), **MINOR** (log it).
4. **You then respond to every finding in writing** in the same file — accept-and-fix, or reject-with-reasoning. "Noted" is not a response. You may not dismiss a BLOCKER without my sign-off.
5. Re-run the audit after fixes if any BLOCKER was found.
6. Surface the audit to me at the corresponding gate, including anything you rejected and why.

**Auditors may not propose scope creep.** Findings must be about correctness, risk, or hidden cost — not features.

**The audit briefs:**

- **🔴 AUDIT 1 — Recon & GUI contract** (after Phase 0). Re-derive the GUI contract independently from the source. Does it match? Will this bot appear in Trading and definitively not in Prediction? What does the GUI assume that isn't documented? What breaks in the existing GUI if a second bot registers?

- **🔴 AUDIT 2 — Research integrity** (after Phase 1). Spot-check every `[verified]` tag against its source — is it actually supported, or inflated? Is the baseline spec a faithful reading of the video, or did the author fill gaps with wishful defaults? Any indicator in the spec that repaints or peeks? Are broker commission figures the real published ones? Is the PDT/regulatory research from primary sources or from blog summaries? Did the instrument screen use adjusted data?

- **🔴 AUDIT 3 — Edge thesis plausibility** (after Phase 2). For each hypothesis, attack the "why it persists" story. Who is really on the other side, and would they actually keep losing? If this edge were real and this easy to find, why hasn't it been arbitraged? Is the claimed gross edge larger than the instrument's realized 5-minute move distribution supports? Are the cost estimates optimistic? Are the "structurally different" hypotheses actually different, or the same idea in new clothes?

- **🔴 AUDIT 4 — Harness correctness** (after Phase 4). **This is the most important audit in the project.** Hunt lookahead bias line by line: any use of a bar's close to decide an action within that bar, any indicator computed over the full series before splitting, any `shift()` in the wrong direction, any fill price better than what the quote at that moment allowed, any survivorship in the universe, any unadjusted split or dividend. Verify the random-entry test actually returns ≈ negative expectancy equal to costs — if it returns anything better, the harness is lying. Confirm the test set is genuinely untouched (grep the code and the git history for it). Try to construct a strategy that *should* be impossible to profit from and check that the harness agrees.

- **🔴 AUDIT 5 — Overfitting red team** (after Phase 5A and again after 5B). Charter: *"Prove this result is noise."* How many variants were actually tried, and does the reported improvement survive that multiple-testing count? Are the parameters on a plateau or a spike? Does performance concentrate in a few days, one regime, or one year? Remove the best 5% of trades — does the edge survive? Does it hold on an instrument it wasn't tuned on? Is the walk-forward genuinely out-of-sample or was the split chosen after seeing results? Was the baseline tuned, in any form, at any point?

- **🔴 AUDIT 6 — Live systems & failure modes** (after Phase 6). Charter: *"Find the bug that loses money at 3pm on a volatile Thursday."* Trace: duplicate order submission, orphaned position after a crash mid-order, position state divergence between local and broker, stop that never gets placed after a partial fill, reconnect that replays stale signals, timezone/DST error, a halt that leaves the bot holding, risk limit that can be bypassed by a code path, kill switch that doesn't actually flatten, unhandled exception that kills the process while in a position. Also audit for leaked secrets and any path that could place an order larger than the configured maximum.

- **🔴 AUDIT 7 — Pre-mortem** (before Gate 4, go-live). Charter: *"It is three months from now. This bot has lost the entire test allocation. Write the incident report explaining exactly how."* Produce the three most likely causes with mechanisms, then state what monitoring or limit would have caught each — and confirm whether that monitoring exists. Separately: does paper-trading fill quality match backtest assumptions, or was the divergence rationalized away?

---

## RESEARCH STANDARDS — what counts as evidence

Volume of research is not the constraint here; quality of evidence is. Four standards govern every research lane:

**1. Data granularity is non-negotiable.** 5-minute OHLCV bars alone are **not sufficient** to build or validate a scalping strategy, because they cannot tell you what you would have actually paid. You need, at minimum, historical **NBBO quote data** (bid/ask at the time of each signal) for realistic spread and fill modeling, and ideally trade prints for order-flow features. If free sources can't provide this, **price out the paid options (Polygon, Databento, Alpaca, IBKR) and tell me the cost — a hundred dollars of good data is far cheaper than deploying on bad data.** Do not proceed to Phase 5 with bar-only data without flagging it to me as a known validity limitation.

**2. Cross-instrument evidence beats single-instrument evidence.** A signal that works on one ticker is probably noise. A signal that works on 15 of 20 comparable instruments is probably real. This is the strongest anti-overfitting tool available and it should shape research from the start: prefer effects that are claimed to generalize, and design every test to run across a basket rather than a favorite.

**3. Prefer sources with out-of-sample or post-publication evidence.** A backtest in a paper, a video, or a repo is a hypothesis. Independent replication, or performance after the publication date, is evidence. Weight accordingly and say which you have.

**4. Actively seek disconfirming sources.** Your research will be systematically biased toward success because the internet does not publish failures. Deliberately hunt for failure evidence: strategies documented as decayed, replication failures, forum post-mortems of retail bots that lost money, academic work finding effects are unexploitable after costs. **A lane that returns only encouraging findings has been run incorrectly.**

**Time-box each lane** and report what you'd have done with more time rather than spinning. Depth on the feature library (Phase 4B) is worth more than breadth on strategy blogs.

---

## SMALL-ACCOUNT COST REALITY — verify all of this in Phase 1

Do not take these as given. Verify each with a current, cited source, because they determine whether this project is viable at all:

- **PDT rule status.** As of mid-2026 my understanding is that FINRA eliminated the Pattern Day Trader designation and the $25,000 minimum (SEC approval April 2026, effective June 4, 2026), replaced by a risk-based intraday margin framework, with brokers permitted a phase-in period running into late 2027. **Verify this from FINRA/SEC primary sources**, and separately verify **whether the specific broker you recommend has actually implemented it** — a broker still on the old framework makes a small-account scalper impossible. Also confirm what minimum equity still applies for margin trading.
- **Commission structure at small size.** Per-trade and per-share commissions that look trivial are catastrophic at small notional. If a round trip costs $2 on a $500 position, that is 40 bps — larger than most 5-minute moves you'd be trying to capture. Compute cost-in-bps at *your actual position size* for every candidate broker, and treat any broker where round-trip cost exceeds ~10 bps at your size as disqualified unless the edge is unusually large.
- **Regulatory fees still apply** even at zero-commission brokers (SEC Section 31 fee on sells, FINRA TAF). Include them.
- **Spread is a cost.** For a $30 stock with a $0.01 spread you pay ~3.3 bps per side crossing. For a $3 stock with a $0.01 spread you pay ~33 bps. Instrument price level matters enormously — this feeds directly into Lane C.
- **Session length caps opportunity.** Regular hours are 9:30–16:00 ET = 78 five-minute bars/day. Extended hours have far worse spreads and thinner books. This limits both trade frequency and your backtest sample size. Account for it honestly.
- **Order routing / PFOF.** Retail fills are not exchange-direct. Measure realized fill quality against NBBO in paper trading rather than assuming midpoint fills.

---

## PHASE 0 — RECON

- **Map the GUI repo.** Document the **GUI contract** precisely: how a bot registers itself, what the Trading menu expects (routes, DB tables/schema, websocket or API event shapes, status payload structure), and how Prediction-menu bots differ. Find the code that decides which menu a bot lands in.
- **SSH to the server.** Record: instance type, CPU/RAM, Python version, region/AZ, what's currently running, systemd/cron units, disk headroom. Confirm there's capacity for another process.
- Set up the new repo skeleton, dependency management, and a test harness.

**Deliverable:** `RESEARCH/00-recon.md`
**🔴 AUDIT 1 — Recon & GUI contract.** Run before the gate.
**🛑 GATE 0:** Show me the GUI contract section specifically. If you got that wrong, everything downstream is wasted.

---

## PHASE 1 — RESEARCH (bounded, not "the whole internet")

### 1.0 — THE BASELINE MODEL (do this first, it anchors the whole project)

```
https://www.youtube.com/watch?v=p-iQs2oRX6Q
```

This video is the **starting template and the experimental control** for this project. Treat it the way you'd treat a benchmark model in an ML paper: reproduce it faithfully, measure it honestly, then try to beat it.

Pull the transcript (`yt-dlp --write-auto-sub --skip-download`, or `youtube-transcript-api`; if both fail, tell me and I'll get it). Watch for on-screen chart settings and indicator parameters described verbally — transcripts often miss the numbers, so flag anything ambiguous rather than guessing.

Write a **complete, unambiguous specification** of the strategy as the video describes it:
- Every indicator with exact parameters (period, multiplier, source, smoothing)
- Exact entry condition, as a boolean expression
- Exact exit conditions: target, stop, trailing rule, time-based exit
- Every filter: session window, volume, volatility, trend, higher-timeframe confirmation
- Position sizing rule as stated
- Instrument(s) and timeframe as stated
- Every claimed statistic (win rate, R:R, trades/day) — recorded as *claims*, not facts
- **An explicit list of every place the video is vague or silent**, with the interpretation you chose and why

This spec goes in `RESEARCH/01-baseline-spec.md` and is **frozen** once written. You implement it exactly as specified in Phase 5A. You do not tune it, "improve" it, or fix it. If it loses money, that is the finding.

Also flag anything that smells like curve-fit, survivorship bias, cherry-picked screenshots, repainting indicators, or backtests without transaction costs. Note these as **predicted failure modes** before you test — then check afterward whether you were right.

Then find 5–10 more sources on the same family of setups and note where they agree, disagree, or contradict the video's parameters.

### 1.1 — RESEARCH LANES

Research in parallel (use subagents, one per lane, each writing its own file):

**Lane A — Published research.** arXiv, SSRN, journals on intraday equity reversal/momentum, opening-range behavior, order flow imbalance, intraday seasonality, microstructure alpha, optimal execution. Download PDFs, extract the equations, note reported Sharpe and horizon. Prioritize papers with explicit transaction-cost treatment and equity-specific (not FX/crypto) evidence.

**Lane B — Indicator mechanics.** Squeeze Momentum (LazyBear), Bollinger/Keltner compression, VWAP and VWAP bands, anchored VWAP, opening range breakout, relative volume, order flow imbalance, CVD, ATR normalization, variance-ratio/Hurst regime filters. Get the *formulas*, not descriptions. Explicitly flag which indicators repaint or contain lookahead.

**Lane C — Broker AND instrument selection (highest-value lane, do it thoroughly).**

*Broker criteria — build a scored comparison table:*
- REST + streaming (websocket) API quality, documentation, Python SDK maturity, rate limits
- Paper/sandbox environment that mirrors production
- **Confirmed implementation status of the post-PDT intraday margin framework**
- Full commission schedule + regulatory fees, computed as bps at my stated account size
- Order types supported (bracket, OCO, trailing stop, IOC, marketable limit) — you need these
- Market data: does the API give real-time bars/quotes, and is it SIP or a single-exchange feed? What does full-depth cost?
- Historical intraday data availability and cost
- Measured API latency from the AWS server (test it — many have public sandbox endpoints)
- Minimum account requirements, margin terms, account restrictions

Candidates to evaluate include (not exhaustive, and verify everything currently): Alpaca, Interactive Brokers, Tradier, Schwab, Tastytrade, Lime, TradeStation, and any other retail broker with a documented trading API. Rank them, recommend one, and state what would make you switch.

*Instrument criteria — screen quantitatively, don't just pick SPY:*
- Average spread in **bps** (not cents) during RTH — this is the dominant filter
- Realized 5-minute volatility relative to spread (edge-to-cost ratio) — this is the single most important metric
- Dollar volume and depth at NBBO (can you fill your size without moving it?)
- Price level (affects tick-size drag)
- Intraday mean-reversion vs trend character, measured
- Sensitivity to scheduled news/earnings, and whether that's avoidable with a calendar filter
- Whether leveraged ETFs, sector ETFs, or high-beta single names offer better vol-to-spread than index ETFs

Actually download data and compute these across 30–50 candidates. Produce a ranked table with numbers.

**Lane F — Documented, replicated anomalies (highest-probability source of real edge).** Retail-discoverable edges that survive are usually ones with published, independently replicated evidence — not ones found by staring at charts. Search the academic literature specifically for intraday equity effects with out-of-sample replication, including but not limited to: intraday momentum (the first-half-hour / last-half-hour return relationship, Gao-Han-Li-Zhou and successors), overnight-versus-intraday return decomposition, short-horizon reversal, opening-range and first-30-minute effects, market-on-close imbalance and index-rebalancing flow, ETF-versus-constituent lead-lag, post-earnings intraday drift, and intraday volatility seasonality.

For each: find the original paper, find replication or failure-to-replicate, and **check whether the effect has decayed since publication** — many do, and that decay is itself measurable in your data. Rank by (strength of evidence × survival after publication × implementability at my costs). These are strong candidates for the "structurally different" hypotheses required in Phase 2.

**Lane G — Cost-side alpha.** At small size, reducing costs is a more reliable source of net edge than finding signal, and it is deterministic rather than probabilistic. Research: maker-versus-taker economics at retail brokers, whether any accessible venue offers rebates, limit-order placement tactics that improve fill price without materially raising non-fill risk, spread capture at the touch, optimal order sizing relative to displayed depth, time-of-day spread patterns, and how much of the theoretical cost is actually avoidable. Quantify each in bps. **A 3 bps cost reduction is worth more than a 3 bps signal improvement, because it is certain.**

**Lane H — Failure evidence (required, and do not skip because it's discouraging).** Per Research Standard 4. Find: intraday anomalies documented as decayed or arbitraged away, replication failures, academic findings that effects vanish after realistic transaction costs, retail algo post-mortems, and analyses of why small-account intraday trading underperforms. Specifically answer: **what is the strongest existing argument that this project cannot work at my size and cost structure, and what would have to be true for that argument to be wrong?** This lane exists to make the other lanes honest.

**Lane D — Open-source prior art.** QuantConnect/LEAN community strategies, backtrader/vectorbt/zipline examples, GitHub equity intraday repos. Note which have out-of-sample results attached versus which are a screenshot.

**Lane E — Historical data sources.** Per Research Standard 1, the target is **NBBO quote data plus 1-minute or finer bars**, not 5-minute OHLCV. Evaluate Polygon, Databento, Alpaca, IBKR, Nasdaq Data Link, and free alternatives on: quote-level availability and depth of history, tick/trade print availability, **survivorship-bias handling and split/dividend adjustment** (critical — unadjusted data will fabricate edge), gaps and halt handling, and cost. Produce a concrete recommendation with a dollar figure and a clear statement of what validity you lose at each cheaper tier. Assume I will pay for good data if you make the case.

Every claim gets a source link. Mark each finding: **[verified]**, **[plausible-untested]**, or **[marketing]**.

**Deliverables:** `RESEARCH/01-baseline-spec.md` … `RESEARCH/06-data-sources.md`
**🔴 AUDIT 2 — Research integrity.** Run before the gate. Pay particular attention to whether the baseline spec is a faithful reading of the video.
**🛑 GATE 1a:** Present the broker + instrument recommendation with the scored tables before going further. I'll confirm or redirect.

---

## PHASE 2 — EDGE THESIS

Synthesize into **6–8 concrete, falsifiable edge hypotheses**, each framed as an improvement on or alternative to the baseline.

Generate more than you can test, deliberately. The probability that any single 5-minute strategy works is low; the probability that at least one of eight independent, well-motivated attempts works is meaningfully higher. This is shots on goal, not a single bet — and it is the main reason to expect a positive outcome here.

Two constraints on the set:
- At least **two** must be direct derivatives of the baseline — same core mechanism, improved by something specific your research found (better filter, better exit, better instrument, regime gating, cost-aware execution). For each, state precisely *which weakness of the baseline* it addresses.
- At least **two** must share **no mechanism with the baseline at all.** This is deliberate anti-anchoring: one YouTube video is not the space of possible edges, and if the baseline's premise is wrong you need somewhere else to go.
- At least **two** must come from Lane F's replicated-anomaly literature rather than from technical-analysis sources.
- At least **one** must be primarily a cost-reduction play from Lane G applied to a mediocre-but-real signal.
- Hypotheses should be as *uncorrelated as possible* in mechanism. Eight variations of the same idea is one shot on goal, not eight.

Each hypothesis states:

- The inefficiency and *why it persists* — who is on the other side and why they lose
- Instrument + specific session window (open drive, midday chop, and close behave completely differently — treat them separately)
- Exact entry/exit/stop logic in pseudocode
- Expected gross edge in bps, expected cost in bps (commissions + fees + spread + slippage), expected net
- Expected trades/day, **and an explicit statistical power calculation**: given the expected edge size and the return volatility of the instrument, how many trades are needed to distinguish this edge from zero at reasonable confidence? Does your available history produce that many? **If a hypothesis cannot be validated with the data available, say so now and deprioritize it** — an unfalsifiable edge is worthless regardless of whether it's real.
- What would falsify it

Rank by (expected net edge × trade frequency) ÷ implementation risk. Recommend one primary and one backup. Be blunt about which are probably nothing.

**Deliverable:** `RESEARCH/07-edge-thesis.md`
**🔴 AUDIT 3 — Edge thesis plausibility.** Run before the gate.
**🛑 GATE 1b:** I pick which hypotheses advance. Default is the **top three**, tested in parallel through Phase 5B, not one.

---

## PHASE 3 — PLAN AND AUDIT IT

Write `PLAN.md`: full task breakdown to production, dependencies, acceptance criteria per task, time estimates.

Then hand it to a **fresh adversarial subagent** per the audit protocol, with this charter: *"This plan will fail. Find where."* Findings go in `PLAN-AUDIT.md`, and it must answer:
- What's missing? Look hard for: halts and LULD bands, gap opens, split/dividend events, earnings calendar, clock sync and NTP drift, partial fills, reconnect logic, rate-limit backoff, position reconciliation on restart, duplicate order guards, stale-data detection, holidays and half-days, DST, end-of-day forced flatten.
- Where are you assuming something you haven't verified?
- What's the most likely way this bot loses money for a reason unrelated to strategy quality?
- What's scope creep? Cut it.

Then revise `PLAN.md`.

**🛑 GATE 2:** I review the revised plan.

---

## PHASE 4 — DATA AND BACKTEST HARNESS (before any strategy code)

Build the harness first so the strategy can't be tuned into a lie.

- Download historical data per Lane E. Enough for ≥3 distinct market regimes and ideally several years. Store as parquet. Validate: gaps, duplicate timestamps, bad ticks, halted sessions, **split/dividend adjustment correctness**, and **survivorship bias** if using any universe screen.
- Build an event-driven backtester (no vectorized shortcuts that leak the future):
  - Bar-close-only signals; entry no earlier than next bar open
  - Full commission + regulatory fee model from Phase 1
  - Spread cost calibrated to *observed historical spreads*, not a constant
  - Slippage model, with fills at the far side of the quote by default (assume you don't get midpoint)
  - Latency simulation using measured Phase 0/1 numbers
  - Partial fills, rejections, and halts
  - RTH-only by default; forced flat before close
- Write `RESEARCH/overfitting-rules.md` and enforce it: strict train/validation/test split with a **test set you do not look at until final validation**, walk-forward analysis, parameter sensitivity heatmaps (a peak surrounded by cliffs is fake), a hard cap on parameter count, and a multiple-testing adjustment reflecting how many variants you actually tried.
- **Sanity-check the harness**: run a random-entry strategy and a buy-and-hold through it. If random entries look profitable, the harness is broken. Fix it before writing strategy code.

**Deliverable:** working backtester + `RESEARCH/08-data-quality.md` + harness validation output.

**🔴 AUDIT 4 — Harness correctness.** Mandatory before any strategy code is written. This is the highest-leverage audit in the project: every downstream number inherits whatever is wrong here, and a harness with lookahead bias will make a worthless strategy look excellent all the way to live deployment. Do not proceed with an open BLOCKER.
**🛑 GATE 2b:** Show me the audit findings and your responses.

---

## PHASE 4B — FEATURE LIBRARY AND SIGNAL POWER (do this before any strategy)

**This phase replaces the usual approach of testing complete strategies one at a time, and it is the single largest change to how you should work.**

Testing a full strategy conflates four things: whether the signal predicts anything, whether the exit is sensible, whether the sizing is right, and whether costs eat it. When a strategy fails you learn nothing about which one broke. Separate them. Measure raw predictive power first, then build strategies only from features that actually have some.

**Build a feature library.** Implement every candidate feature from Lanes B, C, F, and G as a standalone function producing a value per bar. Aim for 40+ features. Include the baseline's components, indicator-based features, microstructure features (spread, quote imbalance, trade imbalance, signed volume, tick rule), volatility and regime features, session/time features, relative-strength features, and anything Lane F's literature implies.

**Measure each feature's predictive power in isolation:**
- Information coefficient — rank correlation between the feature value and forward returns at 1, 5, 15, and 30 minutes
- Compute it **across the full instrument basket, not one ticker** (Research Standard 2). Report the distribution of IC across instruments, not just the mean. A feature with mean IC 0.03 that is positive on 18 of 20 instruments is far more interesting than one with mean IC 0.06 that is positive on 4.
- Compute it separately by session window and by volatility regime — many features only work in specific conditions, and unconditional testing hides real effects
- Report IC decay across the four horizons: this tells you the natural holding period, and **if the strongest signal decays at 15 minutes rather than 5, that is a finding I want to hear about**
- Include a **feature correlation matrix**: features that are nearly identical give false confidence when combined

**Establish the hurdle before you look.** Compute what IC is required, given the instrument's volatility and your Phase 1 cost structure, for a strategy to be profitable net of costs. Features below that hurdle are not worth building around no matter how appealing the mechanism.

**Run a null control.** Generate 20 random features with no predictive content and measure their ICs the same way. This gives you the noise floor. **Any real feature must clear the best random feature by a clear margin** — if your top feature's IC sits inside the random distribution, you have found nothing, and knowing that now saves weeks.

**Deliverables:** the feature library as tested code, plus `RESEARCH/10-feature-ranking.md` with the ranked table, the IC distributions, the correlation matrix, the cost hurdle, and the null-control comparison.

**Then build strategies from the top features, not from chart patterns.** Prefer **combining several weakly-predictive, uncorrelated features** over hunting for one strong one — an ensemble of five features with IC 0.02 each and low mutual correlation typically beats any single feature and is far more robust out of sample. Single-feature strategies are fragile and decay fast.

**🔴 AUDIT 4B — Signal validity.** Charter: *"These ICs are inflated."* Check for lookahead in feature construction (the most common source, and it is subtle — a feature using the current bar's close to predict the current bar's return will look spectacular), improper handling of overlapping return windows, ICs computed on data that later becomes the test set, survivorship in the basket, and whether the null control was genuinely random.

**🛑 GATE 2c:** Show me the ranked feature table and the null-control comparison. **If nothing clears the cost hurdle and the noise floor, that is a major finding and we should discuss before you build anything.**

---

## PHASE 5A — RUN THE BASELINE (control arm)

Before building anything of your own, implement the frozen spec from `RESEARCH/01-baseline-spec.md` exactly as written and run it through the harness.

- **Zero tuning.** No parameter sweeps, no "obvious" fixes, no substituting a better exit. Where the spec listed an ambiguity, use the interpretation you already committed to. If you find yourself wanting to change something, write it down as a Phase 5B idea instead.
- Run it on the video's stated instrument and timeframe, then on the instrument Lane C recommended. Report both.
- Report the full metric set (below) at three cost levels: **zero cost** (what the video implicitly claims), **realistic cost**, and **stressed cost**. The gap between the first two is usually the entire story.
- Compare measured results against the video's claimed statistics. Quantify the divergence.
- Revisit your predicted failure modes from Phase 1.0 — which fired, which didn't, what surprised you.

This result is **BASELINE_0** and it is now the number to beat. Record it prominently in the repo.

**Expect this to fail after costs — that is the most common outcome and it is not a problem.** A dead baseline still does three useful jobs: it proves the harness is honest, it establishes a floor, and it tells you which specific component (entry quality, exit, cost drag, instrument choice) is the binding constraint. Diagnose *why* it failed by ablating components one at a time — that diagnosis is the most valuable input to Phase 5B. Do not attempt to rescue it.

**Deliverable:** `RESEARCH/09-baseline-results.md`
**🔴 AUDIT 5 — Overfitting red team (first pass).** Focus here on whether the baseline was tuned in any form, and whether the failure diagnosis is supported by the ablations or is a just-so story.
**🛑 GATE 3a:** Present BASELINE_0 and the failure diagnosis before building the challenger.

---

## PHASE 5B — CHALLENGER STRATEGY

Now build the hypotheses I selected at Gate 1b — by default three, developed in parallel — informed by what the baseline taught you **and constrained by the Phase 4B feature rankings**. A hypothesis whose core feature did not clear the IC hurdle should be dropped or reformulated before you spend time on it, whatever its narrative appeal.

Develop them **in parallel rather than sequentially**, and resist the pull to fall in love with whichever shows an early good number. Early leaders are usually the luckiest, not the best. Carry all three to the same level of validation before ranking them.

- Implement: signal generation, regime/session filters, entry, stop, target, trailing/time-based exit, position sizing (fractional-Kelly, capped, with a hard per-trade risk limit), max concurrent positions, end-of-day flatten.
- **Every result is reported against BASELINE_0**, on the same data, same costs, same windows. Absolute numbers alone are not acceptable.
- Keep a changelog of every variant you test and its result. This count feeds the multiple-testing adjustment — an improvement found on the 40th variant is not the same as one found on the 2nd, and you must report which it was.
- Backtest on train. Tune on validation. **Then one single run on test.**
- Walk-forward across the full history. Report per-window, not just aggregate.
- Report: net expectancy per trade in bps, Sharpe, Sortino, max drawdown, drawdown duration, win rate, avg win/loss, profit factor, trades/day, **cost drag as % of gross P&L**, and performance broken out by session hour, by volatility regime, and by year.
- Stress it: 2× slippage, 2× latency, commissions +50%, worst 5% of spreads, and a version where every fill is at the far touch. If it dies under any of these, say so plainly.

**🔴 AUDIT 5 — Overfitting red team (second pass).** Full charter this time. Run before the gate.
**🛑 GATE 3b:** Present results as a side-by-side table against BASELINE_0, with an explicit "reasons this might not be real" section and the variant count.

Three conditions to pass, all required: net expectancy after costs clearly positive across walk-forward windows; outperformance of BASELINE_0 by a margin surviving the multiple-testing adjustment; and **positive performance on a majority of comparable instruments it was not tuned on** (Research Standard 2 — this is the condition most likely to fail, and the one that most reliably separates a real effect from a fitted one). Do not talk yourself into a marginal edge — at small account size, a marginal edge is a guaranteed loss after costs.

---

## PHASE 5C — ITERATION LOOP

**Assume the first pass fails.** That is the normal outcome, not a crisis, and having a defined next move is most of what separates a project that eventually works from one that stalls at the first disappointment.

If no candidate clears Gate 3b, do not stop and do not lower the bar. Work down this ladder in order, and return to Gate 3b after each rung:

1. **Diagnose, don't discard.** For each failed candidate, decompose: was the entry signal informative at all (check raw forward-return conditional on signal, before any exit logic)? Was the exit destroying a real edge? Was cost drag the whole gap? A signal with genuine predictive power and a bad exit is a fixable project; a signal with no predictive power is not. Say which you have.
2. **Attack costs first** (Lane G). If the gap to profitability is smaller than the total cost load, cost reduction is the highest-probability fix available and the only deterministic one. Better broker, maker-side fills, larger per-trade size to amortize fixed costs, fewer and higher-conviction trades.
3. **Change the instrument, not the logic.** Re-run the surviving signals across the full Lane C ranked list. Edge-to-cost ratio varies enormously across instruments and a strategy that fails on one may work on another for reasons that have nothing to do with the signal.
4. **Change the session window.** Open, midday, and close are different markets. Test each separately — an edge concentrated in the first hour is a real and tradeable finding even if the all-day version is flat.
5. **Question the timeframe.** If the evidence points to the edge living at 1-minute, 15-minute, or 30-minute resolution rather than 5, **tell me.** I specified 5 minutes as a starting assumption, not a constraint worth losing money over.
6. **Advance the next hypotheses** from the Phase 2 list, in rank order.
7. **Return to Phase 1 research** with everything you learned about what specifically didn't work.

**Iteration budget: cycle this loop up to three times.** Track cumulative variant count across all cycles — the multiple-testing adjustment gets stricter every pass, and this is exactly the mechanism that stops "keep trying until something looks good" from manufacturing a fake edge.

**If the budget is exhausted with no candidate clearing the bar**, deliver the honest negative: what you tested, the numbers, why each failed, what the binding constraint is (cost, signal, or size), and precisely what would have to change — a larger account, a different asset class, a different timeframe, better data, cheaper execution — for this to become viable. Then finish and deploy the system in paper mode so the infrastructure is ready when a real signal shows up. **That is a completed project, not a failed one.**

---

## PHASE 6 — LIVE ENGINE (greenfield — build all of this)

- **Market data:** streaming websocket (not REST polling), 5-minute bar aggregation from ticks/1m with correct bar-close semantics, reconnect + gap backfill + staleness watchdog, NTP-synced clock.
- **Execution:** marketable-limit orders rather than market orders (protects against bad fills on thin books), correct rounding to tick/lot, idempotent client order IDs, bracket/OCO for stop and target, fill reconciliation, **position state recovered from the broker API on restart — never from local memory alone**.
- **Latency:** be honest about the ceiling here. For retail equities, broker routing dominates and colocation is meaningless — you are not competing with HFT and should not pretend to. Do the things that actually matter: server in the same region as the broker's API endpoint, persistent websocket connections, connection pooling and keep-alive, precomputed indicator state (incremental updates, never recomputing full history per bar), async I/O throughout. Profile before optimizing. Report measured tick-to-order p50/p95/p99 before and after, and stop when further work stops paying.
- **Risk layer, enforced in code, independent of strategy logic:** max daily loss kill-switch, max position size, max concurrent positions, max trades/hour, consecutive-loss circuit breaker, auto-flatten + halt on data staleness or repeated API errors, halt on detected trading halt, mandatory flat before close.
- **Persistence and observability:** trade/signal/order logging to the DB schema the GUI expects, structured application logs, alerting (Telegram or similar) on fills, errors, and circuit-breaker trips.
- Deploy under systemd with restart policy, log rotation, and a health endpoint. Do not disturb anything already running on the server.

**🔴 AUDIT 6 — Live systems & failure modes.** Run against the deployed code, not a description of it. Every BLOCKER must be fixed and the audit re-run before paper trading begins.
**🛑 GATE 3c:** Show me the audit findings and your responses.

---

## PHASE 7 — GUI INTEGRATION

Per the Phase 0 GUI contract, register this bot under the **Trading** menu. It must not appear under Prediction. Verify by actually loading the GUI.

Surface: live status, current position(s), open orders, today's P&L, equity curve, trade log, current signal state, latency stats, circuit-breaker status, and a manual **kill / flatten** button that actually works.

Match existing GUI styling and conventions. Extend existing components rather than forking or duplicating them.

---

## PHASE 8 — PAPER, THEN LIVE

1. Run paper/sandbox for **a minimum of 20 trading sessions**, and long enough to accumulate at least 100 trades, across varied market conditions. Do not shorten this. Paper trading is the only stage that tests the strategy, the data pipeline, the execution logic, and the broker's real behavior simultaneously, and it is the last checkpoint before the mistakes cost money.
2. **Compare paper results against backtest expectations for the same window.** Divergence means the backtest is wrong — investigate before proceeding. Pay specific attention to realized fill prices versus NBBO at signal time.
3. Write `RUNBOOK.md`: how to start/stop, what each alert means, failure modes, manual intervention steps, rollback procedure.

**🔴 AUDIT 7 — Pre-mortem.** Run last, after the runbook exists.
**🛑 GATE 4 — GO LIVE:** Present the paper-vs-backtest comparison, the runbook, and a proposed starting size at a small fraction of the account. Wait for my explicit approval.

**Then ramp in stages, never in one step.** Propose a ladder — minimum viable size, then increments — with an explicit criterion for advancing between rungs and an explicit criterion for stepping back down. Live-at-minimum-size is its own validation stage, not a formality: it is the first time you see real fills, real slippage, and real routing. Compare realized live fills against both the backtest and the paper results, and treat any divergence as a finding to investigate rather than noise to average out. Each ramp step requires my approval.

---

## DEFINITION OF DONE

- Bot runs unattended on the server, survives restart and network drop, recovers position state correctly from the broker
- **Either** positive net expectancy demonstrated out-of-sample and confirmed in paper trading after realistic costs at my actual size, with documented outperformance versus BASELINE_0 and the cumulative variant count disclosed — **or** a rigorous, well-documented negative result with the binding constraint identified. Both count as done. A fabricated positive does not.
- Visible and controllable in the GUI Trading menu, with a working kill switch
- Risk limits enforced in code and verified by test
- `RESEARCH/` complete and sourced, `PLAN.md` current, `RUNBOOK.md` written
- All seven audits complete, with every BLOCKER closed and every rejected finding justified in writing
- Tests pass; nothing in the existing GUI is broken

---

## OUTPUT DISCIPLINE

At each phase end: what you did, what you found, what you're uncertain about, what you need from me. Short. No summary padding. If something in this prompt is wrong or infeasible given what you find, say so instead of quietly working around it.

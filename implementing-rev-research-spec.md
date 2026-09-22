# Algorithmic Trading Bot Optimization & Architectural Specification
> **Target Audience:** Claude Code IDE (Autonomous Coding Agent)  
> **Objective:** Refactor trading bot pipeline to eliminate fee drag, control model bloat, enforce type-safe signal fusion, and integrate sub-100ms confidence gating.

---

## 1. Executive Summary & Core Engineering Imperative

The trading bot currently exhibits a **gross positive return that is eroded into a net loss by transaction friction, slippage, and low-conviction trade churning**. To transform this system into an institutional-grade, net-profitable automated strategy, Claude Code must implement four foundational research breakthroughs:

1. **Bloat Control via Double Lexicase Selection (DLS):** Replace standard tournament selection/depth limits with a two-stage selection mechanism that uses roulette wheel size transformation to optimize predictive performance ($R^2$) while maintaining compact program trees.
2. **Type-Safe Multi-Modal Fusion via STGP-SATA:** Structure strategy trees into segregated technical analysis (`Type_TA`) and sentiment analysis (`Type_SA`) subtrees rooted by an `AND` function to prevent feature domination and eliminate syntactically invalid operations.
3. **Walk-Forward Validation & Regime Gating:** Implement rolling 34-fold out-of-sample testing and suspend execution during low-volatility regimes ($\text{RealizedVol} < 2\%$) where signal-to-noise ratios are too low to exceed transaction fees.
4. **Sub-100ms Gatekeeper Middleware via TypeSafe Jev:** Integrate Jev (System One foundation model) as a pre-execution middleware to evaluate trade signals against calibrated probabilities ($\ge 0.85$ confidence), eliminating 80%+ of fee-bleeding noise trades and routing orders as post-only maker limit orders.

---

## 2. Module 1: Bloat Control via Double Lexicase Selection (DLS)

### 2.1 The Problem: Genetic Program Bloat
Genetic Programming (GP) parse trees naturally accumulate non-functional code blocks (**introns**) due to hitchhiking, defense against destructive crossover, and search space bias. Standard bloat control methods (e.g., hard depth limits or aggressive parsimony penalties) either fail to halt tree growth or over-exploit trivial, single-node trees, destroying predictive accuracy.

### 2.2 DLS Two-Stage Selection Mechanism
DLS separates parent selection into a fitness/semantics stage followed by a soft parsimony stage:

```
Population P (size p)
       │
       ▼
[Stage 1: Automatic 𝜖-Lexicase Selection (ALS)]
  • Runs Cap = 10 rounds across shuffled training instances
  • Filters candidates within adaptive threshold 𝜖_k
  • Outputs Candidate Pool C (|C| = Cap)
       │
       ▼
[Stage 2: Transformed Size Scoring & Roulette Wheel]
  • Inverts size scores: sc_i = max(size) + min(size) - size_i
  • Applies Roulette Wheel Selection proportional to sc_i
       │
       ▼
Selected Parent Individual
```

### 2.3 Implementation Rules for Claude Code
* **Candidate Pool Capacity ($Cap$):** Set $Cap = 10$. Increasing $Cap$ beyond 10 yields marginal size reduction while significantly increasing evaluation latency.
* **Avoid Greedy Minimum Selection:** Selecting the absolute smallest individual ($\min(\text{size})$) from $C$ causes the population to collapse into low-capacity models ($R^2$ degraded across 67/98 datasets). Roulette wheel selection maintains soft selection pressure, allowing complex, high-accuracy subtrees to survive.

---

## 3. Module 2: Structural Signal Fusion via STGP-SATA

### 3.1 The Problem: Closure Violation & Domain Domination
Standard untyped GP violates domain logic by allowing functions to process invalid inputs (e.g., taking the moving average of a discrete sentiment flag). In combined Technical Analysis (TA) + Sentiment Analysis (SA) models, continuous high-frequency price data overwhelms sparse, discrete sentiment updates (**domain domination**).

### 3.2 Dual-Branch STGP Architecture
STGP-SATA enforces static type constraints across terminal nodes, function arguments, and return types, structuring the strategy parse tree into two distinct branches:

```
                        [Root: ITE] (If-Then-Else)
                             │
                      [AND Decision Node]
                     ╱                 ╲
      [Type_SA Branch]                 [Type_TA Branch]
      • Sentiment Polarity             • Moving Average (n=5, 10)
      • Subjectivity (TextBlob)        • Momentum / ROC
      • AFINN / SentiWordNet           • Williams %R / Volatility
      • Title / Summary Sentiment      • Midprice Range
```

### 3.3 Genetic Operator Constraints
* **Type-Preserving Crossover:** Subtree crossover must exchange $SA$ subtrees strictly with donor $SA$ subtrees, and $TA$ subtrees strictly with donor $TA$ subtrees.
* **Point Mutation:** Mutation operators must only replace a node with another operator belonging to the exact same type signature.

### 3.4 Financial & Quantitative Benefits
* **Trade Frequency Reduction:** Cuts baseline trading frequency from ~230 indiscriminate trades down to 10 high-conviction trades per evaluation window.
* **Sharpe Ratio Enhancement:** Increases average Sharpe ratio from 0.15 (un-gated baseline) to **10.8** (net of $c_t = 0.025\%$ transaction cost).
* **Interpretability:** Reduces parse tree size from 82–142 unreadable nodes down to ~28 total nodes (14 per branch), producing human-auditable trading rules.

---

## 4. Module 3: Walk-Forward Validation, Microstructure & Fee Control

### 4.1 Friction & Transaction Cost Modeling
All strategy backtesting and live execution logic must explicitly incorporate transaction costs into fitness functions and position sizing:
$$\text{Cost}_{\text{trade}} = c_{\text{fixed}} + c_{\text{slippage}} \cdot |q| \cdot P_{\text{exec}}$$
where $c_{\text{fixed}} = \$1.00$, $c_{\text{slippage}} = 0.0005$ ($5\text{ bps}$), and overall transaction fee friction $c_t = 0.025\%$.

### 4.2 Regime-Dependent Execution Gating
Market microstructure signals derived from daily/intraday OHLCV data depend heavily on market volatility regimes:
* **Low-Volatility Regime ($\text{RealizedVol} < 2\%$):** Noise trading dominates; expected price movement per trade is smaller than $c_t$, causing net losses ($-0.16\%$ quarterly). **Action:** Suspend bot trading ($\text{PositionSize} = 0$).
* **High-Volatility Regime ($\text{RealizedVol} \ge 2\%$):** Information arrival rate increases; microstructure signals yield positive net returns ($+0.60\%$ quarterly, Sharpe ratio $1.01$). **Action:** Enable trading execution.

### 4.3 Validation Protocol Requirements
* **Rolling Walk-Forward Window:** Use 34 rolling out-of-sample folds ($W = 252\text{ days}$ training, $H = 63\text{ days}$ testing, step $\Delta = 63\text{ days}$).
* **Overfitting Diagnostics:** Calculate the **Deflated Sharpe Ratio (DSR)** to adjust for skewness, kurtosis, and trial count, and apply **Combinatorial Purged Cross-Validation (CPCV)** to prevent lookahead leakage.

---

## 5. Module 4: Low-Latency System One Middleware via TypeSafe Jev

### 5.1 System Architecture: Jev vs Generative LLMs
Generative LLMs (3–15s latency, $\$0.20\text{--}\$10.00/\text{MTok}$) introduce unacceptable execution slippage and exhibit a 0.58%–45.5% schema error rate that causes runtime bot crash loops. **TypeSafe Jev** is a System One foundation model ($70\text{--}500\text{ms}$ cloud / sub-$15\text{ms}$ edge latency, $\$0.042/\text{MTok}$ with free output tokens, 0% schema error rate by architectural construction).

### 5.2 The 3 Core Primitives
1. `Choice`: Selects 1-of-$N$ categorical options (up to 255 options) alongside full probability distributions and a derived confidence score.
2. `Score`: Evaluates state against an ordered 2-to-10 level rubric, returning a probability-weighted floating score.
3. `Noul`: Evaluates a single Bernoulli (yes/no) statement, returning a calibrated probability $p \in [0, 1]$.

### 5.3 The Observe-Judge-Reason-Act Integration Pattern

```
1. OBSERVE  ──> Ingest market feeds (OHLCV, order book imbalance, headline news).
2. JUDGE    ──> Execute parallel System One evaluation pass via Jev API.
3. ROUTE    ──> Evaluate Jev RLCD-calibrated confidence score:
                • High Confidence (≥ 0.85): Execute direct deterministic trade.
                • Medium Confidence (0.50 - 0.84): Route to human trader review.
                • Low Confidence (< 0.50): Escalate to System Two LLM or HOLD.
4. ACT      ──> Dispatch POST-ONLY limit order one tick inside spread (maker rebate).
```

---

## 6. Comprehensive Problem-Solution Matrix for Code Refactoring

| # | Existing Code Defect / Shortfall | Root Cause | Implemented Solution in Bot Codebase |
|---|:---|:---|:---|
| **1** | Bot gross positive but net negative/neutral due to fees. | Over-trading on low-conviction signals where expected gain < round-trip fee. | Insert Jev `Noul` / `Choice` pre-trade gate with `confidence >= 0.85`. Cuts 80%+ of marginal noise trades. |
| **2** | Severe execution slippage and high latency during news events. | Using generative LLMs (GPT-4/Sonnet) in execution loops ($3\text{--}15\text{s}$ latency). | Replace LLM execution calls with TypeSafe Jev System One model ($11\text{--}35\text{ms}$ edge latency). |
| **3** | Bot crashes during high market volatility. | LLM JSON syntax errors / markdown formatting breaks runtime parser. | Jev guarantees 0% schema error rate by architectural pre-bounding. |
| **4** | Uncontrolled GP program tree size growth (bloat). | Hitchhiking introns and crossover defense mechanisms in standard GP. | Implement Double Lexicase Selection (DLS) with candidate capacity $Cap = 10$. |
| **5** | GP evolves trivial, low-capacity 1-node trees. | Greedily selecting absolute minimum tree size ($\min(\text{size})$) during bloat control. | Use DLS Stage 2 transformed size scoring with roulette wheel selection. |
| **6** | Invalid operations (e.g., moving average of discrete sentiment). | Standard GP closure principle treating all terminals as generic floats. | Enforce Strongly Typed Genetic Programming (STGP) with strict AST type checking. |
| **7** | Technical price indicators overwhelm sparse sentiment signals. | Domain domination in untyped multi-modal feature vectors. | Implement STGP-SATA dual-branch tree (`Type_SA` and `Type_TA`) joined at root `AND`. |
| **8** | Heavy losses during quiet, sideways market regimes. | Trading during low volatility where signal-to-noise ratio is too weak. | Add code-level regime gate: set $\text{PositionSize} = 0$ when $\text{RealizedVol} < 2\%$. |
| **9** | Strategy backtest looks great in-sample but fails in live trading. | Overfitting, lookahead bias, unadjusted multiple testing. | Enforce 34-fold Walk-Forward Validation, CPCV data purging, and Deflated Sharpe Ratio (DSR). |
| **10** | Taker order execution fees eating profit margins. | Using market orders that pay taker fees and incur adverse selection. | Route execution via post-only limit orders placed one tick inside the spread. |

---

## 7. Production Python Implementation Specifications

Below is the exact FastAPI microservice implementation pattern for Claude Code to deploy as the bot's execution router:

```python
import time
import os
from typing import Optional
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
from typesafe_sdk import TypeSafeClient, Choice, Score, Noul
from opentelemetry import trace

tracer = trace.get_tracer("algo.bot.execution.tracer")
app = FastAPI(title="Claude Code Anti-Fee Trading Gateway")

# Pin model version for calibration stability
jev_client = TypeSafeClient(
    api_key=os.getenv("TYPESAFE_API_KEY"),
    timeout=2.0
)

class MarketStatePayload(BaseModel):
    ticker: str
    bid_ask_spread_bps: float
    volume_imbalance_5m: float
    realized_volatility_20d: float
    headline_text: Optional[str] = None

class TradeExecutionSignal(BaseModel):
    action: str = Field(description="BUY, SELL, or HOLD")
    confidence: float
    is_high_conviction: bool
    post_only_limit_allowed: bool
    latency_ms: float

@app.post("/v1/gatekeeper/evaluate", response_model=TradeExecutionSignal)
async def gatekeeper_evaluate(payload: MarketStatePayload):
    t0 = time.perf_counter()

    with tracer.start_as_current_span("jev_gatekeeper_pass") as span:
        # 1. Hard Code-Level Regime Gate (Low Volatility Protection)
        if payload.realized_volatility_20d < 0.02:
            span.set_attribute("gatekeeper.status", "REJECTED_LOW_VOLATILITY")
            return TradeExecutionSignal(
                action="HOLD",
                confidence=1.0,
                is_high_conviction=False,
                post_only_limit_allowed=False,
                latency_ms=(time.perf_counter() - t0) * 1000
            )

        # 2. Issue Parallel System One Pass to TypeSafe Jev
        jev_response = jev_client.system_one(
            state=payload.model_dump(),
            questions={
                "directional_intent": Choice(
                    instructions="Judge short-term price direction based on market state.",
                    criteria={
                        "bullish": "Positive order flow imbalance and bullish news tailwind.",
                        "bearish": "Negative imbalance and downside risk event.",
                        "neutral": "Balanced order book or conflicting signals."
                    }
                ),
                "noise_trade_risk": Noul(
                    instructions="Is this a low-conviction noise signal likely to be eaten by fees?"
                )
            }
        )

        direction = jev_response.answers["directional_intent"]
        noise_prob = jev_response.answers["noise_trade_risk"].noul
        latency_ms = (time.perf_counter() - t0) * 1000

        # Attach telemetry to OpenTelemetry trace
        span.set_attribute("jev.choice", direction.choice)
        span.set_attribute("jev.confidence", direction.confidence)
        span.set_attribute("jev.noise_prob", noise_prob)
        span.set_attribute("jev.latency_ms", latency_ms)

        # 3. Confidence Threshold Gating
        is_high_conviction = (direction.confidence >= 0.85) and (noise_prob < 0.20)

        if not is_high_conviction or direction.choice == "neutral":
            return TradeExecutionSignal(
                action="HOLD",
                confidence=direction.confidence,
                is_high_conviction=False,
                post_only_limit_allowed=False,
                latency_ms=latency_ms
            )

        # 4. Post-Only Limit Order Router Check
        allow_post_only = payload.bid_ask_spread_bps >= 1.5

        return TradeExecutionSignal(
            action=direction.choice.upper(),
            confidence=direction.confidence,
            is_high_conviction=True,
            post_only_limit_allowed=allow_post_only,
            latency_ms=latency_ms
        )
```

---

## 8. Summary Checklist for Code Deployment

- [ ] **Configure STGP AST Type Checker:** Register static types (`Type_TA`, `Type_SA`, `Type_Bool`) and enforce node type matching during initialization, crossover, and mutation.
- [ ] **Implement DLS Selection Operator:** Set candidate pool capacity $Cap = 10$, run ALS selection, apply $sc_i$ inverted size transformation, and select parents via roulette wheel.
- [ ] **Integrate Volatility Regime Switch:** Wrap execution triggers with a `RealizedVol >= 0.02` threshold check to halt trading during quiet market regimes.
- [ ] **Deploy Jev Gatekeeper Middleware:** Route all raw bot signals through `FastAPI` + `typesafe_sdk`, applying the `confidence >= 0.85` threshold.
- [ ] **Switch Broker Execution to Post-Only:** Ensure all approved buy/sell orders are submitted as post-only limit orders placed 1 tick inside the spread to capture maker rebates.

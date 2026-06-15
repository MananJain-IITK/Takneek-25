# PESHWAS — BOLT: Book Oriented Limit Trader

> *A multi-strategy algorithmic trading system built for Zenith Takneek, featuring 4 distinct strategies across 7 assets, backtested on a microstructure-aware simulation engine.*

---

## Strategies

### Asset 1 — Acceleration-Based Momentum (MACD)
Exploits the **second derivative of price** (acceleration) as a leading indicator. A cross-correlation analysis revealed a **0.995 correlation** between a proprietary `no_trade` signal and Asset 1's momentum (leading by 1 step). A **double-smoothed MACD** triggers entries:

```
MACD        = EMA(S, 12) − EMA(S, 26)
Signal Line = EMA(EMA(MACD, 9), 9)
```
| Signal | Condition |
|---|---|
| `+1` Long | `MACD > SignalLine` |
| `-1` Short | `MACD < SignalLine` |
| `0` Hold | No crossover |

---

### Assets 2, 4, 7 — Statistical Arbitrage (Pairs Trading)
A **market-neutral** strategy exploiting a cointegrated 1:1:1 relationship between three assets (Assets 2 & 7 have **0.73 correlation**). Trades the spread:

```
Sₜ = P₇ₜ − P₄ₜ − P₂ₜ
```

- **Short spread** (`Sₜ > τ`): Sell 7, Buy 2 & 4
- **Long spread** (`Sₜ < −τ`): Buy 7, Sell 2 & 4

The spread exhibits a quasi-sinusoidal mean-reverting pattern ideal for high-frequency trading.

---

### Assets 3 & 6 — Stochastic Residual Spread (Kalman Filter Pairs)
An econometric pairs trading strategy on two assets with **~85% historical correlation**. Models the latent mispricing as a **Vasicek mean-reverting process**, estimated in real-time via a **Kalman Filter**:

```
Transition:  dxₜ = κ(θ − xₜ)dt + σdBₜ
Measurement: yₖ = xₖ + Γrᵐₖ + Hωₖ
```

Adaptive thresholds based on rolling volatility generate signals. Parameters `{θ, κ, σ, Γ, H}` are fit via Maximum Likelihood Estimation (MLE) on a rolling basis.

---

### Asset 5 — Momentum-Equilibrium (Order Book Imbalance)
Combines **microstructure signals** with a momentum framework:

1. **Order Book Imbalance (OBI)** quantifies real-time supply/demand pressure
2. **Synthetic predictive candles** encode OBI into Open/High/Low/Close/Volume
3. **Momentum Conductor** `(max(High₉) + min(Low₉)) / 2` crosses **Equilibrium Baseline** `(max(High₂₆) + min(Low₂₆)) / 2` to trigger signals
4. **Conviction scoring** sizes positions proportionally to signal strength

---

## Backtesting Engine

A microstructure-aware simulation that "walks the book" on every trade:

- **Realistic execution**: Liquidity consumed level-by-level from the order book (±50 unit position cap)
- **Transaction costs**: PnL updated fill-by-fill with slippage parameter `cₜₓₙ`
- **Volume Fix**: Unfilled orders carry forward; position reversals auto-square
- **Stop-Loss**: Percentage-based dynamic stop on every asset in both directions
- **Terminal liquidation**: All positions closed at final bid/ask at `t = T`

---

## Backtesting Results

| Asset | Strategy | PnL |
|---|---|---|
| Asset 1 | Acceleration MACD | 319,067.78 |
| Asset 2 | Stat Arb | 44,600.23 |
| Asset 3 | Kalman Pairs | 113,655.10 |
| Asset 4 | Stat Arb | 54,206.36 |
| Asset 5 | Momentum-Equilibrium | 702,892.75 |
| Asset 6 | Kalman Pairs | 95,531.10 |
| Asset 7 | Stat Arb | 89,492.47 |
| **Total** | **Multi-Strategy Portfolio** | **1,419,445.80** |

---

## Stack

```
pandas · numpy · matplotlib · yfinance
```

## Setup

```bash
pip install pandas numpy matplotlib yfinance jupyter
jupyter notebook Takneek_SntCode.ipynb
```

## Project Structure

```
.
├── Takneek_SntCode.ipynb   # Strategy implementation & backtesting
└── README.md
```

---

*Submitted for Zenith Takneek — BOLT (Book Oriented Limit Trader) by PESHWAS*

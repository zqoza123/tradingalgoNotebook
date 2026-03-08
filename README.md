# Overnight Mean-Reversion Research on Equity Index Futures (BOT5)

## Why This Project Exists
I've been trading MES overnight on prop firm accounts (Alpha Futures, TopStep) and made 5+ figures doing it with RSI divergence setups. This project was about taking that same edge and asking: is it actually real, or have I just been lucky?

The strategy is based on a retail-style signal (RSI divergence + distance-from-EMA), but I wanted to evaluate it with real rigor — institutional data, reproducible backtesting, statistical significance testing, out-of-sample validation, exposure decomposition, regime analysis, and transaction cost sensitivity. Not just "it looks good on TradingView."

This repo is intentionally focused on one strategy notebook (`BOT5`) so the full research chain is easy to audit end-to-end.

## Research Question
Can an overnight-only mean-reversion strategy in index futures produce risk-adjusted returns that are:
1. Statistically distinguishable from noise,
2. Not just hidden market beta,
3. Reasonably stable under small parameter perturbations,
4. Still positive out-of-sample,
5. Improved by simple regime controls?

## Economic Hypothesis
Overnight futures trading is structurally different from regular cash hours: thinner liquidity, episodic hedging/roll flows, and higher inventory risk for liquidity providers. In that setting, local dislocations can overshoot and then mean-revert intranight.

BOT5 expresses this by combining divergence detection (momentum exhaustion proxy) with a distance-from-EMA constraint (only trade measurable dislocations). The expected failure mode is one-way overnight flow regimes, where mean-reversion fades get repeatedly run over.

## Data
- **Source:** Databento CME Globex (`GLBX.MDP3`) local DBN file
- **File:** `glbx-mdp3-20230306-20260304.ohlcv-1m.dbn.zst`
- **Frequency:** 1-minute OHLCV
- **Sample:** 2023-06-01 to 2026-03-04 (US/Eastern)
- **Session:** strategy trades only 00:00–08:00 ET

The loader normalizes multi-symbol DBN content to one front-like minute stream by filtering to outright contracts and selecting the max-volume contract per timestamp.

## Strategy Snapshot
- **Signal:** RSI divergence + distance from EMA200
- **Session:** 00:00–08:00 ET (overnight only)
- **Slippage:** 1 tick per side
- **Commission:** $1.25/side ($2.50 round-trip)

## Validation Layers
1. **Permutation significance test** — sign-randomization null on Sharpe
2. **Alpha/Beta decomposition** — vs ES buy-and-hold
3. **Parameter stability grid** — Sharpe across nearby parameter configurations
4. **Expanding-window walk-forward** — monthly OOS evaluation
5. **Regime split** — pre/post 2025-04-10 performance breakdown
6. **Realized-volatility regime filter** — RV20 quantile sweep
7. **Drawdown forensics** — Aug-Sep 2025 deep dive + filter mitigation
8. **Transaction cost sensitivity** — commission sweep from $0 to $7.50 RT

## Key Results
| Metric | Value |
|--------|-------|
| Permutation p-value | 0.0036 |
| Full-sample Sharpe | 1.399 |
| Walk-forward OOS Sharpe | 1.793 |
| Beta to benchmark | 0.015 |
| Annualized alpha | 3.95% |
| Parameter Sharpe range | 0.430 (stable) |
| Pre 2025-04-10 return | +9.13% |
| Post 2025-04-10 return | +6.04% |
| RV40 filtered Sharpe | 2.179 |
| RV40 filtered return | +20.5% (280 trades) |

## Limitations
I want to be upfront about what this is and isn't:
- The signal is still indicator-based. It's not a microstructure-native alpha model.
- The regime filter is simple (realized-vol) and needs deeper causal work.
- Contract stitching from local DBN is a practical approximation, not exchange-provided continuous data.
- These are research outputs, not live-trading PnL statements.

## What's Next
To push this toward an institutional standard:
1. Add state-space/Kalman residual features to replace or gate indicator-only entries
2. Incorporate microstructure covariates (order-flow imbalance, queue pressure, spread dynamics)
3. Run multi-market robustness (ES/MES/NQ) with common risk controls
4. Move from parameter tuning to proper model-selection with strict leakage guards

## Repository Contents
- `notebooks/bot5_backtest.ipynb` — full strategy + diagnostics notebook
- `outputs/bot5_backtest.rigor_fullperiod.tex` — LaTeX export of executed notebook
- `outputs/bot5_backtest.rigor_fullperiod_files/` — figures referenced by LaTeX
- `mesdata/` — local DBN data source (gitignored)

## Reproducibility
Open `notebooks/bot5_backtest.ipynb` and run all cells top-to-bottom.

## Disclaimer
This is educational/research work and not investment advice.

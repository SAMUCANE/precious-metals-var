# Precious Metals Portfolio — Value-at-Risk & Risk Management

A complete market-risk workbench for a **\$1,000,000 equal-weight portfolio** of
Gold, Silver and Platinum COMEX futures, built as a single, fully reproducible
Jupyter notebook. It computes Value-at-Risk by three independent methods,
Expected Shortfall, a Kupiec backtest, historical stress scenarios and a
rolling-window VaR — exporting every table to Excel and every chart to PNG.

> **Sample period:** 2019-01-01 → 2024-12-31 · **Data:** live COMEX futures via
> `yfinance` (`GC=F`, `SI=F`, `PL=F`), with liquid-ETF fallback (`GLD`, `SLV`, `PPLT`).

---

## Portfolio

| | |
|---|---|
| Notional | \$1,000,000 |
| Weights | 1/3 Gold · 1/3 Silver · 1/3 Platinum |
| Returns | daily log returns, `r = ln(P_t / P_{t-1})` |

---

## What the notebook does

| Section | Content |
|---|---|
| **0** | Data loading (futures → ETF fallback), log returns, descriptive stats |
| **1** | Static & rolling correlation, Jarque–Bera normality tests |
| **2** | VaR — Historical Simulation, Parametric (variance–covariance), Monte Carlo |
| **3** | Expected Shortfall (CVaR) + ES/VaR tail-risk ratio |
| **4** | Backtesting the VaR model (Kupiec proportion-of-failures test) |
| **5** | Historical stress scenarios (COVID, Russia invasion, gold flash crash, silver squeeze) |
| **6** | Rolling 252-day VaR (model stability through time) |
| **7** | Consolidated risk summary |

---

## Headline results (live data, N = 1,508 daily returns)

| Metric | Value |
|---|---|
| VaR 95% (1-day, Historical) | **\$22,962** (2.30% of NAV) |
| VaR 99% (1-day, Historical) | **\$37,985** (3.80% of NAV) |
| VaR 99% (10-day) | **\$120,121** (12.0% of NAV) |
| Expected Shortfall 99% (1-day) | **\$57,162** — ES/VaR = **1.50** |
| Parametric vs Historical (99%) | **−12.1%** (normal model understates the fat tail) |
| Kupiec backtest | 16 breaches (1.06% vs 1.0%) → **model ACCEPTED** (p = 0.81) |
| Worst stress scenario | **COVID crash: −\$272,527** over 23 days |
| Full-sample correlations | Gold–Silver 0.78 · Gold–Platinum 0.53 · Silver–Platinum 0.63 |

**Key insight:** the normal-distribution (Parametric) VaR sits ~12% *below* the
Historical VaR at 99% — the cost of ignoring fat tails, consistent with the
rejected Jarque–Bera tests. Correlations also spike toward 1 in crises, so the
diversification benefit shrinks exactly when losses cluster.

---

## Repository contents

```
precious_metals_var.ipynb          # the analysis notebook (run top to bottom)
build_pm_var_notebook.py           # generator script that builds the notebook
precious_metals_var_overview.tex   # in-depth LaTeX methodology & code overview
precious_metals_tex.pdf            # compiled overview
precious_metals_var_results.xlsx   # all result tables (7 sheets, S0–S6)
fig0_prices_returns.png            # price & return panels
fig1_rolling_correlations.png      # rolling 90-day correlations
fig1c_distributions.png            # return distributions vs normal
fig3_var_es_distribution.png       # P&L distribution with VaR & ES
fig4_var_backtesting.png           # VaR breaches over time
fig5_stress_scenarios.png          # stress-scenario P&L
fig6_rolling_var.png               # rolling 252-day VaR
requirements.txt
```

The Excel workbook has one sheet per section: `S0_Returns`, `S1_Correlations`,
`S2_VaR_Summary`, `S3_ES`, `S4_Backtesting`, `S5_Stress`, `S6_RollingVaR`.

---

## Quick start

```bash
# 1. install dependencies
pip install -r requirements.txt

# 2a. run the existing notebook
jupyter notebook precious_metals_var.ipynb

# 2b. ...or regenerate it from scratch and execute end-to-end
python build_pm_var_notebook.py
jupyter nbconvert --to notebook --execute --inplace precious_metals_var.ipynb
```

The Monte-Carlo seed is fixed (`np.random.seed(42)`) so results are
reproducible. Every download is wrapped in `try/except`; if the futures ticker
fails the loader falls back to the ETF proxy and prints a source tag.

---

## Methodology notes

- **Historical Simulation** — empirical percentile of realised P&L; no
  distributional assumption, keeps real fat tails and crisis days.
- **Parametric** — `VaR = (−μ + z·σ)·V` with `z₉₅ = 1.645`, `z₉₉ = 2.326`.
- **Monte Carlo** — 10,000 correlated draws from the empirical mean vector and
  covariance matrix via Cholesky decomposition.
- **Horizon scaling** — 10-day VaR = 1-day VaR × √10.
- **Expected Shortfall** — mean loss conditional on breaching VaR; a *coherent*
  risk measure (Basel FRTB standard).
- **Kupiec POF test** — likelihood-ratio test, `LR ~ χ²(1)`, of whether the
  observed breach rate matches the target (1% at 99% confidence).

A full derivation of every formula, with the corresponding code, is in
`precious_metals_var_overview.tex` / `precious_metals_tex.pdf`.

---

## Disclaimer

For research and educational purposes only. Not investment advice. Market data
is provided by Yahoo Finance via `yfinance` and is subject to its terms of use.

## License

Released under the MIT License — see [LICENSE](LICENSE).

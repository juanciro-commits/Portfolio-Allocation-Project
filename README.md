# Portfolio Allocation Project

Multi-asset portfolio allocation using 10 ETFs (U.S. and international stocks, Treasuries, corporate and EM bonds, gold and REITs), with daily data from January 2016 to October 2026. The project builds minimum-variance portfolios and tests them out of sample against an equal-weight (1/N) benchmark.

| Ticker | Asset class | Ticker | Asset class |
|---|---|---|---|
| SPY | U.S. large stocks (S&P 500) | IEF | Intermediate U.S. Treasuries (7–10 yr) |
| IWM | U.S. small stocks (Russell 2000) | LQD | Investment-grade corporate bonds |
| EFA | Developed international stocks | EMLC | Emerging-market local-currency bonds |
| EEM | Emerging-market stocks | GLD | Gold |
| TLT | Long U.S. Treasuries (20+ yr) | VNQ | U.S. real estate (REITs) |

## Notebooks

Run them in order: each part uses `returns.csv` from Part 1.

### Part 1: Data pipeline and exploratory analysis

[`portfolio_data_pipeline.ipynb`](portfolio_data_pipeline.ipynb) downloads adjusted prices, builds daily log returns and studies their properties before any optimization:

- Annualized return, volatility, skewness and excess kurtosis, with a robustness check that excludes the COVID crash.
- Correlations under stress: how co-movements changed during the COVID crash.
- Rolling stock–bond correlation (SPY vs TLT) and the regime change after the 2022 inflation shock.

### Part 2: Minimum-variance portfolio

[`portfolio_min_variance.ipynb`](portfolio_min_variance.ipynb) estimates $\Sigma$ on 4 years of data, computes the minimum-variance weights

$$\hat{\mathbf a} = \frac{\hat\Sigma^{-1}\boldsymbol\iota}{\boldsymbol\iota^\top\hat\Sigma^{-1}\boldsymbol\iota}$$

and tests them on the following year (2020, 2022 and 2025), including the borrowing cost of the short positions.

### Part 3: Rolling-window backtest

[`portfolio_rolling_backtest.ipynb`](portfolio_rolling_backtest.ipynb) re-estimates $\Sigma$ every month on the last 3 years of data and holds the new weights for the next month, giving one continuous out-of-sample record from February 2019 to October 2026. It also analyzes how the weights move, turnover and trading costs, a no-trade band that only rebalances when the weights change enough, and a leveraged version scaled to 1/N's volatility, compared with Sharpe ratios.

### Part 4: Long-only minimum variance

[`portfolio_long_only.ipynb`](portfolio_long_only.ipynb) adds the constraint that every weight must be between 0 and 1 (no short sales). Since there is no closed-form solution, the weights are found with a numerical optimizer (`scipy.optimize.minimize`, SLSQP). It reruns the rolling backtest of Part 3 and compares long-only minimum variance with the unconstrained version and 1/N: performance, how concentrated the weights are, and turnover.

## Key results

Rolling backtest, February 2019 – October 2026 (93 monthly rebalances):

| | Minimum variance | Long-only min variance | Leveraged min variance | 1/N |
|---|---|---|---|---|
| Annual return | 2.5% | 2.3% | 0.1% | 6.3% |
| Annual volatility | 4.8% | 6.9% | 14.7% | 11.6% |
| Sharpe ratio | −0.04 | −0.03 | −0.11 | 0.36 |
| Maximum drawdown | −13.7% | −19.9% | −45.2% | −24.9% |
| Growth of $1 | $1.21 | $1.19 | $1.01 | $1.62 |

Sharpe ratios use the 3-month T-bill rate, which averaged 2.75% over the backtest. The leveraged version targets 1/N's volatility each month, financed at the T-bill rate.

- **Lower risk, not a better trade-off.** Minimum variance had less than half the volatility of 1/N and a much smaller worst fall, but its return was slightly below the T-bill rate, so its Sharpe ratio was slightly negative.
- **Leverage doesn't fix it.** Leveraged to 1/N's volatility, minimum variance ended almost flat with a −45% drawdown: leverage amplifies a negative excess return. It also overshot the target volatility, because in-sample volatility underestimates real risk.
- **It behaves like a leveraged Treasury position:** about 124% long IEF financed by about −49% in TLT on average, with total short positions of 48%–83% of capital.
- **Without shorts, risk rises but return doesn't change.** The long-only version holds about 4 assets, anchored on IEF (66% on average). Without the TLT short to hedge interest-rate risk, it lost twice as much in 2022 (−16.3% vs −8.0%). It is more practical, but not more efficient.
- **Regime matters.** It protected well in the COVID crash (a growth shock) but had its worst loss in 2022 (an inflation shock), when Treasuries fell. This matches the change in the stock–bond correlation found in Part 1.
- **Trading costs are negligible** with liquid ETFs (below 0.1% per year). A no-trade band cuts the number of trades by two thirds without changing the risk, but saves very little.
- **Single test years can mislead.** In Part 2, out-of-sample volatility was about 2.7 times the in-sample estimate in 2020 and 2022, while 2025 looked unusually good.

## Data

| File | Content |
|---|---|
| `prices.csv` | Adjusted daily closing prices (dividends and splits), from Yahoo Finance via `yfinance` |
| `returns.csv` | Daily log returns |
| `risk_free.csv` | 3-month U.S. T-bill yield (`^IRX`, annualized %), used for Sharpe ratios and financing costs in Parts 3 and 4 |

Part 1 loads `prices.csv` if it exists, so results are reproducible without downloading the data again. To extend the sample, change `END` in Part 1, delete `prices.csv` and rerun the notebooks in order.

## Setup

```
pip install numpy pandas scipy matplotlib yfinance plotly nbformat
```

## Next steps

- A maximum weight per asset (for example 40%) in the long-only version, to force more diversification.
- Net returns after the borrow fee on the short positions.
- Assets that hedge inflation shocks, such as T-bills or TIPS.
- Alternative covariance estimators (EWMA, Ledoit-Wolf shrinkage) and window lengths.
- Constrained mean-variance portfolios.

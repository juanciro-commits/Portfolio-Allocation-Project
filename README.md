# Portfolio Allocation Project

Multi-asset portfolio allocation using 9 ETFs (U.S. and international stocks, Treasuries, corporate bonds, gold and REITs), with daily data from January 2005 to October 2026. The project builds minimum-variance portfolios and tests them out of sample against an equal-weight (1/N) benchmark, over a period that includes the 2008 financial crisis, the COVID crash and the 2022 inflation shock.

| Ticker | Asset class | Ticker | Asset class |
|---|---|---|---|
| SPY | U.S. large stocks (S&P 500) | IEF | Intermediate U.S. Treasuries (7–10 yr) |
| IWM | U.S. small stocks (Russell 2000) | LQD | Investment-grade corporate bonds |
| EFA | Developed international stocks | GLD | Gold |
| EEM | Emerging-market stocks | VNQ | U.S. real estate (REITs) |
| TLT | Long U.S. Treasuries (20+ yr) | | |

The sample starts in 2005 because GLD, the youngest ETF, started trading in November 2004. An emerging-market local-currency bond ETF (EMLC) was left out because it only exists since 2010, and including it would have removed the 2008 crisis from the sample.

## Notebooks

Run them in order: each part uses `returns.csv` from Part 1.

### Part 1: Data pipeline and exploratory analysis

[`portfolio_data_pipeline.ipynb`](portfolio_data_pipeline.ipynb) downloads adjusted prices, builds daily log returns and studies their properties before any optimization:

- Annualized return, volatility, skewness and excess kurtosis, with a robustness check that excludes the 2008 crisis and the COVID crash.
- Correlations under stress: how co-movements changed during both crises compared with normal times.
- Rolling stock–bond correlation (SPY vs TLT) and the regime change after the 2022 inflation shock.

### Part 2: Minimum-variance portfolio

[`portfolio_min_variance.ipynb`](portfolio_min_variance.ipynb) estimates $\Sigma$ on the previous 3–4 years of data, computes the minimum-variance weights

$$\hat{\mathbf a} = \frac{\hat\Sigma^{-1}\boldsymbol\iota}{\boldsymbol\iota^\top\hat\Sigma^{-1}\boldsymbol\iota}$$

and tests them on the following year (2008, 2020, 2022 and 2025), including the borrowing cost of the short positions.

### Part 3: Rolling-window backtest

[`portfolio_rolling_backtest.ipynb`](portfolio_rolling_backtest.ipynb) re-estimates $\Sigma$ every month on the last 3 years of data and holds the new weights for the next month, giving one continuous out-of-sample record from February 2008 to October 2026 (225 monthly rebalances). It also analyzes the results by interest-rate regime, how the weights move, turnover and trading costs, a no-trade band that only rebalances when the weights change enough, and a leveraged version scaled to 1/N's volatility.

### Part 4: Long-only minimum variance

[`portfolio_long_only.ipynb`](portfolio_long_only.ipynb) adds the constraint that every weight must be between 0 and 1 (no short sales). Since there is no closed-form solution, the weights are found with a numerical optimizer (`scipy.optimize.minimize`, SLSQP). It reruns the rolling backtest of Part 3 and compares long-only minimum variance with the unconstrained version and 1/N: performance, how concentrated the weights are, and turnover.

## Key results

Rolling backtest, February 2008 – October 2026:

| | Minimum variance | Long-only min variance | Leveraged min variance | 1/N |
|---|---|---|---|---|
| Annual return | 3.5% | 3.7% | 7.2% | 5.7% |
| Annual volatility | 4.5% | 6.2% | 15.2% | 12.9% |
| Sharpe ratio | 0.48 | 0.41 | 0.46 | 0.40 |
| Maximum drawdown | −13.9% | −20.4% | −46.8% | −37.2% |
| Growth of $1 | $1.91 | $2.01 | $3.87 | $2.88 |

Sharpe ratios use the 3-month T-bill rate, which averaged 1.4% over the backtest. The leveraged version targets 1/N's volatility each month and pays the T-bill rate on the borrowed money.

**These results exclude two real costs:** the spread that investors pay above the T-bill rate when they borrow, and the borrow fee on the short positions. Both matter, because the leveraged version borrows on average 2.2 times its capital and is short 1.8 times its capital:

| Leveraged min variance | Annual return | Sharpe ratio | Growth of $1 |
|---|---|---|---|
| T-bill rate only (table above) | 7.2% | 0.46 | $3.87 |
| + 0.5% spread over T-bills | 6.2% | 0.39 | $3.16 |
| + 0.5% spread and 0.5% borrow fee on shorts | 5.3% | 0.33 | $2.67 |
| + 1% spread and 0.5% borrow fee on shorts | 4.2% | 0.26 | $2.18 |

With realistic costs, the leveraged version no longer beats 1/N (5.7% return, Sharpe 0.40). Even unleveraged minimum variance, with a 0.5% borrow fee on its shorts, falls to a Sharpe ratio of 0.41, practically tied with 1/N.

Sharpe ratio by interest-rate regime:

| | Minimum variance | Long-only min variance | Leveraged min variance | 1/N |
|---|---|---|---|---|
| 2008–2021 (falling and near-zero rates) | 0.79 | 0.75 | 0.77 | 0.49 |
| 2022–2026 (inflation and rising rates) | −0.24 | −0.37 | −0.41 | 0.08 |

- **Lower risk in every period.** Minimum variance had about a third of the volatility of 1/N and a much smaller worst fall, and it protected especially well in the 2008 crisis and the COVID crash.
- **Its efficiency depends on the regime.** Over the full sample it had a higher Sharpe ratio than 1/N, but that advantage came entirely from 2008–2021. In 2022–2026, with rising rates and a positive stock–bond correlation, it earned less than T-bills. A backtest starting in 2019 would have reached the opposite conclusion.
- **It behaves like a leveraged Treasury position:** about 117% long IEF financed by about −46% in TLT on average, with total short positions of 45%–81% of capital. It hedges growth shocks (2008, COVID) but not inflation shocks (2022).
- **Leverage magnifies the regime, and costs erase its edge.** Leveraged to 1/N's volatility and financed at the T-bill rate, minimum variance earned more over the full sample (7.2% vs 5.7% per year), but with drawdowns of −40% in 2008 and −47% in 2021–2022, and it lost money in 2022–2026. With a realistic borrowing spread and borrow fee on the shorts, its advantage over 1/N disappears. This is the risk-parity argument, with both of its sides.
- **Without shorts, risk rises but return doesn't fall.** The long-only version holds about 4 assets, anchored on IEF (63% on average). Without the TLT short to hedge interest-rate risk, it lost twice as much in 2022 (−16.8% vs −8.2%).
- **Trading costs are negligible** with liquid ETFs (below 0.1% per year). Turnover spikes when a crisis enters or leaves the 3-year estimation window. A no-trade band cuts the number of trades by three quarters without changing the risk, but saves very little.
- **Single test years can mislead.** In Part 2, out-of-sample volatility was 2.7 to 3.3 times the in-sample estimate in 2008, 2020 and 2022, while 2025 looked unusually good.

## Data

| File | Content |
|---|---|
| `prices.csv` | Adjusted daily closing prices (dividends and splits), from Yahoo Finance via `yfinance` |
| `returns.csv` | Daily log returns |
| `risk_free.csv` | 3-month U.S. T-bill yield (`^IRX`, annualized %), used for Sharpe ratios and financing costs in Parts 3 and 4 |

Part 1 loads `prices.csv` if it exists, so results are reproducible without downloading the data again. To change the sample, edit `START`/`END` in Part 1, delete `prices.csv` and `risk_free.csv`, and rerun the notebooks in order.

## Setup

```
pip install numpy pandas scipy matplotlib yfinance plotly nbformat
```

## Next steps

- A maximum weight per asset (for example 40%) in the long-only version, to force more diversification.
- Build realistic borrowing costs (spread and borrow fee) into the backtests themselves, not only as a sensitivity check.
- Alternative covariance estimators (EWMA, Ledoit-Wolf shrinkage) and window lengths.
- Constrained mean-variance portfolios, which also use expected returns.
- Assets that hedge inflation shocks, such as T-bills or TIPS.

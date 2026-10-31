# Portfolio Allocation Project

Multi-asset portfolio allocation using 10 ETFs (U.S. and international stocks, Treasuries, corporate and EM bonds, gold and REITs), with daily data from January 2016 to October 2026.

## Part 1: Data pipeline and exploratory analysis

[`portfolio_data_pipeline.ipynb`](portfolio_data_pipeline.ipynb) downloads adjusted prices, builds daily log returns and studies their properties before any optimization:

- Annualized return, volatility, skewness and excess kurtosis, with a robustness check that excludes the COVID crash.
- Correlations under stress: how co-movements changed during the COVID crash.
- Rolling stock–bond correlation (SPY vs TLT) and the regime change after the 2022 inflation shock.

## Data

| File | Content |
|---|---|
| `prices.csv` | Adjusted daily closing prices (dividends and splits), from Yahoo Finance via `yfinance` |
| `returns.csv` | Daily log returns |

The notebook loads `prices.csv` if it exists, so results are reproducible without downloading the data again.

## Setup

```
pip install numpy pandas matplotlib yfinance plotly nbformat
```

## Next steps

Minimum-variance and constrained mean-variance portfolios, with time-varying covariance estimates (rolling window and EWMA) and an out-of-sample backtest with monthly rebalancing.

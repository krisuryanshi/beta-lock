# BetaLock 📊  
**A quantitative portfolio construction algorithm that builds a diversified equity portfolio designed to track a blended S&P 500/TSX benchmark while locking beta to ~1, using constrained optimization (SciPy SLSQP).**

> 🥈 Placed 2nd in the Market Meet category of the CFM 101 Robo-Advising Challenge. Full implementation and outputs are in `main.ipynb`.

## Overview
BetaLock constructs a diversified 20-stock equity portfolio engineered to track a benchmark defined as the average of S&P 500 and TSX Composite total returns. Rather than trying to beat the market, it solves an index-tracking problem: minimizing tracking error variance against the benchmark while holding portfolio beta close to 1, all subject to sector, market-cap, and position-size constraints.

All logic is implemented in code and follows the challenge constraints throughout. The final result is a fee-adjusted portfolio that can be executed using fractional shares.

---

## Strategy Summary
The algorithm reads an input universe from `tickers.csv` and follows this process:

- Filter eligible stocks based on data quality and liquidity  
- Construct a benchmark from S&P 500 and TSX Composite total returns  
- Rank stocks by average distance from the benchmark  
- Select and optimize a 20-stock portfolio under sector and market-cap constraints  
- Execute the portfolio with transaction fees and fractional shares  

---

## Ticker Filtering
Stocks are filtered using these rules:

- Valid historical data from October 1, 2024 to September 30, 2025  
- Average daily volume of at least 5,000 shares  
- Months with fewer than 18 trading days excluded  
- Duplicate and invalid tickers removed  

---

## Analysis Window
A three-year window is used:

- November 21, 2022 to November 21, 2025  

This captures recent market behavior while avoiding early COVID-period distortions and includes higher volatility in 2024–2025. All data pulls are pinned to a fixed as-of date (November 21, 2025) so results are fully reproducible across runs.

---

## Benchmark and Stock Ranking
The benchmark is defined as the average of cumulative total returns from the S&P 500 and TSX Composite.

Each stock is ranked by its average absolute distance from the benchmark's cumulative return. Cross-listed duplicates are removed, keeping the version that tracks the benchmark more closely.

---

## Portfolio Construction and Optimization
A portfolio of 20 stocks is selected with the following constraints:

- At least one large-cap stock (market cap > $10B CAD)  
- At least one small-cap stock (market cap < $2B CAD)  
- No sector above 40 percent exposure  
- Maximum weight of 15 percent per stock  

Weights are optimized using SciPy's SLSQP solver by minimizing tracking error variance, with a beta penalty that keeps portfolio beta close to 1. Initial weights favor stocks that track the benchmark more closely, giving the solver a strong starting point.

---

## Fee-Adjusted Execution
The optimized portfolio is executed using a $1,000,000 CAD budget.

- USD stocks are converted using the USD/CAD exchange rate as of the pinned date  
- Fees applied as $2.15 USD flat or $0.001 USD per share, whichever is smaller  
- Portfolio weights are rescaled after fees  
- Fractional shares are used to deploy capital efficiently  

---

## Final Output
The final portfolio is exported to `portfolio.csv`, containing each ticker and the number of shares purchased. Final weights sum to 100 percent, with total value as close as possible to $1,000,000 CAD after fees.

![Portfolio vs. Benchmark](INSERT_IMAGE_PATH_HERE)

### Results
The optimized portfolio tracks the blended benchmark closely across the full three-year window:

- **Portfolio beta: ~1.00** — the portfolio moves in near-lockstep with the benchmark's direction and magnitude, which was the core objective of the beta penalty term  
- **Annualized tracking error: ~8.9%** — the typical yearly deviation between portfolio and benchmark returns, expected for a concentrated 20-stock basket under sector and weight constraints rather than full index replication  
- **Tracking error variance: ~3.2e-05 daily** (~47% of the benchmark's own daily variance)  

The tight fit in the chart above reflects the beta-locking objective: the portfolio isn't just close in level, it moves with the benchmark day to day. Tracking error is higher than a full-replication index fund (which holds hundreds of names) by design — this is an optimized-sampling approach that matches the benchmark using only 20 stocks.

---

## Repository Structure
```
beta-lock/
│
├── main.ipynb          # Main analysis and portfolio construction notebook
├── tickers.csv         # Input ticker universe
├── portfolio.csv       # Final output (Ticker, Shares)
├── requirements.txt    # Required Python libraries
└── README.md           # Project documentation
```
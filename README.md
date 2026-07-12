# BetaLock 📊  

> 🥈 2nd place, Market Meet category, CFM 101 Robo-Advising Challenge. Full code in `main.ipynb`.

BetaLock builds a diversified 20-stock portfolio that tracks a benchmark defined as the average of S&P 500 and TSX Composite total returns. Rather than trying to beat the market, it solves an index-tracking problem: minimizing tracking error against the benchmark while holding portfolio beta close to 1, subject to sector, market-cap, and weight constraints. The result is a fee-adjusted portfolio executable using fractional shares.

---

## Ticker Filtering
Stocks must pass these screens to enter the universe:

- Valid historical data from October 1, 2024 to September 30, 2025  
- Average daily volume of at least 5,000 shares (months with fewer than 18 trading days excluded)  
- Duplicate and invalid tickers removed  

---

## Analysis Window
A three-year window from November 21, 2022 to November 21, 2025. This gives the most recent market behavior while avoiding the early COVID period, and it captures the volatility that followed Trump returning to office in early 2025, especially the tariffs introduced early on, which moved the market around quite a bit. All data pulls are pinned to a fixed as-of date (November 21, 2025) so results are fully reproducible across runs.

---

## Benchmark and Stock Ranking
The benchmark is the average of cumulative total returns from the S&P 500 and TSX Composite. Each stock is ranked by its average absolute distance from the benchmark's cumulative return, and cross-listed duplicates are removed, keeping the version that tracks the benchmark more closely.

---

## Portfolio Construction and Optimization
A 20-stock portfolio is selected under these constraints:

- At least one large-cap stock (market cap > $10B CAD)  
- At least one small-cap stock (market cap < $2B CAD)  
- No sector above 40% exposure  
- Each stock weighted between 2.5% and 15%  

Weights are optimized using SciPy's SLSQP solver by minimizing tracking error variance, with a beta penalty that keeps portfolio beta close to 1. Initial weights favor stocks that track the benchmark more closely.

---

## Fee-Adjusted Execution
The optimized portfolio is executed on a $1,000,000 CAD budget:

- USD stocks converted using the USD/CAD exchange rate as of the pinned date  
- Fees applied as $2.15 USD flat or $0.001 USD per share, whichever is smaller  
- Weights rescaled after fees  
- Fractional shares used to deploy capital efficiently  

---

## Final Output
The final portfolio is exported to `portfolio.csv` (ticker and shares purchased). Final weights sum to 100%, with total value as close as possible to $1,000,000 CAD after fees.

![Portfolio vs. Benchmark](INSERT_IMAGE_PATH_HERE)

### Results
- **Portfolio beta: ~1.00**, moving in near-lockstep with the benchmark's direction and magnitude, the core objective of the beta penalty  
- **Annualized tracking error: ~8.9%**, expected for a concentrated 20-stock basket under sector and weight constraints rather than a full index fund  
- **Tracking error variance: ~3.2e-05 daily** (~47% of the benchmark's own daily variance)  

Tracking error is higher than a full index fund by design, since BetaLock tracks the benchmark with just 20 stocks instead of holding every stock in it.

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

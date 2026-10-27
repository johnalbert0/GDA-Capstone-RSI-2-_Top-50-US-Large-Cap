# RSI(2) Mean-Reversion Strategy & Prop Firm Risk Analytics Capstone

## 📊 Project Overview
This repository contains the end-to-end data analytics pipeline and quantitative backtest for an **RSI(2) Mean-Reversion Trading Strategy** targeting large-cap U.S. equities ($Price > 100\text{ EMA}$, $RSI < 10$ entry, $RSI > 90$ exit). 

Developed as a capstone project for the Google Data Analytics Professional Certificate, this analysis evaluates strategy performance against strict proprietary trading firm (FTMO) risk parameters (maximum daily and total drawdown constraints, position sizing, and risk-to-reward profiles).

---
```text
rsi2-trading-capstone/
│
├── data/
│   ├── raw_tickers/               # Individual CSV backtest results for 50 US large-cap stocks
│   └── master_stock_dataset.csv   # Consolidated master dataset (125k+ rows with standardized schema)
│
├── scripts/
│   ├── merge_data.py              # Python automation script (glob, pandas, regex) for ETL
│   └── strategy_analysis.py       # DuckDB & Pandas script for expectancy and performance modeling
│
├── sql/
│   └── capstone_queries.sql       # SQL queries computing win rates, profit factors, and net P&L
│
└── README.md                      # Project documentation and key findings

```

⚙️ Data Pipeline & ETL (Python & Pandas)
Extraction & Standardization: Automated ingestion of 50 individual ticker backtest CSVs. Standardized headers to lowercase and injected ticker symbols as primary identifiers.

Consolidation: Merged disparate time-series files into a clean master_stock_dataset.csv optimized for analytical querying.

🔍 Analytical Findings & SQL Insights
Using DuckDB SQL, the master dataset was queried to isolate positive-expectancy assets.

Key Discoveries:
Asset Selectivity: Out of 50 large-cap tickers tested, only 24 showed positive expectancy, emphasizing that mean-reversion rules require rigorous asset filtering to clear prop firm hurdles.

Sector Clustering: High-alpha tech components demonstrated superior recovery characteristics following extreme oversold conditions (RSI < 10).

Sample SQL Query Used for Performance Breakdown:
```text
SELECT 
    ticker,
    COUNT(*) AS total_trades,
    ROUND(SUM(CASE WHEN net_profit > 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS win_rate_pct,
    ROUND(
        SUM(CASE WHEN net_profit > 0 THEN net_profit ELSE 0 END) / 
        NULLIF(ABS(SUM(CASE WHEN net_profit < 0 THEN net_profit ELSE 0 END)), 0), 2
    ) AS profit_factor,
    ROUND(SUM(net_profit), 2) AS total_net_profit
FROM 'master_stock_dataset.csv'
GROUP BY ticker
HAVING COUNT(*) >= 10
ORDER BY total_net_profit DESC;
```
🚀 Tech Stack
Language: Python (pandas, duckdb, glob, re)

Database & Querying: DuckDB (Embedded SQL)

Environment: VS Code, Git & GitHub (Version Control)

## Strategy Methodology: RSI(2) Mean-Reversion

### Entry Criteria (Long Only)
* **Trend Filter:** Asset price must be trading above its 100-day Exponential Moving Average (EMA) to ensure a macro-uptrend context.
* **Oversold Trigger:** 2-period RSI drops below **10** (extreme short-term pullback).

### Exit & Risk Management Criteria
* **Exit Signal:** 2-period RSI crosses back above **90** (mean reversion achieved).
* **Stop Loss:** Hard cut-loss threshold set at **6%** below entry price to manage downside tail risk and comply with strict prop-firm risk parameters.

* ## 🚀 Getting Started & Repository Structure

### Prerequisites
Ensure you have Python installed along with the required analytical libraries:
```bash
pip install duckdb pandas
```

Running the Analysis
You can execute the analytical queries directly via DuckDB in your Jupyter environment or local terminal:
import duckdb

```
# Example analytical query inspecting top performers
query_results = duckdb.sql("""
    SELECT ticker, COUNT(*) AS total_trades, SUM("net pnl usd") AS total_net_profit
    FROM 'master_stock_dataset.csv'
    GROUP BY ticker
    HAVING COUNT(*) >= 10 AND SUM("net pnl usd") > 0
    ORDER BY total_net_profit DESC;
""").df()
```

display(query_results)

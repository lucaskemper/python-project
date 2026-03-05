# Pairs Trading Strategy

![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![License](https://img.shields.io/badge/License-Educational-green)
![Status](https://img.shields.io/badge/Status-In%20Development-yellow)
![WRDS](https://img.shields.io/badge/Data-WRDS%2FCRSP-orange)

A Python implementation of a **cointegration-based pairs trading strategy** on US equities using WRDS/CRSP daily data. This mean-reverting strategy exploits statistical relationships between cointegrated stock pairs.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Roadmap](#roadmap)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Methodology](#methodology)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Performance Metrics](#performance-metrics)
- [Architecture](#architecture)
- [References](#references)

---

## Overview

This project implements a systematic pairs trading strategy that:
- Identifies cointegrated stock pairs from a broad US equity universe
- Takes long-short positions when spreads deviate from their mean
- Profits from mean reversion while maintaining dollar-neutral exposure
- Evaluates performance against an equal-weighted CRSP benchmark

### Project Objectives

| Goal | Target |
|------|--------|
| Traded pairs | 20-50 cointegrated pairs |
| Data coverage | 2000-2024 daily data |
| Benchmark | Equal-weighted CRSP universe |

---

## Features

- **Two-Stage Pair Selection**: Correlation pre-filter + cointegration testing
- **Robust Parameter Selection**: Grid search over validation period
- **Dollar-Neutral Positioning**: Market-neutral exposure within each pair
- **Annual Rebalancing**: Refresh pair universe yearly to adapt to market changes
- **Comprehensive Backtesting**: Full P&L attribution and performance analytics
- **Risk Management**: Stop-loss and time-based exit rules

---

## Roadmap

### Phase 1: Foundation (Week 1-2)
- [ ] Set up project structure and dependencies
- [ ] Implement WRDS/CRSP data loader
- [ ] Build data cleaning pipeline
- [ ] Exploratory data analysis (EDA)
- [ ] Correlation pre-filter implementation

### Phase 2: Pair Selection (Week 2-3)
- [ ] Implement cointegration testing (ADF/Engle-Granger)
- [ ] Hedge ratio estimation via OLS
- [ ] Pair ranking and selection logic
- [ ] Final universe of 20-50 pairs

### Phase 3: Trading Engine (Week 3-4)
- [ ] Spread construction module
- [ ] Rolling z-score calculation
- [ ] Entry/exit signal generation
- [ ] Position sizing logic
- [ ] Daily P&L calculation

### Phase 4: Validation & Optimization (Week 4-5)
- [ ] Parameter grid search (2016-2022)
- [ ] Robustness testing
- [ ] Stop-loss calibration
- [ ] Transaction cost modeling
- [ ] Select final parameters

### Phase 5: Out-of-Sample Testing (Week 5-6)
- [ ] Run locked-parameter backtest (2023-2024)
- [ ] Benchmark comparison
- [ ] Regime analysis
- [ ] Performance attribution

### Phase 6: Documentation & Delivery (Week 6)
- [ ] Final notebook polish
- [ ] Performance report
- [ ] Code documentation
- [ ] Presentation materials

---

## Installation

### Prerequisites

- Python 3.8 or higher
- WRDS account with CRSP access

### Setup

```bash
# Clone the repository
git clone https://github.com/yourusername/pairs-trading.git
cd pairs-trading

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure WRDS credentials
wrds -u your_username  # Follow prompts to set up API key
```

### Dependencies

```
pandas>=1.5.0
numpy>=1.21.0
wrds>=3.1.0
statsmodels>=0.13.0
scipy>=1.7.0
matplotlib>=3.5.0
seaborn>=0.11.0
jupyter>=1.0.0
```

---

## Quick Start

```python
from src.data_cleaning import load_crsp_data, clean_data
from src.pair_selection import select_pairs
from src.trading import backtest_pairs_strategy
from src.evaluation import evaluate_performance

# 1. Load and clean data
prices_df, returns_df = load_crsp_data(start='2000-01-01', end='2024-12-31')
prices_df = clean_data(prices_df, min_history=504, min_coverage=0.95)

# 2. Select cointegrated pairs (using training period)
pairs = select_pairs(
    prices_df['2000':'2015'],
    n_pairs=30,
    correlation_threshold=0.7
)

# 3. Run backtest with chosen parameters
results = backtest_pairs_strategy(
    prices_df,
    pairs,
    entry_threshold=2.0,
    stop_loss=3.0,
    max_holding=40,
    start_date='2016-01-01',
    end_date='2024-12-31'
)

# 4. Evaluate performance
metrics = evaluate_performance(
    results['strategy_returns'],
    results['benchmark_returns']
)

print(f"Sharpe Ratio: {metrics['sharpe']:.2f}")
print(f"Max Drawdown: {metrics['max_drawdown']:.2%}")
```

---

## Methodology

### Data Source

| Attribute | Value |
|-----------|-------|
| Database | WRDS-CRSP daily stock file |
| Securities | Ordinary common shares (share codes 10, 11) |
| Exchanges | NYSE, AMEX, NASDAQ |
| Time span | 2000-2024 daily data |

### Sample Structure

| Period | Years | Purpose |
|--------|-------|---------|
| Training | 2000-2015 | Universe cleaning, pair selection, hedge ratio estimation |
| Validation | 2016-2022 | Parameter robustness testing |
| Out-of-Sample | 2023-2024 | Final performance evaluation |

### Data Cleaning

- Keep only ordinary common shares on major exchanges
- Require minimum 2 years (~504 trading days) of price history
- Require at least 95% non-NaN observations
- Track and handle delistings appropriately

### Pair Selection

```
┌─────────────────────────────────────────────────────┐
│                  CORRELATION PRE-FILTER              │
│   Screen pairs with return correlation > threshold   │
└─────────────────────┬───────────────────────────────┘
                      │ Top 500-1000 candidates
                      ▼
┌─────────────────────────────────────────────────────┐
│               COINTEGRATION TESTING                  │
│   ADF test on log-price spread: p-value < 0.05       │
└─────────────────────┬───────────────────────────────┘
                      │ Cointegrated pairs
                      ▼
┌─────────────────────────────────────────────────────┐
│                  RANKING & SELECTION                 │
│   Select top 20-50 by ADF p-value + correlation      │
└─────────────────────────────────────────────────────┘
```

### Spread Construction

```python
# Log prices
log_A = np.log(price_A)
log_B = np.log(price_B)

# Hedge ratio via OLS
beta = OLS(log_A, log_B).fit().params[0]

# Spread
spread = log_A - beta * log_B

# Z-score normalization
z_score = (spread - spread.rolling(60).mean()) / spread.rolling(60).std()
```

### Trading Rules

| Parameter | Values to Test | Description |
|-----------|----------------|-------------|
| Entry threshold | 1.5, 2.0, 2.5 σ | Z-score level to enter position |
| Stop-loss | 3.0, 3.5 σ | Z-score level to exit with loss |
| Max holding | 20, 40, 60 days | Maximum time in position |

**Entry Signals:**
| Signal | Condition | Action |
|--------|-----------|--------|
| Short spread | z > entry_threshold | Short A, Long B |
| Long spread | z < -entry_threshold | Long A, Short B |

**Exit Signals:**
- **Mean reversion**: z-score crosses back through 0
- **Stop-loss**: |z| exceeds stop-loss threshold
- **Time-based**: holding period exceeds max days

### Capital Allocation

```
Total Capital: $C
Active Positions: N
Capital per Pair: $C / N

Within each pair (dollar-neutral):
  Long leg:  $C_pair / 2
  Short leg: $C_pair / 2 × beta
```

---

## Project Structure

```
pairs-trading/
├── README.md
├── requirements.txt
├── config/
│   └── config.yaml           # Strategy parameters
├── data/
│   ├── raw/                  # Downloaded CRSP data
│   └── processed/            # Cleaned datasets
├── notebooks/
│   ├── 01_eda.ipynb          # Exploratory analysis
│   ├── 02_pair_selection.ipynb
│   ├── 03_backtest.ipynb
│   └── 04_results.ipynb      # Final results
├── src/
│   ├── __init__.py
│   ├── data_cleaning.py      # Load & clean functions
│   ├── pair_selection.py     # Correlation & cointegration
│   ├── trading.py            # Backtest engine
│   ├── evaluation.py         # Performance metrics
│   └── utils.py              # Helper functions
├── tests/
│   ├── test_pair_selection.py
│   └── test_trading.py
└── reports/
    ├── final_report.pdf
    └── figures/
```

---

## Configuration

Edit `config/config.yaml` to customize strategy parameters:

```yaml
data:
  start_date: '2000-01-01'
  end_date: '2024-12-31'
  share_codes: [10, 11]
  min_history_days: 504
  min_coverage: 0.95

pair_selection:
  correlation_threshold: 0.7
  cointegration_pvalue: 0.05
  n_pairs: 30
  correlation_window: 252

trading:
  entry_threshold: 2.0
  stop_loss: 3.0
  max_holding_days: 40
  zscore_window: 60
  transaction_cost_bps: 5

periods:
  training_end: '2015-12-31'
  validation_end: '2022-12-31'
```

---

## Performance Metrics

The strategy evaluates multiple dimensions of performance:

### Return Metrics
| Metric | Description |
|--------|-------------|
| Annualized Return | Geometric mean annual return |
| Cumulative Return | Total return over period |
| Monthly Returns | Time series of monthly P&L |

### Risk Metrics
| Metric | Description |
|--------|-------------|
| Annualized Volatility | Standard deviation of daily returns × √252 |
| Maximum Drawdown | Largest peak-to-trough decline |
| Value at Risk (VaR) | 5th percentile daily loss |

### Risk-Adjusted Metrics
| Metric | Formula |
|--------|---------|
| Sharpe Ratio | (Return - Rf) / Volatility |
| Sortino Ratio | (Return - Rf) / Downside Volatility |
| Calmar Ratio | Return / Max Drawdown |

### Trade Statistics
| Metric | Description |
|--------|-------------|
| Total Trades | Number of round-trip trades |
| Hit Ratio | Percentage of profitable trades |
| Average P&L | Mean profit per trade |
| Average Holding | Mean days in position |

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        DATA LAYER                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │   WRDS/CRSP │→ │   Cleaner   │→ │   Prices &  │           │
│  │   Raw Data  │  │   Pipeline  │  │   Returns   │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                    SELECTION LAYER                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │ Correlation │→ │Cointegration│→ │   Pair      │           │
│  │  Pre-filter │  │    Tests    │  │  Selection  │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                     TRADING LAYER                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │   Spread &  │→ │   Signal    │→ │  Position   │           │
│  │   Z-Score   │  │  Generator  │  │   Manager   │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                   EVALUATION LAYER                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │    P&L      │→ │ Performance │→ │   Report    │           │
│  │  Calculator │  │   Metrics   │  │  Generator  │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
└──────────────────────────────────────────────────────────────┘
```

---

## References

### Documentation
- [WRDS Documentation](https://wrds-www.wharton.upenn.edu/)
- [Statsmodels Time Series](https://www.statsmodels.org/stable/tsa.html)

### Academic Resources
- Engle, R.F. & Granger, C.W. (1987). "Co-integration and Error Correction"
- Gatev, E., Goetzmann, W.N., & Rouwenhorst, K.G. (2006). "Pairs Trading: Performance of a Relative-Value Arbitrage Rule"

### Code Examples
- [Pairs Trading Notebook](https://github.com/rubetron/Pairs_trading)
- [Statsmodels Cointegration Examples](https://www.statsmodels.org/stable/examples/notebooks/generated/cointegration.html)


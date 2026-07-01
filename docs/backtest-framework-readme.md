# Backtest Framework

A modular, multi-market backtesting framework integrated into the Vibe Trading AI project. Supports equities, crypto, forex, futures, and options portfolios with data fetching from 18+ registered sources.

[![Python](https://img.shields.io/badge/python-3.11%2B-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-active-success.svg)]()

---

## Table of Contents

- [Backtest Framework](#backtest-framework)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Features](#features)
  - [Architecture](#architecture)
    - [Engine Inheritance](#engine-inheritance)
  - [Quick Start](#quick-start)
  - [Configuration](#configuration)
    - [Required Fields](#required-fields)
    - [Optional Fields](#optional-fields)
    - [Validation Config](#validation-config)
  - [Data Sources](#data-sources)
    - [Supported Sources](#supported-sources)
    - [Auto Routing](#auto-routing)
  - [Market Engines](#market-engines)
  - [Signal Engine](#signal-engine)
    - [Signal Interface](#signal-interface)
    - [Data Available in `data_map`](#data-available-in-data_map)
  - [Validation](#validation)
    - [Monte Carlo Permutation Test](#monte-carlo-permutation-test)
    - [Bootstrap Sharpe Confidence Interval](#bootstrap-sharpe-confidence-interval)
    - [Walk-Forward Analysis](#walk-forward-analysis)
  - [Metrics \& Artifacts](#metrics--artifacts)
    - [Available Metrics](#available-metrics)
  - [Trust Layer Run Cards](#trust-layer-run-cards)
  - [Project Structure](#project-structure)
  - [Contributing](#contributing)
  - [License](#license)

---

## Overview

This framework provides an event-driven backtesting engine for evaluating trading strategies against historical market data. It supports **8 distinct market engines** (China A-shares, Crypto, Global Equity, Forex, China Futures, Global Futures, Options Portfolios, and Composite) with automatic data fetching from **18+ registered data sources** plus SEC EDGAR filings and RSSHub event enrichment.

The framework is designed to:

- Prevent look-ahead bias through event-driven bar-by-bar data replay
- Validate strategy code via AST-based safety checks before execution
- Enforce schema correctness with immutable dataclasses (Python stdlib `dataclass`)
- Auto-route symbols to the correct market engine by detecting market type from the symbol pattern
- Provide comprehensive performance analytics and statistical validation

## Features

- **8 Market Engines** — China A-shares, Crypto, Global Equity, Forex, China Futures, Global Futures, Options Portfolio, and Composite (cross-market)
- **Event-driven architecture** — sequential bar-by-bar replay prevents accidental use of future information
- **Dataclass models** — `Position`, `TradeRecord`, and `EquitySnapshot` are frozen dataclasses (no external Pydantic dependency for data models)
- **AST-based strategy safety** — signal engine source is statically analyzed before import (no decorators, no non-literal defaults, no top-level executable statements, no unsafe annotations)
- **Dynamic engine routing** — `BaseEngine` subclasses are selected at runtime based on symbol patterns and data source
- **18+ Data Sources** — 18 registered loaders (Tushare, yfinance, OKX, AKShare, CCXT, Futu, MootDX, Alpha Vantage, Finnhub, FMP, BaoStock, EastMoney, Sina, Stooq, Tiingo, Tencent, Yahoo, local) plus SEC EDGAR filings and RSSHub event enrichment.
- **Runtime fallback chains** — when a primary data source returns empty, the runner transparently tries the next source in the fallback chain
- **Fundamental data enrichment** — optional Tushare statement fields (balance sheet, income statement, cash flow) enriched onto price frames before signal generation
- **Event feed enrichment** — RSSHub-based alternative data feeds (news, events) enriched onto price frames with point-in-time safety
- **Portfolio optimization** — 4 built-in optimizers: mean-variance, risk parity, equal volatility, max diversification
- **Statistical validation** — Monte Carlo permutation test, Bootstrap Sharpe CI, Walk-Forward analysis
- **Performance metrics** — Sharpe, Sortino, max drawdown (and duration), CAGR, Calmar, win rate, profit factor, information ratio, benchmark comparison
- **JSON configuration** — reproducible runs via `config.json`
- **Trust Layer run cards** — JSON + Markdown artifacts recording config hash, strategy hash, metrics, and all artifact file hashes for full reproducibility

## Architecture

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────┐     ┌──────────────┐
│  DataLoader   │ --> │  SignalEngine     │ --> │  BaseEngine     │ --> │  Metrics /   │
│  (fetch data) │     │  (generate()     │     │  (bar-by-bar    │     │  Artifacts   │
│               │     │   returns signals)│     │   execution)    │     │              │
└──────────────┘     └──────────────────┘     └─────────────────┘     └──────────────┘
                            │                          │
                            v                          v
                    ┌──────────────────┐     ┌──────────────────────┐
                    │  Optimizer (opt) │     │  Validation (MC, BS, │
                    │  (optional)      │     │  WF statistical)     │
                    └──────────────────┘     └──────────────────────┘
```

The runner (`backtest/runner.py`) orchestrates the pipeline:

1. Load `config.json` and `code/signal_engine.py` from a run directory
2. AST-validate the signal engine source (rejects decorators, non-literal defaults, top-level statements, unsafe annotations)
3. Fetch market data via a selected or auto-routed data loader (with fallback chains)
4. Enrich with optional fundamental / event feed data
5. Run bar-by-bar backtest through the appropriate market engine (selected by symbol pattern)
6. Write CSV artifacts (equity, trades, metrics), validation results, and run cards

### Engine Inheritance

```
BaseEngine (ABC — bar-by-bar loop, position management, assetion)
├── ChinaAEngine        # A-shares (T+1, no short, price limits)
├── GlobalEquityEngine  # US / HK equities
├── CryptoEngine        # Crypto perpetuals (funding fees, liquidation)
├── ForexEngine         # FX spot/CFD (spread, swap, high leverage)
├── ChinaFuturesEngine  # China commodity/financial futures
├── GlobalFuturesEngine # International futures (CME/ICE/Eurex)
├── CompositeEngine     # Cross-market shared capital pool (delegates to sub-engines)
└── FuturesBaseEngine   # Intermediate layer (contract multiplier logic)
    ├── ChinaFuturesEngine
    └── GlobalFuturesEngine
```

See `agent/backtest/engines/__init__.py` for the full engine map.

## Quick Start

```bash
# Inside the vibe-trading-ai project:
python -m backtest.runner <run_dir>
```

Where `run_dir` is a directory containing:

```
run_dir/
├── config.json          # Backtest configuration (required)
└── code/
    └── signal_engine.py  # Strategy signal engine (required)
```

Example `config.json`:

```json
{
  "codes": ["SPY", "AAPL"],
  "start_date": "2023-01-01",
  "end_date": "2024-12-31",
  "source": "auto",
  "interval": "1D",
  "engine": "daily",
  "initial_cash": 1000000
}
```

Example `signal_engine.py`:

```python
from typing import Dict

import pandas as pd


class SignalEngine:
    """Simple moving-average crossover strategy."""

    def __init__(self, fast=20, slow=50):
        self.fast = fast
        self.slow = slow

    def generate(self, data_map: Dict[str, pd.DataFrame]) -> Dict[str, pd.Series]:
        signals = {}
        for code, df in data_map.items():
            close = df["close"]
            ma_fast = close.rolling(self.fast).mean()
            ma_slow = close.rolling(self.slow).mean()
            signals[code] = (ma_fast > ma_slow).astype(float) * 2 - 1  # +1 or -1
            signals[code] = signals[code].shift(1).fillna(0)  # next-bar-open semantics
        return signals
```

Run:

```bash
python -m backtest.runner /path/to/run_dir
```

## Configuration

Backtest configuration lives in a single `config.json` file inside each run directory.

### Required Fields

| Field        | Type        | Default    | Description              |
| ------------ | ----------- | ---------- | ------------------------ |
| `codes`      | `List[str]` | (required) | Instrument codes/symbols |
| `start_date` | `str`       | (required) | Start date (YYYY-MM-DD)  |
| `end_date`   | `str`       | (required) | End date (YYYY-MM-DD)    |

### Optional Fields

| Field                | Type                   | Default     | Description                                                                               |
| -------------------- | ---------------------- | ----------- | ----------------------------------------------------------------------------------------- |
| `source`             | `str`                  | `"tushare"` | Data source. Use `"auto"` for cross-market auto-routing                                   |
| `interval`           | `str`                  | `"1D"`      | Bar size: `1m`, `5m`, `15m`, `30m`, `1H`, `4H`, `1D`                                      |
| `engine`             | `str`                  | `"daily"`   | Backtest engine type: `"daily"` or `"options"`                                            |
| `initial_cash`       | `float`                | `1_000_000` | Starting capital                                                                          |
| `leverage`           | `float`                | `1.0`       | Default leverage                                                                          |
| `benchmark`          | `str`                  | `"auto"`    | Benchmark ticker (or `"auto"` for implicit by market)                                     |
| `fundamental_fields` | `Dict[str, List[str]]` | —           | Tushare statement field enrichment (table → field list)                                   |
| `event_feeds`        | `List[Dict]`           | —           | RSSHub event feed enrichment definitions                                                  |
| `optimizer`          | `str`                  | —           | Optimizer name: `mean_variance`, `risk_parity`, `equal_volatility`, `max_diversification` |
| `optimizer_params`   | `dict`                 | `{}`        | Parameters passed to the optimizer                                                        |
| `validation`         | `dict`                 | —           | Statistical validation configuration (see below)                                          |

### Validation Config

```json
{
  "validation": {
    "monte_carlo": { "n_simulations": 1000, "seed": 42 },
    "bootstrap": { "n_bootstrap": 1000, "confidence": 0.95, "seed": 42 },
    "walk_forward": { "n_windows": 5 }
  }
}
```

This triggers three statistical checks on completed backtests:

- **Monte Carlo permutation test** — shuffles trade P&L order to test if observed Sharpe / max-drawdown is statistically significant (p-value < 0.05 means performance is not explainable by random ordering alone).
- **Bootstrap Sharpe CI** — resamples daily returns to estimate confidence interval on Sharpe ratio and probability that Sharpe > 0.
- **Walk-Forward analysis** — splits the backtest into sequential non-overlapping windows, evaluates metrics per window, and computes a consistency rate (fraction of windows that are profitable).

## Data Sources

The framework includes **18 registered data loaders**, registered in a global loader registry (`backtest/loaders/registry.py`). When `source="auto"`, the runner detects the market type from each symbol and routes it to the appropriate loader with fallback chains. Enrichment providers (SEC EDGAR filings, RSSHub events) can augment data after loading.

### Supported Sources

| Source             | Markets                        | Notes                                                       |
| ------------------ | ------------------------------ | ----------------------------------------------------------- |
| `tushare`          | China A-shares, Funds, Futures | Primary source for mainland China                           |
| `yahoo`            | US / HK equities               | Direct Yahoo chart API (no-auth, no yfinance dependency)    |
| `yfinance`         | US / HK equities               | Universal fallback via yfinance package                     |
| `okx`              | Crypto                         | Direct OKX exchange API                                     |
| `akshare`          | A-shares, Macro, Forex         | Broad Chinese market coverage                               |
| `ccxt`             | Crypto                         | Multi-exchange via CCXT library                             |
| `futu`             | HK / A-shares                  | Futu Securities API                                         |
| `mootdx`           | A-shares                       | MootDX unofficial API                                       |
| `alphavantage`     | Global equities                | Alpha Vantage API                                           |
| `finnhub`          | Global equities                | Finnhub API                                                 |
| `fmp`              | Global equities                | Financial Modeling Prep API                                 |
| `baostock`         | A-shares                       | BaoStock data                                               |
| `eastmoney`        | A-shares                       | EastMoney data                                              |
| `sina`             | A-shares                       | Sina Finance API                                            |
| `stooq`            | Global                         | Stooq data                                                  |
| `tiingo`           | US equities                    | Tiingo API                                                  |
| `tencent`          | A-shares / HK                  | Tencent finance data                                        |
| `sec_edgar_client` | US filings                     | SEC EDGAR filings enrichment (not a price data source)      |
| `local`            | Local files                    | Read from local CSV files                                   |
| `rsshub_events`    | Events                         | RSSHub event feed enrichment only (not a price data source) |

### Auto Routing

When `source="auto"` (recommended for multi-asset portfolios), each symbol is classified by its pattern and routes through an ordered fallback chain:

| Symbol Pattern            | Market      | Primary Source | Fallback Chain                                                                                |
| ------------------------- | ----------- | -------------- | --------------------------------------------------------------------------------------------- |
| `600519` (A-share ticker) | `a_share`   | tencent        | mootdx → eastmoney → baostock → akshare → tushare → local                                     |
| `SPY`, `AAPL` (US ticker) | `us_equity` | yahoo          | stooq → sina → eastmoney → yfinance → tiingo → fmp → finnhub → alphavantage → akshare → local |
| `0700.HK` (HK ticker)     | `hk_equity` | eastmoney      | yahoo → futu → yfinance → akshare → local                                                     |
| `BTC-USDT` (crypto pair)  | `crypto`    | okx            | ccxt → yfinance → local                                                                       |
| `EUR/USD` (forex pair)    | `forex`     | akshare        | —                                                                                             |
| `IF2406` (China futures)  | `futures`   | tushare        | akshare → local                                                                               |

## Market Engines

The framework includes 8 market-specific engine classes, all inheriting from `BaseEngine` (`backtest/engines/base.py`). The runner selects the appropriate engine automatically based on detected market type:

| Engine                | Module                 | Market Rules                                                                                                          |
| --------------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **China A-Shares**    | `ChinaAEngine`         | T+1 settlement, no short selling, price limits (±10% / ±20% / ±5%), minimum 100-share lots, stamp tax 0.05% sell-side |
| **Global Equity**     | `GlobalEquityEngine`   | US / HK equities, T+0, no short restrictions                                                                          |
| **Crypto**            | `CryptoEngine`         | Crypto perpetuals with funding fees, liquidation thresholds, 24/7 trading (365 trading days)                          |
| **Forex**             | `ForexEngine`          | FX spot/CFD with spread modeling, swap/rollover, high leverage                                                        |
| **China Futures**     | `ChinaFuturesEngine`   | CFFEX/SHFE/DCE/ZCE/INE contract multipliers, margin requirements                                                      |
| **Global Futures**    | `GlobalFuturesEngine`  | CME/ICE/Eurex contract multipliers                                                                                    |
| **Options Portfolio** | `run_options_backtest` | Black-Scholes pricing (European/American), IV smile, multi-leg strategies, Greeks                                     |
| **Composite**         | `CompositeEngine`      | Cross-market engine with a shared capital pool; delegates market-rule calls to sub-engines                            |

## Signal Engine

Strategies are implemented as a `SignalEngine` class inside `code/signal_engine.py`. The runner:

1. **AST-validates** the source file before importing it (rejects decorators, non-literal defaults, top-level executable statements, unsafe annotations, and unsafe base classes).
2. Verifies the class can be **instantiated with no arguments**.
3. Calls `generate(data_map: Dict[str, pd.DataFrame]) -> Dict[str, pd.Series]`.

### Signal Interface

```python
class SignalEngine:
    def __init__(self, ...):  # all params must have defaults (runner calls SignalEngine() with no args)
        ...

    def generate(self, data_map: Dict[str, pd.DataFrame]) -> Dict[str, pd.Series]:
        """Return a dict mapping each symbol code to a pd.Series of signals.

        The Series should have a DatetimeIndex matching the OHLCV data timestamps.
        Signal values should be in range [-1, +1], where +1 = long, -1 = short, 0 = flat.

        The returned signals are shifted by 1 bar (next-bar-open semantics) by the engine
        to prevent look-ahead bias.
        """
        ...
```

### Data Available in `data_map`

For each symbol code, `data_map[code]` is a pandas DataFrame with columns:
`open`, `high`, `low`, `close`, `volume`, plus any extra fields fetched from the data source.

If `fundamental_fields` is configured in the config, the DataFrame is **enriched** with Tushale statement data (balance sheet, income statement, cash flow) before signal generation.

If `event_feeds` is configured, an `event_score` column is enriched from RSSHub event feeds.

## Validation

Statistical validation runs automatically when `config["validation"]` is present, via `backtest/validation.py`:

### Monte Carlo Permutation Test

Shuffles the order of completed trade P&L to test whether the observed Sharpe ratio and max drawdown are statistically significant. A p-value < 0.05 means the performance is unlikely to be explained by random ordering of the same trades.

### Bootstrap Sharpe Confidence Interval

Resamples daily returns with replacement to estimate the confidence interval on the Sharpe ratio and computes the probability that the true Sharpe ratio is positive.

### Walk-Forward Analysis

Splits the backtest into sequential non-overlapping windows, evaluates Sharpe, max drawdown, win rate, and total return per window, then computes a consistency rate (fraction of windows that are profitable). Tests robustness across market regimes.

## Metrics & Artifacts

After each run, the following artifacts are written to `run_dir/artifacts/`:

| File               | Content                                                                                                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `equity.csv`       | Portfolio equity curve, returns, drawdowns, and benchmark equity                                                                                                                     |
| `trades.csv`       | Entry and exit events (order code, side, price, qty, P&L, holding days)                                                                                                              |
| `ohlcv_{code}.csv` | Per-symbol OHLCV data                                                                                                                                                                |
| `positions.csv`    | Target position weights over time                                                                                                                                                    |
| `metrics.csv`      | Summary metrics (final value, total return, annual return, Sharpe, Sortino, max drawdown, Calmar, win rate, profit factor, trade count, benchmark/excess returns, information ratio) |
| `validation.json`  | Statistical validation results (Monte Carlo p-values, Bootstrap CI, Walk-Forward consistency)                                                                                        |

### Available Metrics

| Metric                 | Description                                 |
| ---------------------- | ------------------------------------------- |
| `final_value`          | Portfolio value at end of backtest          |
| `total_return`         | Total return as a fraction                  |
| `annual_return`        | Annualized return (CAGR)                    |
| `max_drawdown`         | Maximum peak-to-trough drawdown             |
| `sharpe`               | Annualized Sharpe ratio                     |
| `sortino`              | Annualized Sortino ratio                    |
| `calmar`               | Calmar ratio (annual return / max drawdown) |
| `win_rate`             | Fraction of profitable trades               |
| `profit_loss_ratio`    | Average win / average loss                  |
| `profit_factor`        | Gross profit / gross loss                   |
| `max_consecutive_loss` | Longest losing streak                       |
| `avg_holding_days`     | Average bars held per position              |
| `trade_count`          | Number of completed round-trip trades       |
| `benchmark_return`     | Return of the benchmark index               |
| `excess_return`        | Strategy return minus benchmark return      |
| `information_ratio`    | Excess return per unit of tracking error    |

## Trust Layer Run Cards

Each backtest produces a **run card** (`run_card.json` and `run_card.md`) in the run directory for full reproducibility:

- **Config hash** — SHA-256 of the configuration (enables re-running with identical parameters)
- **Strategy hash** — SHA-256 of the `signal_engine.py` source (enables identical strategy reproduction)
- **Data sources** — Which data sources were actually used for each code
- **Metrics** — Scalar metric values at time of writing
- **Artifacts manifest** — List of all artifact files with paths, sizes, and SHA-256 hashes

The run card provides an auditable trail: anyone can verify that a run used the exact same configuration and strategy code by comparing hashes, and can locate every artifact file needed to reproduce the results.

## Project Structure

```
agent/backtest/
├── __init__.py          # Empty (package marker)
├── benchmark.py         # Benchmark ticker resolution (SPY, CSI 300, etc.)
├── correlation.py       # Cross-asset correlation matrix (Pearson / Spearman)
├── metrics.py           # Performance metrics (Sharpe, Sortino, max DD, CAGR, etc.)
├── models.py            # Data models: Position, TradeRecord, EquitySnapshot (dataclasses)
├── run_card.py          # Trust Layer run card (JSON + Markdown)
├── runner.py            # Core entrypoint: config loading, data fetching, engine routing, execution
├── validation.py        # Statistical validation (Monte Carlo, Bootstrap CI, Walk-Forward)
├── engines/
│   ├── __init__.py      # Engine map (Wave 1 + Wave 2 documentation)
│   ├── base.py          # BaseEngine — shared bar-by-bar execution loop
│   ├── _market_hooks.py # Market detection, funding/liquidation/swap helpers
│   ├── china_a.py       # China A-shares engine (T+1, price limits, stamp tax)
│   ├── global_equity.py # US / HK equities engine
│   ├── crypto.py        # Crypto perpetuals engine (funding fees, liquidation)
│   ├── forex.py         # FX spot/CFD engine (spread, swap, high leverage)
│   ├── futures_base.py  # Futures base — contract multiplier logic (Wave 2)
│   ├── china_futures.py # China commodity/financial futures (Wave 2)
│   ├── global_futures.py# International futures (CME/ICE/Eurex, Wave 2)
│   ├── composite.py     # Cross-market engine with shared capital pool (Wave 3)
│   └── options_portfolio.py  # Options (Black-Scholes, IV smile, Greeks)
├── loaders/
│   ├── __init__.py      # Loader registry initialization
│   ├── base.py          # DataLoader protocol, validation, retry helpers
│   ├── registry.py      # Global loader registry + market-to-source fallback chains
│   ├── tushare.py       # Tushale (A-shares, funds, futures)
│   ├── yfinance_loader.py  # Yahoo Finance (US/HK equity universal fallback)
│   ├── okx.py           # OKX crypto exchange
│   ├── akshare_loader.py    # AKShare (broad China markets)
│   ├── ccxt_loader.py       # CCXT (multi-exchange crypto)
│   ├── futu.py            # Futu Securities
│   ├── mootdx_loader.py     # MootDX (A-shares)
│   ├── alphavantage_loader.py  # Alpha Vantage (daily OHLCV, key-gated)
│   ├── finnhub_loader.py      # Finnhub (stock candles, key-gated)
│   ├── fmp_loader.py            # Financial Modeling Prep
│   ├── baostock_loader.py       # BaoStock
│   ├── eastmoney_loader.py      # EastMoney
│   ├── sina_loader.py           # Sina Finance
│   ├── stooq_loader.py          # Stooq
│   ├── tiingo_loader.py         # Tiingo
│   ├── tencent_loader.py        # Tencent Finance
│   ├── yahoo_loader.py          # Yahoo (direct HTTP, no-auth)
│   ├── yahoo_client.py          # Yahoo low-level client
│   ├── sec_edgar_client.py      # SEC EDGAR filings
│   ├── local_loader.py          # Local CSV files
│   ├── tushare_fundamentals.py  # Tushare statement enrichment
│   ├── rsshub_events.py         # RSSHub event feed enrichment
└── optimizers/
    ├── __init__.py
    ├── base.py          # Optimizer base class
    ├── mean_variance.py  # Mean-variance (Markowitz)
    ├── risk_parity.py    # Risk parity (inverse-vol weighting)
    ├── equal_volatility.py  # Equal volatility
    └── max_diversification.py  # Maximum diversification ratio
```

## Contributing

Contributions are welcome. Please open an issue to discuss significant changes before submitting a pull request.

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Commit your changes
4. Push and open a PR

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

**Disclaimer:** This framework is for educational and research purposes only. Backtested performance does not guarantee future results. Nothing in this repository constitutes financial advice.

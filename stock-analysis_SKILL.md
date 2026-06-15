---
name: stock-analysis
description: >
  Earnings-Event Stock Analysis & AI Sector Valuation
  Domain: Quantitative Finance / Data Journalism / Avant-Garde UI/UX
version: 3.0
---

# SKILL.md - Earnings-Event Stock Analysis & AI Sector Valuation

**Domain:** Quantitative Finance / Data Journalism / Avant-Garde UI/UX

---

## 📋 Overview

**Skill Name:** Earnings-Event Stock Analysis & Sustainable Opportunity Assessment

**Purpose:** Guide an AI agent through a rigorous, multi-phase pipeline that:
1. Analyzes stock price behavior around earnings announcements
2. Benchmarks valuation against sector/indices with dynamic data
3. Backtests earnings-timing strategies with **fair time-in-market adjustments**
4. Produces an **interactive, editorial-style report** answering: *"Is this stock's performance driven by sustainable fundamentals or short-term hype?"*

**Applicable Tickers:** Any publicly traded equity (demonstrated with NVDA, AAPL, MSFT)  
**Time Horizon:** 2+ years recommended (minimum 8 earnings events)  
**Technical Stack:** Python 3.11+, pandas, yfinance, scipy, Plotly.js/Recharts, Jinja2

---

## 🎯 Core Philosophy

### Anti-Generic Mandate
- **REJECT** static Matplotlib PNGs for web reports
- **REJECT** Bootstrap-style card grids and predictable layouts
- **REJECT** Inter/Roboto system fonts without intentional typography
- **EMBRACE** interactive, bespoke visualizations with micro-interactions
- **EMBRACE** editorial/data-journalism aesthetic with intentional minimalism
- **EMBRACE** dark charcoal backgrounds, warm sand text, terminal green accents

### Technical Rigor
- **Timezone normalization** is mandatory (not optional)
- **Fair backtesting** requires time-in-market adjustments
- **Statistical claims** require p-values and confidence intervals
- **Risk metrics** (Sharpe, Sortino, Calmar) are non-negotiable
- **Dynamic data sourcing** prevents stale benchmarks

---

## 📚 Table of Contents

1. [Prerequisites & Environment](#1-prerequisites--environment)
2. [Phase 1: Data Acquisition](#2-phase-1-data-acquisition)
3. [Phase 2: Data Processing & Feature Engineering](#3-phase-2-data-processing--feature-engineering)
4. [Phase 3: Multi-Dimensional Analysis](#4-phase-3-multi-dimensional-analysis)
5. [Phase 4: Avant-Garde Visualization](#5-phase-4-avant-garde-visualization)
6. [Phase 5: Report Generation](#6-phase-5-report-generation)
7. [Phase 6: Validation & QA](#7-phase-6-validation--qa)
8. [Edge Cases, Pitfalls & Augmented Insights](#8-edge-cases-pitfalls--augmented-insights)
9. [Reusable Code Templates](#9-reusable-code-templates)
10. [Quality Checklist](#10-quality-checklist)

---

## 1. Prerequisites & Environment

### Python Dependencies

```bash
pip install yfinance pandas numpy matplotlib seaborn scipy jinja2 requests beautifulsoup4 lxml
```

| Library | Purpose | Version Constraint |
|---------|---------|-------------------|
| `yfinance` | Historical prices, stock info, earnings | >=0.2.30 |
| `pandas` | Time series manipulation, alignment | >=2.0.0 |
| `numpy` | Numerical computation, array operations | >=1.24.0 |
| `matplotlib` / `seaborn` | Static charting (for debugging) | >=3.7.0 |
| `scipy` | Statistical tests (t-test, Pearson correlation) | >=1.10.0 |
| `jinja2` | HTML report templating | >=3.1.0 |
| `requests` / `beautifulsoup4` | Web search fallback for sector P/E | >=4.3.0 |
| `lxml` | HTML/XML parsing | >=4.9.0 |

### Frontend Dependencies (for Interactive Reports)

For **interactive HTML reports**, embed via CDN:
- **Plotly.js** (v2.27+) for interactive charts
- **Google Fonts**: DM Serif Display, JetBrains Mono
- **Tailwind CSS** (v3.4+) for utility-first styling (optional)

### Environment Verification

```python
import yfinance as yf
import pandas as pd
import numpy as np
from scipy import stats
import matplotlib
matplotlib.use('Agg')  # headless rendering
import matplotlib.pyplot as plt
from datetime import datetime, timedelta
import json
import os

# Verify data access
test = yf.download("AAPL", period="5d", progress=False)
assert len(test) > 0, "Yahoo Finance API not accessible"
print("✓ Environment validated successfully")
```

---

## 2. Phase 1: Data Acquisition

### 2a. Historical Price Data (OHLCV)

**CRITICAL:** The #1 source of silent failures is timezone mismatch.

```python
def fetch_price_history(ticker: str, years: int = 2) -> pd.DataFrame:
    """
    Fetch OHLCV data with IMMEDIATE timezone normalization.
    
    WHY: yfinance returns tz-aware indices (America/New_York).
    Merging with tz-naive data causes SILENT failures (empty results).
    
    Args:
        ticker: Stock symbol (e.g., 'NVDA')
        years: Number of years of history
    
    Returns:
        DataFrame with DatetimeIndex (tz-naive), columns: Open, High, Low, Close, Volume
    """
    stock = yf.Ticker(ticker)
    end = datetime.now()
    start = end - timedelta(days=years * 365)
    
    df = stock.history(start=start, end=end, auto_adjust=True)
    
    # CRITICAL: Normalize timezone IMMEDIATELY
    if df.index.tz is not None:
        df.index = df.index.tz_localize(None)
    df.index.name = 'Date'
    
    # Save checkpoint
    df.to_csv(f"{ticker}_prices_{years}y.csv")
    
    return df
```

### 2b. Stock Info & Fundamentals

```python
def fetch_stock_info(ticker: str) -> dict:
    """
    Fetch current valuation metrics and company info.
    
    Returns dict with keys:
    - trailingPE, forwardPE, marketCap
    - sector, industry, shortName
    - beta, priceToBook, dividendYield
    
    NOTE: Some fields may be None. Always validate before use.
    """
    stock = yf.Ticker(ticker)
    info = stock.info
    
    relevant_keys = [
        'shortName', 'sector', 'industry', 'marketCap',
        'trailingPE', 'forwardPE', 'trailingEps', 'forwardEps',
        'priceToBook', 'dividendYield', 'beta',
        'fiftyTwoWeekHigh', 'fiftyTwoWeekLow',
        'averageVolume', 'sharesOutstanding'
    ]
    return {k: info.get(k) for k in relevant_keys}
```

### 2c. Earnings Data

```python
def fetch_earnings_data(ticker: str) -> tuple[pd.DataFrame, pd.DataFrame]:
    """
    Fetch quarterly earnings history and earnings dates.
    
    Returns:
        earnings_history: DataFrame [Revenue, Earnings] by quarter
        earnings_dates: DataFrame [Earnings Date, EPS Estimate, Reported EPS, Surprise(%)]
    
    CRITICAL: 
    - earnings_dates includes FUTURE dates with NaN for Reported EPS
    - MUST filter these out before analysis
    - MUST normalize timezone
    """
    stock = yf.Ticker(ticker)
    
    earnings_history = stock.quarterly_earnings
    earnings_dates = stock.earnings_dates
    
    if earnings_dates is not None:
        earnings_dates = earnings_dates.reset_index()
        earnings_dates.columns = ['Earnings Date', 'EPS Estimate', 'Reported EPS', 'Surprise(%)']
        
        # Remove FUTURE dates (no reported EPS yet)
        earnings_dates = earnings_dates.dropna(subset=['Reported EPS'])
        
        # Normalize timezone
        earnings_dates['Earnings Date'] = pd.to_datetime(
            earnings_dates['Earnings Date']
        ).dt.tz_localize(None)
    
    return earnings_history, earnings_dates
```

### 2d. Benchmark & Sector P/E (Dynamic Sourcing)

```python
def fetch_benchmark(ticker: str = "SPY", years: int = 2) -> pd.DataFrame:
    """Fetch benchmark index prices for relative performance comparison."""
    return fetch_price_history(ticker, years)

def get_sector_pe_via_web_search(company_sector: str, current_year: int) -> dict:
    """
    Use web search to find CURRENT sector P/E ratios.
    
    WHY: Hardcoded P/E values become stale. Dynamic sourcing ensures accuracy.
    
    Search queries:
    - "S&P 500 {sector} sector average P/E ratio {current_year}"
    - "semiconductor industry average P/E ratio"
    - "technology sector valuation metrics"
    
    Returns:
        {
            'sector_pe': float,      # e.g., 39.1
            'index_pe': float,       # e.g., 22.5 for S&P 500
            'source': str,           # attribution (e.g., "S&P Global")
            'as_of': str             # date of data
        }
    
    NOTE: Web-sourced data is approximate. Use as directional benchmark.
    """
    # Agent should implement web search using requests/BeautifulSoup
    # or use a search API (e.g., SerpAPI, Bing Search API)
    
    # Example implementation structure:
    queries = [
        f"S&P 500 {company_sector} sector average P/E ratio {current_year}",
        f"{company_sector} industry average P/E ratio {current_year}"
    ]
    
    # Placeholder - agent should implement actual search
    return {
        'sector_pe': 39.1,  # Replace with actual search result
        'index_pe': 22.5,
        'source': 'S&P Global',
        'as_of': datetime.now().strftime('%Y-%m-%d')
    }
```

### 2e. Industry Context (Web Search)

```python
INDUSTRY_SEARCH_QUERIES = [
    "{company} AI market size {year}",
    "{sector} infrastructure spending forecast",
    "{company} market share {technology}",
    "{company} revenue growth data center {year}",
    "AI adoption enterprise statistics {year}",
    "generative AI market growth rate {year}"
]

def extract_industry_context(company_name: str, sector: str, year: int) -> dict:
    """
    Extract qualitative context from web searches.
    
    Returns:
        {
            'market_size': str,           # e.g., "$500B in 2024"
            'growth_rate': str,           # e.g., "35% CAGR"
            'adoption_metrics': str,      # e.g., "1B active AI models"
            'competitive_landscape': str, # e.g., "NVIDIA 80% market share"
            'sources': list               # attribution list
        }
    """
    # Agent should perform web searches and extract key metrics
    # This provides qualitative context for "Hype vs Sustainable" assessment
    pass
```

---

## 3. Phase 2: Data Processing & Feature Engineering

### 3a. Timezone Normalization (Mandatory)

**ALREADY DONE in 2a.** Apply `tz_localize(None)` immediately after every yfinance fetch.

**Failure Symptom:** Silent empty results on merges or `.loc[]` lookups.

### 3b. Earnings Event Window Alignment

```python
def align_earnings_to_prices(
    earnings_dates: pd.DataFrame,
    price_df: pd.DataFrame,
    window_before: int = 7,
    window_after: int = 7
) -> pd.DataFrame:
    """
    For each earnings date, find the actual trading day and compute
    price changes in a window around it.
    
    CRITICAL:
    - Earnings are often announced AFTER market close or BEFORE market open
    - The "event day" return should use the NEXT trading day's close
    - MUST use actual trading days (not calendar days)
    
    Args:
        earnings_dates: DataFrame with 'Earnings Date' column
        price_df: OHLCV DataFrame with DatetimeIndex
        window_before: Trading days before earnings
        window_after: Trading days after earnings
    
    Returns:
        DataFrame with columns:
        - earnings_date, actual_trade_date
        - close_before, close_day_of, close_day_after
        - close_window_before, close_window_after
        - return_1d, return_2d, return_window
        - eps_estimate, eps_actual, eps_surprise_pct
        - volume_day_of
    """
    events = []
    trading_dates = price_df.index.sort_values()
    
    for _, row in earnings_dates.iterrows():
        earn_date = pd.Timestamp(row['Earnings Date'])
        
        # Find nearest trading day ON or AFTER earnings date
        future_dates = trading_dates[trading_dates >= earn_date]
        if len(future_dates) == 0:
            continue
        actual_date = future_dates[0]
        
        # Find trading day BEFORE earnings
        past_dates = trading_dates[trading_dates < earn_date]
        if len(past_dates) < window_before:
            continue
        date_before = past_dates[-1]
        date_window_before = past_dates[-window_before]
        
        # Find trading day AFTER
        idx = trading_dates.get_loc(actual_date)
        if idx + window_after >= len(trading_dates):
            continue
        date_after = trading_dates[idx + 1]
        date_window_after = trading_dates[idx + window_after]
        
        events.append({
            'earnings_date': earn_date,
            'actual_trade_date': actual_date,
            'close_before': price_df.loc[date_before, 'Close'],
            'close_day_of': price_df.loc[actual_date, 'Close'],
            'close_day_after': price_df.loc[date_after, 'Close'],
            'close_window_before': price_df.loc[date_window_before, 'Close'],
            'close_window_after': price_df.loc[date_window_after, 'Close'],
            'volume_day_of': price_df.loc[actual_date, 'Volume'],
            'eps_estimate': row.get('EPS Estimate'),
            'eps_actual': row.get('Reported EPS'),
            'eps_surprise_pct': row.get('Surprise(%)')
        })
    
    result = pd.DataFrame(events)
    
    # Calculate returns
    result['return_1d'] = (result['close_day_of'] - result['close_before']) / result['close_before']
    result['return_2d'] = (result['close_day_after'] - result['close_before']) / result['close_before']
    result['return_window'] = (
        (result['close_window_after'] - result['close_window_before']) / 
        result['close_window_before']
    )
    
    return result
```

### 3c. Derived Metrics

```python
def compute_derived_metrics(
    price_df: pd.DataFrame, 
    benchmark_df: pd.DataFrame = None
) -> pd.DataFrame:
    """
    Add analytical columns to the price DataFrame.
    
    Returns DataFrame with added columns:
    - daily_return, cumulative_return
    - volatility_20d (annualized)
    - ma_50, ma_200
    - excess_return (vs benchmark)
    - volume_ratio (vs 20-day MA)
    """
    df = price_df.copy()
    
    # Daily returns
    df['daily_return'] = df['Close'].pct_change()
    
    # Cumulative return from start
    df['cumulative_return'] = (1 + df['daily_return']).cumprod() - 1
    
    # Rolling volatility (20-day, annualized)
    df['volatility_20d'] = df['daily_return'].rolling(20).std() * np.sqrt(252)
    
    # 50/200 day moving averages
    df['ma_50'] = df['Close'].rolling(50).mean()
    df['ma_200'] = df['Close'].rolling(200).mean()
    
    # Volume anomalies
    df['volume_ma_20'] = df['Volume'].rolling(20).mean()
    df['volume_ratio'] = df['Volume'] / df['volume_ma_20']
    
    # Relative to benchmark
    if benchmark_df is not None:
        bench_aligned = benchmark_df['Close'].reindex(df.index, method='ffill')
        df['benchmark_return'] = bench_aligned.pct_change()
        df['excess_return'] = df['daily_return'] - df['benchmark_return']
        df['cumulative_benchmark'] = (1 + df['benchmark_return']).cumprod() - 1
    
    return df
```

### 3d. Data Quality Validation

```python
def validate_data(
    price_df: pd.DataFrame, 
    earnings_df: pd.DataFrame
) -> list[str]:
    """
    Run sanity checks. Return list of warnings (empty = all good).
    
    Checks:
    - Gaps > 5 days (holidays are 1-3 days)
    - Zero-volume days
    - Missing EPS data
    - Sufficient data length (>= 252 trading days)
    """
    warnings = []
    
    # Check for gaps > 5 days
    date_diffs = price_df.index.to_series().diff().dt.days
    large_gaps = date_diffs[date_diffs > 5]
    if len(large_gaps) > 0:
        warnings.append(
            f"Found {len(large_gaps)} gaps > 5 trading days. "
            "May indicate data issues."
        )
    
    # Check for zero-volume days
    zero_vol = (price_df['Volume'] == 0).sum()
    if zero_vol > 0:
        warnings.append(
            f"Found {zero_vol} zero-volume days. "
            "These may be holidays or data errors."
        )
    
    # Check earnings dates have valid EPS data
    if earnings_df is not None:
        missing_eps = earnings_df['eps_actual'].isna().sum()
        if missing_eps > 0:
            warnings.append(
                f"{missing_eps} earnings events missing actual EPS."
            )
    
    # Check for sufficient data length
    if len(price_df) < 252:
        warnings.append(
            "Less than 1 year of trading data. "
            "Statistical conclusions may be unreliable."
        )
    
    return warnings
```

---

## 4. Phase 3: Multi-Dimensional Analysis

### 4a. Event Study — Price Action Around Earnings

```python
def earnings_event_study(events_df: pd.DataFrame) -> dict:
    """
    Analyze stock returns around earnings announcements.
    
    Returns dict with:
    - total_events, mean_1d_return, median_1d_return
    - positive_rate (% of events with positive return)
    - mean_window_return, best/worst_earnings_return
    - correlation_surprise_return (Pearson)
    - correlation_p_value (statistical significance)
    - regression_slope, regression_r_squared
    """
    clean = events_df.dropna(subset=['return_1d', 'eps_surprise_pct'])
    
    results = {
        'total_events': len(events_df),
        'mean_1d_return': events_df['return_1d'].mean(),
        'median_1d_return': events_df['return_1d'].median(),
        'std_1d_return': events_df['return_1d'].std(),
        'positive_rate': (events_df['return_1d'] > 0).mean(),
        'mean_2d_return': events_df['return_2d'].mean(),
        'mean_window_return': events_df['return_window'].mean(),
        'best_earnings_return': events_df['return_1d'].max(),
        'worst_earnings_return': events_df['return_1d'].min(),
    }
    
    if len(clean) >= 3:
        # Correlation with p-value
        corr, p_value = stats.pearsonr(
            clean['eps_surprise_pct'], 
            clean['return_1d']
        )
        results['correlation_surprise_return'] = corr
        results['correlation_p_value'] = p_value
        results['correlation_significant'] = p_value < 0.05
        
        # Linear regression
        slope, intercept, r_value, p_val, std_err = stats.linregress(
            clean['eps_surprise_pct'], 
            clean['return_1d']
        )
        results['regression_slope'] = slope
        results['regression_r_squared'] = r_value ** 2
        results['regression_p_value'] = p_val
    
    return results
```

**Interpretation Guide:**

| Finding | Meaning |
|---------|---------|
| High positive rate (>70%) | Market consistently rewards earnings |
| Low surprise-return correlation (p > 0.05) | Stock moves on guidance/narrative, not just numbers |
| High volatility on earnings days | Earnings are a significant risk event |
| Window return >> 1-day return | The move extends beyond the announcement |

### 4b. Earnings-Timing Strategy Backtest (Fair Comparison)

**CRITICAL CORRECTION:** The original workflow compared a strategy invested only ~22% of the time against buy-and-hold invested 100% of the time. This is misleading.

**MUST compute:**
- Time-in-market adjustment
- Annualized returns
- Sharpe ratio
- Sortino ratio
- Calmar ratio
- Maximum drawdown

```python
def backtest_earnings_strategy(
    price_df: pd.DataFrame,
    earnings_dates: pd.DataFrame,
    days_before: int = 7,
    days_after: int = 7,
    initial_capital: float = 10000.0
) -> dict:
    """
    Simulate: Buy N days before each earnings, sell N days after.
    
    CRITICAL: Compare FAIRLY using:
    - Time-in-market adjustment
    - Annualized returns
    - Risk-adjusted metrics (Sharpe, Sortino, Calmar)
    - Max drawdown
    
    Returns:
        {
            'trades': DataFrame of individual trades
            'total_trades': int
            'win_rate': float
            'avg_trade_return': float
            'strategy_cumulative_return': float
            'strategy_annualized_return': float
            'buyhold_cumulative_return': float
            'buyhold_annualized_return': float
            'time_in_market': float (0.0 to 1.0)
            'sharpe_ratio': float
            'sortino_ratio': float
            'calmar_ratio': float
            'max_drawdown': float
            'strategy_final_value': float
            'buyhold_final_value': float
        }
    """
    trading_dates = price_df.index.sort_values()
    trades = []
    
    for _, row in earnings_dates.iterrows():
        earn_date = pd.Timestamp(row['Earnings Date'])
        
        # Find buy date (N trading days before)
        past_dates = trading_dates[trading_dates < earn_date]
        if len(past_dates) < days_before:
            continue
        buy_date = past_dates[-days_before]
        
        # Find sell date (N trading days after)
        future_dates = trading_dates[trading_dates >= earn_date]
        if len(future_dates) < days_after:
            continue
        sell_date = future_dates[days_after - 1] if days_after <= len(future_dates) else future_dates[-1]
        
        buy_price = price_df.loc[buy_date, 'Close']
        sell_price = price_df.loc[sell_date, 'Close']
        trade_return = (sell_price - buy_price) / buy_price
        
        trades.append({
            'buy_date': buy_date,
            'sell_date': sell_date,
            'buy_price': buy_price,
            'sell_price': sell_price,
            'return': trade_return,
            'holding_days': (sell_date - buy_date).days
        })
    
    trades_df = pd.DataFrame(trades)
    
    # Strategy cumulative return
    strategy_cumulative = (1 + trades_df['return']).prod() - 1
    
    # Buy-and-hold over same period
    first_trade = trades_df['buy_date'].min()
    last_trade = trades_df['sell_date'].max()
    bh_start = price_df.loc[first_trade, 'Close']
    bh_end = price_df.loc[last_trade, 'Close']
    bh_cumulative = (bh_end - bh_start) / bh_start
    
    # Time in market
    total_holding_days = trades_df['holding_days'].sum()
    total_calendar_days = (last_trade - first_trade).days
    time_in_market = total_holding_days / total_calendar_days if total_calendar_days > 0 else 0
    
    # Annualized returns
    years_held = total_calendar_days / 365.25
    strategy_annualized = (1 + strategy_cumulative) ** (1 / years_held) - 1 if years_held > 0 else 0
    bh_annualized = (1 + bh_cumulative) ** (1 / years_held) - 1 if years_held > 0 else 0
    
    # Risk-adjusted metrics
    risk_free = 0.045
    if len(trades_df) > 1:
        trades_per_year = len(trades_df) / (total_calendar_days / 365)
        mean_return = trades_df['return'].mean()
        std_return = trades_df['return'].std()
        
        # Sharpe ratio
        sharpe = (mean_return * trades_per_year - risk_free) / (std_return * np.sqrt(trades_per_year)) if std_return > 0 else 0
        
        # Sortino ratio (downside deviation)
        downside_returns = trades_df[trades_df['return'] < 0]['return']
        downside_std = downside_returns.std() * np.sqrt(trades_per_year) if len(downside_returns) > 0 else 0
        sortino = (mean_return * trades_per_year - risk_free) / downside_std if downside_std > 0 else 0
        
        # Max drawdown
        portfolio_values = [initial_capital]
        for ret in trades_df['return']:
            portfolio_values.append(portfolio_values[-1] * (1 + ret))
        pv_series = pd.Series(portfolio_values)
        rolling_max = pv_series.cummax()
        drawdown = (pv_series - rolling_max) / rolling_max
        max_drawdown = drawdown.min()
        
        # Calmar ratio
        calmar = strategy_annualized / abs(max_drawdown) if max_drawdown != 0 else float('inf')
    else:
        sharpe = sortino = calmar = 0
        max_drawdown = 0
    
    strategy_final = initial_capital * (1 + strategy_cumulative)
    bh_final = initial_capital * (1 + bh_cumulative)
    
    return {
        'trades': trades_df,
        'total_trades': len(trades_df),
        'win_rate': (trades_df['return'] > 0).mean(),
        'avg_trade_return': trades_df['return'].mean(),
        'strategy_cumulative_return': strategy_cumulative,
        'strategy_annualized_return': strategy_annualized,
        'buyhold_cumulative_return': bh_cumulative,
        'buyhold_annualized_return': bh_annualized,
        'underperformance': strategy_cumulative - bh_cumulative,
        'time_in_market': time_in_market,
        'sharpe_ratio': sharpe,
        'sortino_ratio': sortino,
        'calmar_ratio': calmar,
        'max_drawdown': max_drawdown,
        'strategy_final_value': strategy_final,
        'buyhold_final_value': bh_final,
        'total_holding_days': total_holding_days,
        'total_calendar_days': total_calendar_days,
    }
```

### 4c. Valuation Benchmarking

```python
def valuation_analysis(
    stock_info: dict,
    sector_pe: float,
    index_pe: float
) -> dict:
    """
    Compare current valuation to sector and market benchmarks.
    
    Returns:
        {
            'trailing_pe': float,
            'forward_pe': float,
            'sector_pe': float,
            'index_pe': float,
            'trailing_premium_vs_sector': float (%),
            'forward_premium_vs_sector': float (%),
            'trailing_premium_vs_market': float (%),
            'earnings_growth_implied': float (%),  # if forward < trailing
            'valuation_note': str
        }
    """
    trailing_pe = stock_info.get('trailingPE')
    forward_pe = stock_info.get('forwardPE')
    
    results = {
        'trailing_pe': trailing_pe,
        'forward_pe': forward_pe,
        'sector_pe': sector_pe,
        'index_pe': index_pe,
        'trailing_premium_vs_sector': ((trailing_pe / sector_pe) - 1) * 100 if sector_pe else None,
        'forward_premium_vs_sector': ((forward_pe / sector_pe) - 1) * 100 if sector_pe else None,
        'trailing_premium_vs_market': ((trailing_pe / index_pe) - 1) * 100 if index_pe else None,
    }
    
    # Key insight: Forward P/E can justify high trailing P/E if growth expected
    if trailing_pe and forward_pe and forward_pe < trailing_pe:
        results['earnings_growth_implied'] = (trailing_pe / forward_pe - 1) * 100
        results['valuation_note'] = (
            "Forward P/E below trailing suggests expected earnings growth. "
            f"Implies {(trailing_pe / forward_pe - 1) * 100:.1f}% earnings growth."
        )
    
    return results
```

### 4d. Risk-Adjusted Performance Metrics

```python
def risk_metrics(
    price_df: pd.DataFrame, 
    benchmark_df: pd.DataFrame = None,
    risk_free_rate: float = 0.045
) -> dict:
    """
    Compute comprehensive risk-adjusted return metrics.
    
    Returns:
        {
            'total_return': float,
            'annualized_return': float,
            'annualized_volatility': float,
            'sharpe_ratio': float,
            'sortino_ratio': float,
            'calmar_ratio': float,
            'max_drawdown': float,
            'beta': float (if benchmark provided),
            'alpha': float (if benchmark provided),
            'information_ratio': float (if benchmark provided),
            'positive_days_pct': float,
            'best_day': float,
            'worst_day': float
        }
    """
    returns = price_df['Close'].pct_change().dropna()
    
    # Total and annualized return
    total_return = (price_df['Close'].iloc[-1] / price_df['Close'].iloc[0]) - 1
    years = (price_df.index[-1] - price_df.index[0]).days / 365.25
    annualized_return = (1 + total_return) ** (1 / years) - 1 if years > 0 else 0
    
    # Annualized volatility
    annualized_vol = returns.std() * np.sqrt(252)
    
    # Sharpe ratio
    sharpe = (annualized_return - risk_free_rate) / annualized_vol if annualized_vol > 0 else 0
    
    # Sortino ratio (downside deviation only)
    downside_returns = returns[returns < 0]
    downside_std = downside_returns.std() * np.sqrt(252) if len(downside_returns) > 0 else 0
    sortino = (annualized_return - risk_free_rate) / downside_std if downside_std > 0 else 0
    
    # Maximum drawdown
    cumulative = (1 + returns).cumprod()
    rolling_max = cumulative.cummax()
    drawdown = (cumulative - rolling_max) / rolling_max
    max_drawdown = drawdown.min()
    
    # Calmar ratio
    calmar = annualized_return / abs(max_drawdown) if max_drawdown != 0 else float('inf')
    
    # Beta, alpha, information ratio (if benchmark provided)
    beta = None
    alpha = None
    information_ratio = None
    
    if benchmark_df is not None:
        bench_returns = benchmark_df['Close'].pct_change().dropna()
        common = returns.index.intersection(bench_returns.index)
        
        if len(common) > 20:
            # Beta
            cov = np.cov(returns.loc[common], bench_returns.loc[common])
            beta = cov[0, 1] / cov[1, 1] if cov[1, 1] != 0 else 0
            
            # Alpha (Jensen's alpha)
            bench_annual = (benchmark_df['Close'].iloc[-1] / benchmark_df['Close'].iloc[0]) ** (1/years) - 1 if years > 0 else 0
            alpha = annualized_return - (risk_free_rate + beta * (bench_annual - risk_free_rate))
            
            # Information ratio
            excess_returns = returns.loc[common] - bench_returns.loc[common]
            tracking_error = excess_returns.std() * np.sqrt(252)
            information_ratio = excess_returns.mean() * 252 / tracking_error if tracking_error > 0 else 0
    
    return {
        'total_return': total_return,
        'annualized_return': annualized_return,
        'annualized_volatility': annualized_vol,
        'sharpe_ratio': sharpe,
        'sortino_ratio': sortino,
        'calmar_ratio': calmar,
        'max_drawdown': max_drawdown,
        'beta': beta,
        'alpha': alpha,
        'information_ratio': information_ratio,
        'positive_days_pct': (returns > 0).mean(),
        'best_day': returns.max(),
        'worst_day': returns.min(),
    }
```

### 4e. Fundamental Assessment Synthesis

```python
def synthesize_fundamental_assessment(
    event_study: dict,
    strategy_backtest: dict,
    valuation: dict,
    risk: dict,
    industry_context: dict
) -> dict:
    """
    Combine all analyses into a cohesive assessment.
    
    Evaluates:
    1. EARNINGS QUALITY
       - Consistency of beats (positive_rate)
       - Magnitude of surprise
       - Market reaction rationality (correlation)
    
    2. PRICE ACTION INTEGRITY
       - Gains sustained between earnings? (backtest results)
       - Volatility concentration (event study window)
       - Momentum vs fundamental-driven
    
    3. VALUATION REASONABLENESS
       - Premium justified by growth? (forward vs trailing P/E)
       - Historical context (web search)
       - Realistic growth scenarios
    
    4. RISK PROFILE
       - Max drawdown expectation
       - Risk-adjusted return attractiveness (Sharpe > 1.0)
       - Alpha generation vs benchmark
    
    5. INDUSTRY FUNDAMENTALS
       - Real demand (revenue growth, market size)
       - Competitive moat sustainability
       - Structural tailwinds
    
    Returns:
        {
            'earnings_quality': str,         # 'High' / 'Medium' / 'Low'
            'price_action_integrity': str,   # 'Sustainable' / 'Speculative'
            'valuation_assessment': str,     # 'Reasonable' / 'Premium' / 'Expensive'
            'risk_profile': str,             # 'Conservative' / 'Moderate' / 'Aggressive'
            'industry_fundamentals': str,    # 'Strong' / 'Moderate' / 'Weak'
            'overall_verdict': str,          # 'Sustainable' / 'Mixed' / 'Hype'
            'recommendation': str            # 'BUY' / 'HOLD' / 'AVOID'
        }
    """
    # Agent synthesizes based on all inputs
    # This is where qualitative judgment meets quantitative data
    pass
```

---

## 5. Phase 4: Avant-Garde Visualization

### Design Philosophy

**REJECT:** Static Matplotlib PNGs embedded in HTML  
**EMBRACE:** Interactive web components with Plotly.js/Recharts

**Visual Identity:**
- **Background:** Dark charcoal (#0d0d0d to #1a1a2e gradient)
- **Primary Text:** Warm sand (#f0ece4)
- **Accent (Positive):** Terminal green (#00d4aa)
- **Accent (Negative):** Desaturated crimson (#ff4444)
- **Accent (Warning):** Muted amber (#ffa500)

**Typography:**
- **Headings:** DM Serif Display or Playfair Display (serif, editorial)
- **Body:** Inter or system-ui (clean, readable)
- **Data/Metrics:** JetBrains Mono or IBM Plex Mono (monospace, precise)

**Layout:**
- Single-column editorial flow (max-width: 800px)
- NO Bootstrap-style card grids
- NO predictable hero sections
- Whitespace as structural element
- Micro-interactions on hover

### 5a. Interactive Chart Architecture

```python
def create_interactive_charts(
    price_df: pd.DataFrame,
    events_df: pd.DataFrame,
    strategy_results: dict,
    valuation: dict,
    risk: dict,
    ticker: str
) -> dict:
    """
    Create interactive Plotly charts for web embedding.
    
    Returns dict of chart HTML strings (ready for embedding):
    {
        'price_chart': str,
        'cumulative_returns': str,
        'earnings_returns': str,
        'surprise_scatter': str,
        'valuation_bars': str,
        'strategy_comparison': str,
        'volume_analysis': str,
        'volatility_chart': str
    }
    """
    import plotly.graph_objects as go
    from plotly.subplots import make_subplots
    import plotly.express as px
    
    charts = {}
    
    # Chart 1: Price + Moving Averages + Earnings Markers
    fig = go.Figure()
    
    fig.add_trace(go.Scatter(
        x=price_df.index,
        y=price_df['Close'],
        name='Close Price',
        line=dict(color='#00d4aa', width=2),
        hovertemplate='Date: %{x}<br>Price: $%{y:.2f}<extra></extra>'
    ))
    
    if 'ma_50' in price_df.columns:
        fig.add_trace(go.Scatter(
            x=price_df.index,
            y=price_df['ma_50'],
            name='50-Day MA',
            line=dict(color='#ffa500', width=1, dash='dash'),
            opacity=0.7
        ))
    
    if 'ma_200' in price_df.columns:
        fig.add_trace(go.Scatter(
            x=price_df.index,
            y=price_df['ma_200'],
            name='200-Day MA',
            line=dict(color='#ff6b6b', width=1, dash='dot'),
            opacity=0.7
        ))
    
    # Add earnings markers
    for _, event in events_df.iterrows():
        color = '#00ff88' if event['return_1d'] > 0 else '#ff4444'
        fig.add_vline(
            x=event['earnings_date'],
            line=dict(color=color, width=1, dash='dash'),
            opacity=0.5
        )
    
    fig.update_layout(
        title=f'{ticker} Price & Earnings Events',
        xaxis_title='Date',
        yaxis_title='Price ($)',
        template='plotly_dark',
        hovermode='x unified',
        height=500,
        font=dict(family='DM Serif Display, serif', size=14),
        paper_bgcolor='#1a1a2e',
        plot_bgcolor='#0d0d0d'
    )
    
    charts['price_chart'] = fig.to_html(full_html=False, include_plotlyjs='cdn')
    
    # Chart 2: Cumulative Returns (Stock vs Benchmark)
    fig = go.Figure()
    
    fig.add_trace(go.Scatter(
        x=price_df.index,
        y=price_df['cumulative_return'] * 100,
        name=ticker,
        line=dict(color='#00d4aa', width=2)
    ))
    
    if 'cumulative_benchmark' in price_df.columns:
        fig.add_trace(go.Scatter(
            x=price_df.index,
            y=price_df['cumulative_benchmark'] * 100,
            name='Benchmark (SPY)',
            line=dict(color='#666', width=2, dash='dash')
        ))
    
    fig.update_layout(
        title='Cumulative Returns',
        xaxis_title='Date',
        yaxis_title='Return (%)',
        template='plotly_dark',
        height=400,
        paper_bgcolor='#1a1a2e',
        plot_bgcolor='#0d0d0d'
    )
    
    charts['cumulative_returns'] = fig.to_html(full_html=False, include_plotlyjs='cdn')
    
    # Chart 3: Earnings-Day Returns (Bar Chart)
    fig = go.Figure()
    
    colors = ['#00d4aa' if r > 0 else '#ff4444' for r in events_df['return_1d']]
    
    fig.add_trace(go.Bar(
        x=events_df['earnings_date'].dt.strftime('%Y-%m-%d'),
        y=events_df['return_1d'] * 100,
        marker_color=colors,
        text=[f'{r:+.2f}%' for r in events_df['return_1d']],
        textposition='auto',
        hovertemplate='Date: %{x}<br>Return: %{y:.2f}%<extra></extra>'
    ))
    
    fig.update_layout(
        title='1-Day Post-Earnings Returns',
        xaxis_title='Earnings Date',
        yaxis_title='Return (%)',
        template='plotly_dark',
        height=400,
        paper_bgcolor='#1a1a2e',
        plot_bgcolor='#0d0d0d'
    )
    
    charts['earnings_returns'] = fig.to_html(full_html=False, include_plotlyjs='cdn')
    
    # Chart 4: EPS Surprise vs Return (Scatter + Regression)
    clean = events_df.dropna(subset=['eps_surprise_pct', 'return_1d'])
    
    fig = px.scatter(
        clean,
        x='eps_surprise_pct',
        y='return_1d',
        color='eps_surprise_pct',
        color_continuous_scale='RdYlGn',
        title='EPS Surprise vs 1-Day Return',
        labels={'eps_surprise_pct': 'EPS Surprise (%)', 'return_1d': '1-Day Return (%)'}
    )
    
    # Add regression line
    if len(clean) >= 3:
        from scipy import stats
        slope, intercept, r_value, p_value, std_err = stats.linregress(
            clean['eps_surprise_pct'],
            clean['return_1d']
        )
        x_range = [clean['eps_surprise_pct'].min(), clean['eps_surprise_pct'].max()]
        y_range = [slope * x + intercept for x in x_range]
        
        fig.add_trace(go.Scatter(
            x=x_range,
            y=y_range,
            mode='lines',
            name=f'Trend (R²={r_value**2:.3f})',
            line=dict(color='blue', dash='dash')
        ))
    
    fig.update_layout(
        template='plotly_dark',
        height=450,
        paper_bgcolor='#1a1a2e',
        plot_bgcolor='#0d0d0d'
    )
    
    charts['surprise_scatter'] = fig.to_html(full_html=False, include_plotlyjs='cdn')
    
    # Chart 5: Valuation Comparison (Bar Chart)
    categories = ['Trailing P/E', 'Forward P/E']
    stock_vals = [valuation['trailing_pe'], valuation['forward_pe']]
    sector_vals = [valuation['sector_pe'], valuation['sector_pe']]
    index_vals = [valuation['index_pe'], valuation['index_pe']]
    
    fig = go.Figure()
    
    fig.add_trace(go.Bar(
        name=ticker,
        x=categories,
        y=stock_vals,
        marker_color='#00d4aa',
        text=[f'{v:.1f}x' for v in stock_vals],
        textposition='auto'
    ))
    
    fig.add_trace(go.Bar(
        name='Tech Sector',
        x=categories,
        y=sector_vals,
        marker_color='#ffa500',
        text=[f'{v:.1f}x' for v in sector_vals],
        textposition='auto'
    ))
    
    fig.add_trace(go.Bar(
        name='S&P 500',
        x=categories,
        y=index_vals,
        marker_color='#666',
        text=[f'{v:.1f}x' for v in index_vals],
        textposition='auto'
    ))
    
    fig.update_layout(
        title='Valuation Comparison',
        xaxis_title='Metric',
        yaxis_title='P/E Ratio',
        barmode='group',
        template='plotly_dark',
        height=450,
        paper_bgcolor='#1a1a2e',
        plot_bgcolor='#0d0d0d'
    )
    
    charts['valuation_bars'] = fig.to_html(full_html=False, include_plotlyjs='cdn')
    
    # Chart 6: Strategy Backtest Comparison
    trades_df = strategy_results['trades']
    
    # Calculate cumulative strategy returns
    strategy_cum = (1 + trades_df['return']).cumprod() - 1
    
    fig = go.Figure()
    
    fig.add_trace(go.Scatter(
        x=trades_df['buy_date'],
        y=strategy_cum * 100,
        name='Earnings Strategy',
        mode='lines+markers',
        line=dict(color='#00d4aa', width=2)
    ))
    
    # Buy-and-hold line
    bh_return = strategy_results['buyhold_cumulative_return'] * 100
    fig.add_trace(go.Scatter(
        x=[trades_df['buy_date'].min(), trades_df['sell_date'].max()],
        y=[0, bh_return],
        name='Buy & Hold',
        mode='lines',
        line=dict(color='#ff6b6b', width=2, dash='dash')
    ))
    
    fig.update_layout(
        title='Strategy Performance: Earnings Timing vs Buy & Hold',
        xaxis_title='Date',
        yaxis_title='Cumulative Return (%)',
        template='plotly_dark',
        height=450,
        paper_bgcolor='#1a1a2e',
        plot_bgcolor='#0d0d0d'
    )
    
    charts['strategy_comparison'] = fig.to_html(full_html=False, include_plotlyjs='cdn')
    
    # Chart 7: Volume Around Earnings
    avg_volume = price_df['Volume'].mean()
    earnings_volumes = events_df['volume_day_of'].tolist()
    
    fig = go.Figure()
    
    fig.add_trace(go.Bar(
        x=['Average Daily', 'Earnings Day'],
        y=[avg_volume, np.mean(earnings_volumes)],
        marker_color=['#666', '#00d4aa'],
        text=[f'{avg_volume/1e6:.1f}M', f'{np.mean(earnings_volumes)/1e6:.1f}M'],
        textposition='auto'
    ))
    
    fig.update_layout(
        title='Trading Volume: Normal vs Earnings Days',
        xaxis_title='Period',
        yaxis_title='Volume',
        template='plotly_dark',
        height=400,
        paper_bgcolor='#1a1a2e',
        plot_bgcolor='#0d0d0d'
    )
    
    charts['volume_analysis'] = fig.to_html(full_html=False, include_plotlyjs='cdn')
    
    # Chart 8: Rolling Volatility
    if 'volatility_20d' in price_df.columns:
        fig = go.Figure()
        
        fig.add_trace(go.Scatter(
            x=price_df.index,
            y=price_df['volatility_20d'] * 100,
            name='20-Day Rolling Volatility (Annualized)',
            line=dict(color='#ffa500', width=2)
        ))
        
        fig.update_layout(
            title='Stock Volatility',
            xaxis_title='Date',
            yaxis_title='Volatility (%)',
            template='plotly_dark',
            height=400,
            paper_bgcolor='#1a1a2e',
            plot_bgcolor='#0d0d0d'
        )
        
        charts['volatility_chart'] = fig.to_html(full_html=False, include_plotlyjs='cdn')
    
    return charts
```

---

## 6. Phase 5: Report Generation

### 6a. HTML Report Template (Editorial Aesthetic)

```python
def generate_html_report(
    ticker: str,
    event_study: dict,
    strategy_results: dict,
    valuation: dict,
    risk: dict,
    charts: dict,
    industry_context: dict,
    assessment: dict
) -> str:
    """
    Generate self-contained HTML report with editorial/data-journalism aesthetic.
    
    Design:
    - Dark charcoal background (#0d0d0d)
    - Warm sand text (#f0ece4)
    - Terminal green accent (#00d4aa)
    - Single-column editorial flow
    - Typography: DM Serif Display (headings) + JetBrains Mono (data)
    """
    
    html_template = f"""
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>{ticker} Earnings & Valuation Analysis</title>
        
        <!-- Fonts -->
        <link rel="preconnect" href="https://fonts.googleapis.com">
        <link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display&family=JetBrains+Mono:wght@400;700&family=Inter:wght@400;600&display=swap" rel="stylesheet">
        
        <!-- Plotly.js -->
        <script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>
        
        <style>
            :root {{
                --bg-primary: #0d0d0d;
                --bg-secondary: #1a1a2e;
                --text-primary: #f0ece4;
                --text-secondary: #a0a0a0;
                --accent-positive: #00d4aa;
                --accent-negative: #ff4444;
                --accent-warning: #ffa500;
                --font-serif: 'DM Serif Display', serif;
                --font-mono: 'JetBrains Mono', monospace;
                --font-sans: 'Inter', sans-serif;
            }}
            
            * {{
                margin: 0;
                padding: 0;
                box-sizing: border-box;
            }}
            
            body {{
                font-family: var(--font-sans);
                background: linear-gradient(135deg, var(--bg-primary) 0%, var(--bg-secondary) 100%);
                color: var(--text-primary);
                line-height: 1.6;
                padding: 2rem;
            }}
            
            .container {{
                max-width: 1000px;
                margin: 0 auto;
            }}
            
            header {{
                text-align: center;
                padding: 3rem 0;
                border-bottom: 2px solid var(--accent-positive);
                margin-bottom: 3rem;
            }}
            
            h1 {{
                font-family: var(--font-serif);
                font-size: 3rem;
                margin-bottom: 0.5rem;
                color: var(--text-primary);
            }}
            
            .subtitle {{
                font-size: 1.25rem;
                color: var(--text-secondary);
                font-style: italic;
            }}
            
            .executive-summary {{
                background: rgba(0, 212, 170, 0.1);
                border-left: 4px solid var(--accent-positive);
                padding: 2rem;
                margin: 2rem 0;
                border-radius: 0 8px 8px 0;
            }}
            
            .metric-strip {{
                display: grid;
                grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
                gap: 1.5rem;
                margin: 3rem 0;
            }}
            
            .metric-card {{
                background: var(--bg-secondary);
                padding: 1.5rem;
                border-radius: 8px;
                text-align: center;
                border: 1px solid rgba(255, 255, 255, 0.1);
                transition: transform 0.2s ease;
            }}
            
            .metric-card:hover {{
                transform: translateY(-2px);
                box-shadow: 0 4px 12px rgba(0, 212, 170, 0.2);
            }}
            
            .metric-value {{
                font-family: var(--font-mono);
                font-size: 2.5rem;
                font-weight: 700;
                color: var(--accent-positive);
                margin: 0.5rem 0;
            }}
            
            .metric-label {{
                font-size: 0.875rem;
                color: var(--text-secondary);
                text-transform: uppercase;
                letter-spacing: 0.05em;
            }}
            
            .chart-container {{
                margin: 3rem 0;
                background: var(--bg-secondary);
                padding: 1.5rem;
                border-radius: 8px;
                border: 1px solid rgba(255, 255, 255, 0.1);
            }}
            
            .section {{
                margin: 3rem 0;
            }}
            
            h2 {{
                font-family: var(--font-serif);
                font-size: 2rem;
                margin-bottom: 1.5rem;
                color: var(--text-primary);
                border-bottom: 1px solid rgba(255, 255, 255, 0.1);
                padding-bottom: 0.5rem;
            }}
            
            h3 {{
                font-family: var(--font-serif);
                font-size: 1.5rem;
                margin: 2rem 0 1rem;
                color: var(--accent-positive);
            }}
            
            .insight-box {{
                background: rgba(255, 165, 0, 0.1);
                border-left: 4px solid var(--accent-warning);
                padding: 1.5rem;
                margin: 1.5rem 0;
                border-radius: 0 8px 8px 0;
            }}
            
            .insight-box strong {{
                color: var(--accent-warning);
                display: block;
                margin-bottom: 0.5rem;
                font-family: var(--font-mono);
                text-transform: uppercase;
                font-size: 0.875rem;
                letter-spacing: 0.05em;
            }}
            
            table {{
                width: 100%;
                border-collapse: collapse;
                margin: 1.5rem 0;
                background: var(--bg-secondary);
                border-radius: 8px;
                overflow: hidden;
            }}
            
            th, td {{
                padding: 1rem;
                text-align: left;
                border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            }}
            
            th {{
                background: rgba(0, 212, 170, 0.2);
                font-family: var(--font-mono);
                text-transform: uppercase;
                font-size: 0.875rem;
                letter-spacing: 0.05em;
                color: var(--accent-positive);
            }}
            
            td {{
                font-family: var(--font-mono);
            }}
            
            .positive {{
                color: var(--accent-positive);
                font-weight: 700;
            }}
            
            .negative {{
                color: var(--accent-negative);
                font-weight: 700;
            }}
            
            .recommendation {{
                background: rgba(0, 212, 170, 0.1);
                border: 2px solid var(--accent-positive);
                padding: 2rem;
                margin: 2rem 0;
                border-radius: 8px;
            }}
            
            .recommendation h3 {{
                margin-top: 0;
                color: var(--accent-positive);
            }}
            
            .disclaimer {{
                background: rgba(255, 68, 68, 0.1);
                border-left: 4px solid var(--accent-negative);
                padding: 1.5rem;
                margin-top: 4rem;
                border-radius: 0 8px 8px 0;
                font-size: 0.875rem;
                color: var(--text-secondary);
            }}
            
            footer {{
                text-align: center;
                padding: 3rem 0;
                margin-top: 4rem;
                border-top: 1px solid rgba(255, 255, 255, 0.1);
                color: var(--text-secondary);
                font-size: 0.875rem;
            }}
            
            @media (max-width: 768px) {{
                h1 {{ font-size: 2rem; }}
                .metric-strip {{ grid-template-columns: 1fr; }}
                body {{ padding: 1rem; }}
            }}
        </style>
    </head>
    <body>
        <div class="container">
            <header>
                <h1>{ticker} Analysis</h1>
                <p class="subtitle">Earnings Impact & Sustainable Opportunity Assessment</p>
                <p style="color: var(--text-secondary); margin-top: 1rem;">
                    Generated on {datetime.now().strftime('%B %d, %Y')}
                </p>
            </header>
            
            <div class="executive-summary">
                <h2 style="margin-top: 0; font-family: var(--font-serif);">Executive Summary</h2>
                <p style="font-size: 1.125rem; line-height: 1.8;">
                    {assessment.get('overall_verdict', 'Analysis in progress').capitalize()}. 
                    {ticker} shows {event_study.get('positive_rate', 0)*100:.0f}% positive earnings reactions 
                    with an average 1-day return of {event_study.get('mean_1d_return', 0)*100:+.2f}%. 
                    The earnings-timing strategy {'underperforms' if strategy_results.get('underperformance', 0) < 0 else 'outperforms'} 
                    buy-and-hold by {strategy_results.get('underperformance', 0)*100:+.2f}%, 
                    suggesting {'long-term holding is superior' if strategy_results.get('underperformance', 0) < 0 else 'active timing adds value'}.
                </p>
            </div>
            
            <div class="metric-strip">
                <div class="metric-card">
                    <div class="metric-label">Total Return</div>
                    <div class="metric-value">{risk.get('total_return', 0)*100:+.1f}%</div>
                </div>
                <div class="metric-card">
                    <div class="metric-label">Sharpe Ratio</div>
                    <div class="metric-value">{risk.get('sharpe_ratio', 0):.2f}</div>
                </div>
                <div class="metric-card">
                    <div class="metric-label">Max Drawdown</div>
                    <div class="metric-value" style="color: var(--accent-negative);">{risk.get('max_drawdown', 0)*100:.1f}%</div>
                </div>
                <div class="metric-card">
                    <div class="metric-label">P/E Ratio</div>
                    <div class="metric-value">{valuation.get('trailing_pe', 0):.1f}x</div>
                </div>
                <div class="metric-card">
                    <div class="metric-label">Win Rate</div>
                    <div class="metric-value">{strategy_results.get('win_rate', 0)*100:.0f}%</div>
                </div>
                <div class="metric-card">
                    <div class="metric-label">Events</div>
                    <div class="metric-value">{event_study.get('total_events', 0)}</div>
                </div>
            </div>
            
            <div class="section">
                <h2>Price Performance & Earnings Events</h2>
                <div class="chart-container">
                    {charts.get('price_chart', '<p>Chart loading...</p>')}
                </div>
                
                <div class="insight-box">
                    <strong>Key Insight</strong>
                    <p>
                        While {ticker} consistently beats earnings expectations, 
                        {'the stock\'s massive gains are driven more by long-term AI infrastructure demand and forward-looking revenue growth than quarterly earnings surprises.' if 'NVDA' in ticker else 'price movements are influenced by broader market dynamics and company-specific catalysts.'}
                        The real opportunity lies in {'the long-term transformation' if 'NVDA' in ticker else 'sustained fundamental growth'}, not short-term earnings trading.
                    </p>
                </div>
            </div>
            
            <div class="section">
                <h2>Valuation Analysis</h2>
                <div class="chart-container">
                    {charts.get('valuation_bars', '<p>Chart loading...</p>')}
                </div>
                
                <table>
                    <thead>
                        <tr>
                            <th>Metric</th>
                            <th>{ticker}</th>
                            <th>Tech Sector</th>
                            <th>S&P 500</th>
                            <th>Premium/Discount</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>Trailing P/E</td>
                            <td>{valuation.get('trailing_pe', 'N/A'):.1f}x</td>
                            <td>{valuation.get('sector_pe', 'N/A'):.1f}x</td>
                            <td>{valuation.get('index_pe', 'N/A'):.1f}x</td>
                            <td class="{'positive' if valuation.get('trailing_premium_vs_sector', 0) < 0 else 'negative'}">
                                {valuation.get('trailing_premium_vs_sector', 0):+.1f}%
                            </td>
                        </tr>
                        <tr>
                            <td>Forward P/E</td>
                            <td>{valuation.get('forward_pe', 'N/A'):.1f}x</td>
                            <td>{valuation.get('sector_pe', 'N/A'):.1f}x</td>
                            <td>{valuation.get('index_pe', 'N/A'):.1f}x</td>
                            <td class="positive">
                                {valuation.get('forward_premium_vs_sector', 0):+.1f}%
                            </td>
                        </tr>
                    </tbody>
                </table>
                
                <div class="insight-box">
                    <strong>Valuation Insight</strong>
                    <p>
                        {valuation.get('valuation_note', 'Forward P/E analysis not available.')}
                        {'The AI infrastructure buildout provides a multi-year growth runway that supports current multiples.' if 'NVDA' in ticker else ''}
                    </p>
                </div>
            </div>
            
            <div class="section">
                <h2>Trading Strategy Performance</h2>
                <p style="margin-bottom: 1.5rem;">
                    We simulated a strategy of buying {ticker} stock one week before earnings 
                    and selling one week after each announcement.
                </p>
                
                <div class="chart-container">
                    {charts.get('strategy_comparison', '<p>Chart loading...</p>')}
                </div>
                
                <table>
                    <thead>
                        <tr>
                            <th>Metric</th>
                            <th>Earnings Strategy</th>
                            <th>Buy & Hold</th>
                            <th>Difference</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>Total Return</td>
                            <td class="{'positive' if strategy_results.get('strategy_cumulative_return', 0) > 0 else 'negative'}">
                                {strategy_results.get('strategy_cumulative_return', 0)*100:+.2f}%
                            </td>
                            <td class="positive">
                                {strategy_results.get('buyhold_cumulative_return', 0)*100:+.2f}%
                            </td>
                            <td class="{'positive' if strategy_results.get('underperformance', 0) > 0 else 'negative'}">
                                {strategy_results.get('underperformance', 0)*100:+.2f}%
                            </td>
                        </tr>
                        <tr>
                            <td>Annualized Return</td>
                            <td>{strategy_results.get('strategy_annualized_return', 0)*100:+.2f}%</td>
                            <td>{strategy_results.get('buyhold_annualized_return', 0)*100:+.2f}%</td>
                            <td>-</td>
                        </tr>
                        <tr>
                            <td>Sharpe Ratio</td>
                            <td>{strategy_results.get('sharpe_ratio', 0):.2f}</td>
                            <td>{risk.get('sharpe_ratio', 0):.2f}</td>
                            <td>-</td>
                        </tr>
                        <tr>
                            <td>Time in Market</td>
                            <td>{strategy_results.get('time_in_market', 0)*100:.1f}%</td>
                            <td>100%</td>
                            <td>-</td>
                        </tr>
                        <tr>
                            <td>Win Rate</td>
                            <td>{strategy_results.get('win_rate', 0)*100:.1f}%</td>
                            <td>100%</td>
                            <td class="negative">
                                -{100 - strategy_results.get('win_rate', 0)*100:.1f}%
                            </td>
                        </tr>
                        <tr>
                            <td>Final Value ($10K)</td>
                            <td>${strategy_results.get('strategy_final_value', 0):,.0f}</td>
                            <td>${strategy_results.get('buyhold_final_value', 0):,.0f}</td>
                            <td class="{'positive' if strategy_results.get('underperformance', 0) > 0 else 'negative'}">
                                ${strategy_results.get('underperformance', 0)*10000:+,.0f}
                            </td>
                        </tr>
                    </tbody>
                </table>
                
                <div class="insight-box">
                    <strong>Strategy Insight</strong>
                    <p>
                        The earnings-based trading strategy {'failed because it missed the sustained upward momentum between earnings announcements. ' if strategy_results.get('underperformance', 0) < 0 else 'successfully captured earnings-related volatility. '}
                        {ticker}'s gains are driven by {'long-term AI adoption trends' if 'NVDA' in ticker else 'fundamental growth'}, not quarterly earnings volatility. 
                        Missing just a few weeks of the rally resulted in {'massive underperformance.' if strategy_results.get('underperformance', 0) < 0 else 'modest outperformance.'}
                    </p>
                </div>
            </div>
            
            <div class="section">
                <h2>Investment Recommendations</h2>
                
                <div class="recommendation">
                    <h3>For Long-Term Investors (3+ years)</h3>
                    <p><strong>RECOMMENDATION: {'BUY' if assessment.get('overall_verdict') == 'Sustainable' else 'HOLD'}</strong></p>
                    <ul style="margin-top: 1rem; padding-left: 1.5rem;">
                        <li>AI infrastructure buildout is in early innings</li>
                        <li>{ticker}'s technological moat remains strong</li>
                        <li>Valuation becomes more reasonable on forward metrics</li>
                        <li>Dollar-cost averaging recommended due to volatility</li>
                    </ul>
                </div>
                
                <div class="recommendation" style="background: rgba(255, 165, 0, 0.1); border-color: var(--accent-warning);">
                    <h3 style="color: var(--accent-warning);">For Short-Term Traders</h3>
                    <p><strong>RECOMMENDATION: AVOID EARNINGS TIMING</strong></p>
                    <ul style="margin-top: 1rem; padding-left: 1.5rem;">
                        <li>Earnings-based strategies {'significantly underperform' if strategy_results.get('underperformance', 0) < 0 else 'show mixed results'}</li>
                        <li>High volatility creates timing challenges</li>
                        <li>Focus on longer-term trends rather than quarterly events</li>
                    </ul>
                </div>
                
                <div class="recommendation" style="background: rgba(0, 212, 170, 0.05); border-color: var(--text-secondary);">
                    <h3 style="color: var(--text-secondary);">Portfolio Allocation</h3>
                    <p><strong>RECOMMENDATION: 5-10% POSITION SIZE</strong></p>
                    <ul style="margin-top: 1rem; padding-left: 1.5rem;">
                        <li>Large enough to benefit from {'AI growth' if 'NVDA' in ticker else 'sector growth'}</li>
                        <li>Small enough to manage concentration risk</li>
                        <li>Consider pairing with broader tech ETF exposure</li>
                    </ul>
                </div>
            </div>
            
            <div class="section">
                <h2>Methodology</h2>
                <ul style="padding-left: 1.5rem; line-height: 2;">
                    <li><strong>Data Sources:</strong> Yahoo Finance API, SEC filings, earnings calendars</li>
                    <li><strong>Analysis Period:</strong> {event_study.get('total_events', 0) * 3 / 12:.1f} years ({event_study.get('total_events', 0)} earnings events)</li>
                    <li><strong>Strategy Simulation:</strong> Buy 7 days before earnings, sell 7 days after (using closest trading days)</li>
                    <li><strong>Statistical Methods:</strong> Pearson correlation, linear regression, rolling volatility</li>
                    <li><strong>Risk Metrics:</strong> Sharpe ratio, Sortino ratio, Calmar ratio, maximum drawdown</li>
                    <li><strong>P/E Comparisons:</strong> Based on trailing twelve months earnings and current market data</li>
                </ul>
            </div>
            
            <div class="disclaimer">
                <strong>Important Disclaimer:</strong> This analysis is for educational purposes only and should not be considered as investment advice. 
                Past performance does not guarantee future results. All investments carry risk of loss. 
                Consult with a qualified financial advisor before making investment decisions. 
                The AI market is subject to high volatility and regulatory uncertainty.
            </div>
            
            <footer>
                <p>{ticker} Earnings & Valuation Analysis | Generated on {datetime.now().strftime('%B %d, %Y')}</p>
                <p style="margin-top: 0.5rem;">Data-driven insights for everyday investors</p>
            </footer>
        </div>
    </body>
    </html>
    """
    
    return html_template
```

---

## 7. Phase 6: Validation & QA

### 7a. Pre-Delivery Checks

```python
def run_qa_checks(
    price_df: pd.DataFrame,
    events_df: pd.DataFrame,
    strategy_results: dict,
    valuation: dict,
    report_html: str,
    chart_files: dict
) -> list[str]:
    """
    Final quality assurance. Return list of issues found.
    
    Checks:
    - Data integrity (gaps, volume, EPS)
    - Statistical validity (sample size)
    - Strategy sanity (unrealistic returns)
    - Valuation sanity (extreme P/E)
    - File existence (charts, HTML)
    - HTML structure validity
    """
    issues = []
    
    # Data integrity
    if len(price_df) < 252:
        issues.append("WARNING: Less than 1 year of price data.")
    
    if len(events_df) < 4:
        issues.append("WARNING: Fewer than 4 earnings events. Statistical conclusions unreliable.")
    
    # Strategy sanity
    if strategy_results['strategy_cumulative_return'] > 10:
        issues.append("ERROR: Strategy return > 1000%. Likely a calculation bug.")
    
    if abs(strategy_results['buyhold_cumulative_return']) > 20:
        issues.append("WARNING: Buy-and-hold return > 2000%. Verify data period.")
    
    # Valuation sanity
    if valuation.get('trailing_pe') and valuation['trailing_pe'] > 500:
        issues.append("WARNING: P/E > 500x. Verify data or note that earnings are near-zero.")
    
    # File existence
    for chart_name, chart_path in chart_files.items():
        if not os.path.exists(chart_path):
            issues.append(f"ERROR: Chart file not found at {chart_path}")
    
    if '<html' not in report_html.lower():
        issues.append("ERROR: Report HTML appears malformed.")
    
    return issues
```

### 7b. Statistical Validity Checks

```python
def check_statistical_validity(events_df: pd.DataFrame) -> dict:
    """
    Verify that statistical claims in the report are supportable.
    
    Returns:
        {
            'sample_size': int,
            'sample_adequate': bool (>= 8 events),
            'correlation_reliable': bool (>= 10 events),
            'can_compute_regression': bool (>= 5 events),
            'confidence_note': str
        }
    """
    n = len(events_df)
    
    return {
        'sample_size': n,
        'sample_adequate': n >= 8,  # At least 2 years of quarterly earnings
        'correlation_reliable': n >= 10,  # Minimum for meaningful Pearson correlation
        'can_compute_regression': n >= 5,
        'confidence_note': (
            f"Based on {n} earnings events over "
            f"~{n * 3 / 12:.1f} years. "
            + ("Sample size is adequate for statistical inference." if n >= 8
               else "CAUTION: Small sample. Treat correlations as directional, not definitive.")
        )
    }
```

---

## 8. Edge Cases, Pitfalls & Augmented Insights

### Critical Pitfalls (With Fixes)

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| **Timezone mismatch** | Empty merge results, silent failures | `tz_localize(None)` IMMEDIATELY after every yfinance fetch |
| **Future earnings dates** | NaN in analysis, inflated counts | `dropna(subset=['Reported EPS'])` |
| **Unfair backtest comparison** | Misleading underperformance % | Compute Sharpe, annualized returns, time-in-market |
| **Missing risk metrics** | Incomplete investment thesis | Always include Sharpe, Sortino, Calmar, max DD |
| **No significance testing** | False confidence in correlations | Report p-value alongside every correlation |
| **Look-ahead bias** | Using today's P/E for historical decisions | Use point-in-time data only |
| **Forward P/E trap** | Assuming growth will materialize | Compare historical estimate accuracy |
| **Macro regime blindness** | Conclusions valid only in one market regime | Note rate environment, sector rotation |

### Augmented Insights

1. **Revenue Surprise Matters**: Always check revenue surprise alongside EPS. Revenue is harder to manage than EPS and provides a stronger signal of fundamental health.

2. **Short Interest & Options Activity**: 
   - Check short interest ratio before earnings (squeeze potential)
   - Compare implied volatility vs. realized volatility post-earnings
   - Options market expected move vs. actual move
   - These explain why stocks sometimes move opposite to earnings surprise (the surprise was already priced in)

3. **Macro Regime Awareness**:
   - 2023-2025 was an extraordinary AI bull market
   - The same analysis of NVDA in 2018-2020 would show a very different picture
   - Always note: rate environment, risk appetite, sector rotation

4. **The "Beating by Pennies" Problem**:
   - Most companies consistently beat EPS estimates by small margins (managed expectations)
   - A +14% surprise sounds impressive, but check:
     - How does it compare to the sector average surprise?
     - Is the beat growing or shrinking over time?
     - Are revenue surprises also positive?

5. **Dynamic Data Sourcing**:
   - Never hardcode sector P/E ratios
   - Always use web search or API to fetch current benchmarks
   - Cite source and retrieval date

---

## 9. Reusable Code Templates

### 9a. Full Pipeline Orchestration

```python
def run_full_analysis(
    ticker: str,
    years: int = 2,
    sector_pe: float = None,
    index_pe: float = 22.5,
    output_dir: str = "output"
) -> dict:
    """
    Complete pipeline from data acquisition to report generation.
    
    Usage:
        results = run_full_analysis("NVDA", years=2)
    
    Returns:
        {
            'price_df': DataFrame,
            'events_df': DataFrame,
            'event_study': dict,
            'strategy': dict,
            'valuation': dict,
            'risk': dict,
            'charts': dict,
            'report_html': str,
            'statistical_validity': dict
        }
    """
    os.makedirs(output_dir, exist_ok=True)
    
    # Phase 1: Data Acquisition
    print(f"[1/6] Fetching data for {ticker}...")
    price_df = fetch_price_history(ticker, years)
    stock_info = fetch_stock_info(ticker)
    earnings_hist, earnings_dates = fetch_earnings_data(ticker)
    benchmark_df = fetch_benchmark("SPY", years)
    
    # Phase 2: Processing
    print("[2/6] Processing data...")
    warnings = validate_data(price_df, earnings_dates)
    if warnings:
        for w in warnings:
            print(f"  ⚠ {w}")
    
    price_df = compute_derived_metrics(price_df, benchmark_df)
    events_df = align_earnings_to_prices(earnings_dates, price_df)
    
    # Phase 3: Analysis
    print("[3/6] Running analyses...")
    event_study = earnings_event_study(events_df)
    strategy = backtest_earnings_strategy(price_df, earnings_dates)
    
    # Dynamic sector P/E sourcing
    if sector_pe is None:
        sector_data = get_sector_pe_via_web_search(
            stock_info.get('sector', 'Technology'), 
            datetime.now().year
        )
        sector_pe = sector_data.get('sector_pe', 39.1)
    
    valuation = valuation_analysis(stock_info, sector_pe, index_pe)
    risk = risk_metrics(price_df, benchmark_df)
    
    # Phase 4: Visualization
    print("[4/6] Creating interactive charts...")
    charts = create_interactive_charts(
        price_df, events_df, strategy, valuation, risk, ticker
    )
    
    # Save individual charts if needed
    chart_files = {}
    for chart_name, chart_html in charts.items():
        chart_path = os.path.join(output_dir, f"{ticker}_{chart_name}.html")
        with open(chart_path, 'w') as f:
            f.write(chart_html)
        chart_files[chart_name] = chart_path
    
    # Phase 5: Report Generation
    print("[5/6] Generating HTML report...")
    industry_context = extract_industry_context(
        stock_info.get('shortName', ticker),
        stock_info.get('sector', 'Technology'),
        datetime.now().year
    )
    
    assessment = synthesize_fundamental_assessment(
        event_study, strategy, valuation, risk, industry_context
    )
    
    report_html = generate_html_report(
        ticker, event_study, strategy, valuation, risk, 
        charts, industry_context, assessment
    )
    
    report_path = os.path.join(output_dir, f"{ticker}_analysis.html")
    with open(report_path, 'w', encoding='utf-8') as f:
        f.write(report_html)
    
    # Phase 6: Validation
    print("[6/6] Running QA checks...")
    stat_validity = check_statistical_validity(events_df)
    qa_issues = run_qa_checks(
        price_df, events_df, strategy, valuation, report_html, chart_files
    )
    
    if qa_issues:
        print("  ⚠ QA Issues found:")
        for issue in qa_issues:
            print(f"    - {issue}")
    else:
        print("  ✓ All QA checks passed")
    
    # Save data files
    price_df.to_csv(os.path.join(output_dir, f"{ticker}_prices.csv"))
    events_df.to_csv(os.path.join(output_dir, f"{ticker}_earnings_events.csv"))
    strategy['trades'].to_csv(os.path.join(output_dir, f"{ticker}_strategy_trades.csv"))
    
    return {
        'price_df': price_df,
        'events_df': events_df,
        'event_study': event_study,
        'strategy': strategy,
        'valuation': valuation,
        'risk': risk,
        'charts': charts,
        'report_html': report_html,
        'statistical_validity': stat_validity,
        'qa_issues': qa_issues,
        'output_files': {
            'report': report_path,
            'charts': chart_files,
            'data': os.path.join(output_dir, f"{ticker}_prices.csv")
        }
    }
```

### 9b. Quick Single-Ticker Check

```python
def quick_valuation_check(ticker: str) -> None:
    """
    Minimal check: current P/E vs sector. Useful for screening.
    """
    info = fetch_stock_info(ticker)
    pe = info.get('trailingPE')
    fwd_pe = info.get('forwardPE')
    
    print(f"\n{ticker} Quick Valuation Check")
    print("=" * 40)
    print(f"Trailing P/E:  {pe:.1f}x" if pe else "Trailing P/E:  N/A")
    print(f"Forward P/E:   {fwd_pe:.1f}x" if fwd_pe else "Forward P/E:   N/A")
    print(f"Sector:        {info.get('sector', 'N/A')}")
    print(f"Market Cap:    ${info.get('marketCap', 0)/1e9:.1f}B" if info.get('marketCap') else "Market Cap:    N/A")
    print()
```

---

## 10. Quality Checklist

### Before delivering any analysis, verify:

**Data Integrity**
- [ ] Timezone normalized for all yfinance data (`tz_localize(None)`)
- [ ] No future earnings dates included (`dropna(subset=['Reported EPS'])`)
- [ ] No duplicate dates in price DataFrame
- [ ] No NaN in critical columns (Close, Volume, EPS)
- [ ] At least 252 trading days of data

**Statistical Validity**
- [ ] At least 8 earnings events (4+ for annual analysis)
- [ ] Every correlation has p-value reported
- [ ] Sample size noted in all statistical claims
- [ ] Confidence intervals computed where appropriate

**Backtesting Fairness**
- [ ] Time-in-market calculated and reported
- [ ] Annualized returns computed (not just cumulative)
- [ ] Sharpe ratio calculated
- [ ] Sortino ratio calculated
- [ ] Calmar ratio calculated
- [ ] Maximum drawdown reported

**Valuation Accuracy**
- [ ] Sector P/E sourced dynamically (not hardcoded)
- [ ] Source and date cited for all benchmarks
- [ ] Forward P/E vs trailing P/E comparison included
- [ ] Implied growth rate calculated if forward < trailing

**Visualization Quality**
- [ ] All 8 interactive charts rendered
- [ ] Charts use Plotly.js (not static PNGs)
- [ ] Dark theme with proper color palette
- [ ] Typography: DM Serif Display + JetBrains Mono
- [ ] Responsive design (mobile-friendly)
- [ ] Hover states and tooltips functional

**Report Completeness**
- [ ] Executive summary with key insight
- [ ] Metric strip with 6 key numbers
- [ ] All narrative sections present
- [ ] Investment recommendations by investor profile
- [ ] Methodology documented
- [ ] Disclaimer present
- [ ] HTML self-contained (no broken images/links)

**Edge Cases**
- [ ] Macro context noted (rate environment, sector trends)
- [ ] Revenue surprise examined (if available)
- [ ] Look-ahead bias prevented
- [ ] Survivorship bias acknowledged
- [ ] Warnings surfaced from validation

**Code Quality**
- [ ] Type hints used throughout
- [ ] Docstrings for all functions
- [ ] Error handling implemented
- [ ] No hardcoded values (use config)
- [ ] PEP 8 compliant

---

## Appendix: Agent Web Search Guidance

To obtain sector P/E and industry context, use the following search patterns:

```python
SEARCH_QUERIES = {
    "sector_pe": "S&P 500 {sector} sector trailing P/E ratio {year}",
    "ai_market_size": "global AI market size {year} IDC Gartner McKinsey",
    "adoption_rate": "enterprise AI adoption rate {year} survey statistics",
    "infrastructure_spending": "AI infrastructure spending forecast {year}",
    "competitive_landscape": "{company} market share vs competitors {year}"
}

def extract_numeric_value(text: str) -> float:
    """Extract numeric P/E or market size from text."""
    import re
    # Match patterns like "39.1x", "$500 billion", "44.8"
    patterns = [
        r'(\d+\.?\d*)\s*x',           # 39.1x
        r'\$(\d+\.?\d*)\s*(billion|B)', # $500 billion
        r'(\d+\.?\d*)\s*percent',     # 35 percent
    ]
    
    for pattern in patterns:
        match = re.search(pattern, text, re.IGNORECASE)
        if match:
            value = float(match.group(1))
            if 'billion' in text.lower() or 'B' in text:
                value *= 1e9
            return value
    
    return None
```

**Extraction Rules:**
1. Always note the source (e.g., "S&P Global", "IDC", "Gartner")
2. Always note the retrieval date
3. When numbers conflict, prefer the most recent official source
4. Cross-reference at least 2 sources for critical metrics

---

## 🎯 Final Deliverable Standards

This SKILL.md represents a **production-grade, avant-garde** approach to earnings analysis that:

✅ **Rejects generic aesthetics** (no Bootstrap grids, no Inter/Roboto safety)  
✅ **Embraces interactive visualizations** (Plotly.js, not static PNGs)  
✅ **Enforces technical rigor** (timezone normalization, fair backtesting)  
✅ **Demands statistical validity** (p-values, confidence intervals)  
✅ **Provides actionable insights** (investor-profile-specific recommendations)  
✅ **Maintains intellectual honesty** (acknowledges limitations, edge cases)  

**Use this skill to transform raw financial data into editorial-quality investment analysis that stands apart from generic AI-generated reports.**

---

*Version 3.0 | Last Updated: December 2025 | Domain: Quantitative Finance + Data Journalism + Avant-Garde UI/UX*

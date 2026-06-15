---
name: stock-analysis
description: >
  Earnings-Event Stock Analysis & AI Sector Valuation
  Guide an AI agent through a rigorous, multi-phase pipeline that analyzes a stock's price behavior around earnings events, benchmarks its valuation against sector/indices,
  backtests an earnings-timing strategy, and produces a comprehensive report answering: *"Is this stock's performance driven by sustainable fundamentals or short-term hype
  Applicable Tickers: Any publicly traded equity (demonstrated with NVDA).
  Time Horizon: 2+ years of historical data recommended.
  requires comfort with pandas, matplotlib, financial APIs.
version: 1.0
---

## Table of Contents

1. [Prerequisites & Environment](#1-prerequisites--environment)
2. [Phase 1: Data Acquisition](#2-phase-1-data-acquisition)
3. [Phase 2: Data Processing & Feature Engineering](#3-phase-2-data-processing--feature-engineering)
4. [Phase 3: Multi-Dimensional Analysis](#4-phase-3-multi-dimensional-analysis)
5. [Phase 4: Visualization](#5-phase-4-visualization)
6. [Phase 5: Report Generation](#6-phase-5-report-generation)
7. [Phase 6: Validation & QA](#7-phase-6-validation--qa)
8. [Edge Cases, Pitfalls & Augmented Insights](#8-edge-cases-pitfalls--augmented-insights)
9. [Reusable Code Templates](#9-reusable-code-templates)
10. [Quality Checklist](#10-quality-checklist)

---

## 1. Prerequisites & Environment

### Python Dependencies

```bash
pip install yfinance pandas numpy matplotlib seaborn scipy jinja2
```

| Library   | Purpose                                    |
|-----------|--------------------------------------------|
| `yfinance`| Historical prices, stock info, financials  |
| `pandas`  | Time series manipulation, alignment        |
| `numpy`   | Numerical computation, array operations    |
| `matplotlib` / `seaborn` | Charting and visualization     |
| `scipy`   | Statistical tests (t-test, correlation CI) |
| `jinja2`  | HTML report templating                     |

### Supplementary Data Sources (via web search)

| Data Need                 | Source                                              |
|---------------------------|-----------------------------------------------------|
| Earnings announcement dates | Yahoo Finance earnings calendar, SEC filings (10-Q/10-K) |
| Sector P/E averages       | S&P Global, Yardeni Research, GuruFocus             |
| Industry market size      | IDC, Gartner, McKinsey reports                      |
| Analyst consensus estimates | Yahoo Finance, Seeking Alpha, MarketBeat           |
| Regulatory/export news    | Reuters, Bloomberg, SEC EDGAR                       |

### Environment Checks

```python
import yfinance as yf
import pandas as pd
import numpy as np
from scipy import stats
import matplotlib
matplotlib.use('Agg')  # headless rendering for server environments
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
from datetime import datetime, timedelta
import json
import os

# Verify data access
test = yf.download("AAPL", period="5d", progress=False)
assert len(test) > 0, "Yahoo Finance API not accessible"
```

---

## 2. Phase 1: Data Acquisition

### 2a. Historical Price Data (OHLCV)

```python
def fetch_price_history(ticker: str, years: int = 2) -> pd.DataFrame:
    """
    Fetch OHLCV data for the given ticker.
    
    CRITICAL: yfinance returns timezone-aware datetime index (America/New_York).
    This will cause issues when merging with timezone-naive data.
    Normalize immediately after fetch.
    """
    stock = yf.Ticker(ticker)
    end = datetime.now()
    start = end - timedelta(days=years * 365)
    
    df = stock.history(start=start, end=end, auto_adjust=True)
    
    # IMMEDIATELY normalize timezone to avoid merge failures
    df.index = df.index.tz_localize(None)
    df.index.name = 'Date'
    
    return df
```

**Save checkpoint:**
```python
df.to_json(f"{ticker}_prices_{years}y.json", orient="split", date_format="iso")
```

### 2b. Stock Info & Fundamentals

```python
def fetch_stock_info(ticker: str) -> dict:
    """
    Fetch current valuation metrics and company info.
    
    Returns dict with keys like: trailingPE, forwardPE, marketCap,
    sector, industry, shortName, etc.
    
    NOTE: Some fields may be None for certain tickers. Always validate.
    """
    stock = yf.Ticker(ticker)
    info = stock.info
    
    # Extract only the fields we need (avoid storing massive dict)
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
        earnings_history: DataFrame with columns [Revenue, Earnings] by quarter
        earnings_dates: DataFrame with columns [Earnings Date, EPS Estimate,
                        Reported EPS, Surprise(%)] — includes future dates
    
    CRITICAL: earnings_dates may include future (unannounced) dates with NaN
    for Reported EPS. Filter these out before analysis.
    """
    stock = yf.Ticker(ticker)
    
    earnings_history = stock.quarterly_earnings
    earnings_dates = stock.earnings_dates
    
    # Reset index for easier manipulation
    if earnings_dates is not None:
        earnings_dates = earnings_dates.reset_index()
        earnings_dates.columns = ['Earnings Date', 'EPS Estimate', 'Reported EPS', 'Surprise(%)']
        # Remove future dates (no reported EPS yet)
        earnings_dates = earnings_dates.dropna(subset=['Reported EPS'])
        # Normalize timezone
        earnings_dates['Earnings Date'] = pd.to_datetime(
            earnings_dates['Earnings Date']
        ).dt.tz_localize(None)
    
    return earnings_history, earnings_dates
```

### 2d. Benchmark / Sector Data

```python
def fetch_benchmark_data(ticker: str = "SPY", years: int = 2) -> pd.DataFrame:
    """
    Fetch benchmark index prices for relative performance comparison.
    Also useful for computing beta and relative strength.
    """
    return fetch_price_history(ticker, years)


def get_sector_pe_from_search() -> dict:
    """
    Web search for current sector P/E ratios.
    
    Search queries to use:
    - "S&P 500 technology sector average P/E ratio {current_year}"
    - "semiconductor industry average P/E ratio"
    - "{ticker} sector valuation metrics"
    
    Returns dict with at minimum:
    {
        'sector_pe': float,       # e.g., 39.1
        'index_pe': float,        # e.g., 22.5 for S&P 500
        'source': str,            # attribution
        'as_of': str              # date of data
    }
    
    NOTE: Web-sourced P/E data is approximate. Use as directional
    benchmark, not precise comparison. Always cite the source and date.
    """
    # Agent should use web search tool to find current data
    # Return structured results
    pass
```

### 2e. Industry & Fundamental Context (Web Search)

```python
INDUSTRY_SEARCH_QUERIES = [
    "{company} AI market size {year}",
    "{sector} infrastructure spending forecast",
    "{company} market share {technology}",
    "{company} revenue growth data center {year}",
    "{company} competitive landscape vs AMD Intel custom chips",
    "AI adoption enterprise statistics {year}"
]
```

**Purpose:** These searches provide qualitative context for the final report's
"Is it hype or real?" assessment. The agent should extract:
- Market size numbers (with source attribution)
- Growth rate statistics
- Competitive positioning data
- Adoption metrics (deployed models, enterprise customers, etc.)

---

## 3. Phase 2: Data Processing & Feature Engineering

### 3a. Timezone Normalization (CRITICAL)

This is the #1 source of silent failures in financial data pipelines.

```python
def normalize_timezone(df: pd.DataFrame) -> pd.DataFrame:
    """
    Remove timezone info from DataFrame index.
    
    yfinance returns tz-aware (America/New_York) indices.
    Manual date comparisons, merges, and .loc lookups FAIL silently
    when mixing tz-aware and tz-naive datetimes.
    
    Apply this IMMEDIATELY after every yfinance fetch.
    """
    if df.index.tz is not None:
        df.index = df.index.tz_localize(None)
    return df
```

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
    
    Args:
        earnings_dates: DataFrame with 'Earnings Date' column
        price_df: OHLCV DataFrame with DatetimeIndex
        window_before: Trading days before earnings to include
        window_after: Trading days after earnings to include
    
    Returns:
        DataFrame with one row per earnings event:
        [
            'earnings_date',
            'actual_trade_date',      # nearest actual trading day
            'close_before',           # close price window_before days prior
            'close_day_of',           # close on earnings day (or next trading)
            'close_day_after',        # close 1 day after
            'close_after_window',     # close window_after days after
            'return_1d',              # 1-day return post-earnings
            'return_window',          # window return (before to after)
            'eps_estimate',
            'eps_actual',
            'eps_surprise',
            'eps_surprise_pct'
        ]
    
    CRITICAL: Earnings are often announced AFTER market close or BEFORE
    market open. The "event day" return should use the NEXT trading day's
    open if announced after close, or the SAME day's close if during hours.
    
    For simplicity (and what most retail analyses do), use:
    - close on the trading day ON or IMMEDIATELY AFTER the earnings date
    - close on the trading day BEFORE as the baseline
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
    result['return_1d'] = (result['close_day_of'] - result['close_before']) / result['close_before']
    result['return_2d'] = (result['close_day_after'] - result['close_before']) / result['close_before']
    result['return_window'] = (result['close_window_after'] - result['close_window_before']) / result['close_window_before']
    
    return result
```

### 3c. Derived Metrics

```python
def compute_derived_metrics(price_df: pd.DataFrame, benchmark_df: pd.DataFrame) -> pd.DataFrame:
    """
    Add analytical columns to the price DataFrame.
    """
    df = price_df.copy()
    
    # Daily returns
    df['daily_return'] = df['Close'].pct_change()
    
    # Cumulative return from start
    df['cumulative_return'] = (1 + df['daily_return']).cumprod() - 1
    
    # Rolling volatility (20-day, annualized)
    df['volatility_20d'] = df['daily_return'].rolling(20).std() * np.sqrt(252)
    
    # Relative to benchmark
    if benchmark_df is not None:
        bench_aligned = benchmark_df['Close'].reindex(df.index, method='ffill')
        df['benchmark_return'] = bench_aligned.pct_change()
        df['excess_return'] = df['daily_return'] - df['benchmark_return']
        df['cumulative_benchmark'] = (1 + df['benchmark_return']).cumprod() - 1
    
    # 50/200 day moving averages (for trend context)
    df['ma_50'] = df['Close'].rolling(50).mean()
    df['ma_200'] = df['Close'].rolling(200).mean()
    
    # Volume anomalies
    df['volume_ma_20'] = df['Volume'].rolling(20).mean()
    df['volume_ratio'] = df['Volume'] / df['volume_ma_20']
    
    return df
```

### 3d. Data Quality Validation

```python
def validate_data(price_df: pd.DataFrame, earnings_df: pd.DataFrame) -> list[str]:
    """
    Run sanity checks. Return list of warnings (empty = all good).
    """
    warnings = []
    
    # Check for gaps > 5 days (holidays are 1-3 days)
    date_diffs = price_df.index.to_series().diff().dt.days
    large_gaps = date_diffs[date_diffs > 5]
    if len(large_gaps) > 0:
        warnings.append(f"Found {len(large_gaps)} gaps > 5 trading days. May indicate data issues.")
    
    # Check for zero-volume days
    zero_vol = (price_df['Volume'] == 0).sum()
    if zero_vol > 0:
        warnings.append(f"Found {zero_vol} zero-volume days. These may be holidays or data errors.")
    
    # Check earnings dates have valid EPS data
    if earnings_df is not None:
        missing_eps = earnings_df['eps_actual'].isna().sum()
        if missing_eps > 0:
            warnings.append(f"{missing_eps} earnings events missing actual EPS.")
    
    # Check for sufficient data length
    if len(price_df) < 252:
        warnings.append("Less than 1 year of trading data. Statistical conclusions may be unreliable.")
    
    return warnings
```

---

## 4. Phase 3: Multi-Dimensional Analysis

### 4a. Event Study — Price Action Around Earnings

This is the core analysis: does the stock move significantly around earnings?

```python
def earnings_event_study(events_df: pd.DataFrame) -> dict:
    """
    Analyze stock returns around earnings announcements.
    
    Returns dict with:
    - mean_1d_return: Average 1-day post-earnings return
    - median_1d_return: Median (more robust to outliers)
    - positive_rate: % of events with positive 1-day return
    - mean_window_return: Average return across the full window
    - vol_surge_ratio: Average earnings-day volume / normal volume
    - correlation_surprise_return: EPS surprise % vs stock return correlation
    - correlation_p_value: Statistical significance of that correlation
    - surprise_return_regression: Slope and R² from linear regression
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
        corr, p_value = stats.pearsonr(
            clean['eps_surprise_pct'], clean['return_1d']
        )
        results['correlation_surprise_return'] = corr
        results['correlation_p_value'] = p_value
        results['correlation_significant'] = p_value < 0.05
        
        # Linear regression for more detail
        slope, intercept, r_value, p_val, std_err = stats.linregress(
            clean['eps_surprise_pct'], clean['return_1d']
        )
        results['regression_slope'] = slope
        results['regression_r_squared'] = r_value ** 2
        results['regression_p_value'] = p_val
    
    return results
```

**Interpretation Guide:**

| Finding                              | Meaning                                                |
|--------------------------------------|--------------------------------------------------------|
| High positive rate (>70%)            | Market consistently rewards earnings                   |
| Low surprise-return correlation      | Stock moves aren't about the numbers — they're about guidance, narrative, sentiment |
| High volatility on earnings days     | Earnings are a significant risk event                  |
| Window return >> 1-day return        | The move extends beyond the announcement               |

### 4b. Earnings-Timing Strategy Backtest

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
    Compare to buy-and-hold over the same total period.
    
    CRITICAL CORRECTION to original workflow's approach:
    The original compared a strategy that was INVESTED for only ~14 days/quarter
    (~56 days/year out of 252) against buy-and-hold which was INVESTED 100% of the time.
    This is an unfair comparison because buy-and-hold captures all the upside.
    
    To make a fair comparison, also compute:
    - Annualized return for the earnings strategy (adjusting for time-in-market)
    - Sharpe ratio for both strategies
    - Max drawdown for both strategies
    
    This tells us whether the earnings-window alpha is worth the complexity.
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
    
    # Sharpe ratio (annualized, assuming risk-free rate ~4.5% for 2024)
    risk_free = 0.045
    if len(trades_df) > 1:
        trades_per_year = len(trades_df) / (total_calendar_days / 365)
        mean_return = trades_df['return'].mean()
        std_return = trades_df['return'].std()
        sharpe = (mean_return * trades_per_year - risk_free) / (std_return * np.sqrt(trades_per_year)) if std_return > 0 else 0
    else:
        sharpe = 0
    
    # Final portfolio values
    strategy_final = initial_capital * (1 + strategy_cumulative)
    bh_final = initial_capital * (1 + bh_cumulative)
    
    return {
        'trades': trades_df,
        'total_trades': len(trades_df),
        'win_rate': (trades_df['return'] > 0).mean(),
        'avg_trade_return': trades_df['return'].mean(),
        'strategy_cumulative_return': strategy_cumulative,
        'buyhold_cumulative_return': bh_cumulative,
        'underperformance': strategy_cumulative - bh_cumulative,
        'time_in_market': time_in_market,
        'sharpe_ratio': sharpe,
        'strategy_final_value': strategy_final,
        'buyhold_final_value': bh_final,
        'total_holding_days': total_holding_days,
        'total_calendar_days': total_calendar_days,
    }
```

**Fair Comparison Note (Augmented Insight):**

The original analysis showed earnings strategy at -2.48% vs. buy-and-hold at +276.4%.
While the directional conclusion is correct (buy-and-hold wins), the magnitude
comparison is misleading because:

1. The earnings strategy was only invested ~22% of the time
2. Annualized returns would be more comparable
3. The real question is: *"Does the earnings window contain excess alpha?"*
   not "Does it beat being invested 100% of the time?"

Always compute **Sharpe ratio** and **time-in-market-adjusted returns** alongside
raw cumulative returns to give a fair picture.

### 4c. Valuation Benchmarking

```python
def valuation_analysis(
    stock_info: dict,
    sector_pe: float,
    index_pe: float
) -> dict:
    """
    Compare current valuation to sector and market benchmarks.
    
    Compute:
    - Trailing P/E premium vs sector (%)
    - Forward P/E premium vs sector (%)
    - PEG ratio if growth rate available
    - Price-to-Sales if available (useful for high-growth tech)
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
    
    # Key insight: Forward P/E can justify a high trailing P/E
    # if earnings growth is expected to be strong
    if trailing_pe and forward_pe and forward_pe < trailing_pe:
        results['earnings_growth_implied'] = (trailing_pe / forward_pe - 1) * 100
        results['valuation_note'] = "Forward P/E below trailing suggests expected earnings growth"
    
    return results
```

### 4d. Risk-Adjusted Performance (Augmented)

```python
def risk_metrics(price_df: pd.DataFrame, benchmark_df: pd.DataFrame = None) -> dict:
    """
    Compute risk-adjusted return metrics.
    
    These were MISSING from the original analysis and are essential
    for a complete investment assessment.
    """
    returns = price_df['Close'].pct_change().dropna()
    
    # Annualized return
    total_return = (price_df['Close'].iloc[-1] / price_df['Close'].iloc[0]) - 1
    years = (price_df.index[-1] - price_df.index[0]).days / 365.25
    annualized_return = (1 + total_return) ** (1 / years) - 1
    
    # Annualized volatility
    annualized_vol = returns.std() * np.sqrt(252)
    
    # Sharpe ratio (assuming 4.5% risk-free rate)
    risk_free = 0.045
    sharpe = (annualized_return - risk_free) / annualized_vol if annualized_vol > 0 else 0
    
    # Sortino ratio (downside deviation only)
    downside_returns = returns[returns < 0]
    downside_std = downside_returns.std() * np.sqrt(252)
    sortino = (annualized_return - risk_free) / downside_std if downside_std > 0 else 0
    
    # Maximum drawdown
    cumulative = (1 + returns).cumprod()
    rolling_max = cumulative.cummax()
    drawdown = (cumulative - rolling_max) / rolling_max
    max_drawdown = drawdown.min()
    
    # Beta (if benchmark provided)
    beta = None
    alpha = None
    if benchmark_df is not None:
        bench_returns = benchmark_df['Close'].pct_change().dropna()
        # Align dates
        common = returns.index.intersection(bench_returns.index)
        if len(common) > 20:
            cov = np.cov(returns.loc[common], bench_returns.loc[common])
            beta = cov[0, 1] / cov[1, 1]
            bench_annual = (benchmark_df['Close'].iloc[-1] / benchmark_df['Close'].iloc[0]) ** (1/years) - 1
            alpha = annualized_return - (risk_free + beta * (bench_annual - risk_free))
    
    return {
        'total_return': total_return,
        'annualized_return': annualized_return,
        'annualized_volatility': annualized_vol,
        'sharpe_ratio': sharpe,
        'sortino_ratio': sortino,
        'max_drawdown': max_drawdown,
        'beta': beta,
        'alpha': alpha,
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
    risk_metrics: dict,
    industry_context: dict  # from web search
) -> dict:
    """
    Combine all analyses into a cohesive assessment.
    
    This is where the agent synthesizes quantitative and qualitative
    findings to answer the core question: "Hype or sustainable?"
    
    The agent should evaluate:
    
    1. EARNINGS QUALITY
       - Are earnings beats consistent? (high positive rate)
       - Is the surprise magnitude meaningful? (not just beating by pennies)
       - Does the market react rationally to surprise magnitude? (correlation)
    
    2. PRICE ACTION INTEGRITY
       - Are gains sustained between earnings? (backtest underperformance)
       - Is volatility concentrated or distributed? (event study window returns)
       - Is the stock momentum-driven or fundamentally-driven?
    
    3. VALUATION REASONABLENESS
       - Is the premium justified by growth? (forward P/E vs trailing)
       - How does it compare to historical norms? (web search for historical P/E)
       - Is the market pricing in realistic growth scenarios?
    
    4. RISK PROFILE
       - What's the max drawdown investors should expect?
       - Is the risk-adjusted return attractive? (Sharpe > 1.0 is good)
       - Is the stock adding alpha vs. the benchmark?
    
    5. INDUSTRY FUNDAMENTALS
       - Is there real demand? (revenue growth, market size)
       - Is the competitive moat sustainable? (market share data)
       - Are there structural tailwinds? (adoption trends, capex cycles)
    """
    pass  # Agent synthesizes based on all inputs
```

---

## 5. Phase 4: Visualization

### Chart Architecture

The original produced an 8-panel figure. Here's the improved layout:

```python
def create_analysis_charts(
    price_df: pd.DataFrame,
    events_df: pd.DataFrame,
    strategy_results: dict,
    valuation: dict,
    risk: dict,
    ticker: str
) -> str:  # returns file path
    """
    Create a comprehensive multi-panel analysis figure.
    
    Layout (2 rows x 4 cols = 8 panels):
    
    Row 1:
    [1. Price + Earnings Events]  [2. Cumulative Returns]
    [3. Earnings-Day Returns]     [4. EPS Surprise vs Return]
    
    Row 2:
    [5. Valuation Comparison]     [6. Strategy Backtest]
    [7. Volume Around Earnings]   [8. Key Metrics Summary]
    
    Design principles:
    - Dark theme (dark charcoal background, not pure black)
    - Accent color for the primary ticker (green for positive, red for negative)
    - Muted colors for benchmarks/secondary data
    - Annotate key data points directly on charts (not just in legend)
    - Use consistent date formatting across all time-series panels
    - Large, readable fonts (title: 14pt, labels: 11pt, ticks: 9pt)
    """
    fig, axes = plt.subplots(2, 4, figsize=(24, 12), facecolor='#1a1a2e')
    fig.suptitle(f'{ticker} Comprehensive Earnings & Valuation Analysis',
                 fontsize=18, fontweight='bold', color='#e0e0e0', y=0.98)
    
    # Configure global plot style
    for ax in axes.flat:
        ax.set_facecolor('#16213e')
        ax.tick_params(colors='#a0a0a0', labelsize=9)
        ax.spines['bottom'].set_color('#333')
        ax.spines['left'].set_color('#333')
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.grid(True, alpha=0.15, color='#555')
    
    # Panel 1: Price chart with earnings markers
    # ... implementation for each panel ...
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    filepath = f'{ticker}_comprehensive_analysis.png'
    fig.savefig(filepath, dpi=150, bbox_inches='tight', facecolor=fig.get_facecolor())
    plt.close()
    
    return filepath
```

### Individual Panel Implementations

```python
# Panel 1: Price + Moving Averages + Earnings Markers
ax1 = axes[0, 0]
ax1.plot(price_df.index, price_df['Close'], color='#00d4aa', linewidth=1.5, label='Close')
ax1.plot(price_df.index, price_df['ma_50'], color='#ffa500', linewidth=0.8, alpha=0.7, label='50-MA')
ax1.plot(price_df.index, price_df['ma_200'], color='#ff6b6b', linewidth=0.8, alpha=0.7, label='200-MA')

# Mark earnings dates
for _, event in events_df.iterrows():
    color = '#00ff88' if event['return_1d'] > 0 else '#ff4444'
    ax1.axvline(event['earnings_date'], color=color, alpha=0.3, linewidth=0.5)

ax1.set_title('Price & Earnings Events', color='#e0e0e0', fontsize=12, fontweight='bold')
ax1.legend(fontsize=8, loc='upper left', facecolor='#1a1a2e', edgecolor='#333', labelcolor='#a0a0a0')
ax1.yaxis.set_major_formatter(plt.FuncFormatter(lambda x, p: f'${x:,.0f}'))


# Panel 5: Valuation Comparison Bar Chart
ax5 = axes[1, 0]
categories = ['Trailing P/E', 'Forward P/E']
stock_vals = [valuation['trailing_pe'], valuation['forward_pe']]
sector_vals = [valuation['sector_pe'], valuation['sector_pe']]  # same for trailing/forward
index_vals = [valuation['index_pe'], valuation['index_pe']]

x = np.arange(len(categories))
width = 0.25
ax5.bar(x - width, stock_vals, width, label=ticker, color='#00d4aa', alpha=0.9)
ax5.bar(x, sector_vals, width, label='Tech Sector', color='#ffa500', alpha=0.7)
ax5.bar(x + width, index_vals, width, label='S&P 500', color='#666', alpha=0.7)

ax5.set_title('Valuation Comparison', color='#e0e0e0', fontsize=12, fontweight='bold')
ax5.set_xticks(x)
ax5.set_xticklabels(categories, color='#a0a0a0')


# Panel 8: Key Metrics Summary (text-based, no axes)
ax8 = axes[1, 3]
ax8.axis('off')

metrics_text = f"""
KEY METRICS SUMMARY
━━━━━━━━━━━━━━━━━━
Total Return:        {risk['total_return']:+.1%}
Annualized Return:   {risk['annualized_return']:+.1%}
Sharpe Ratio:        {risk['sharpe_ratio']:.2f}
Max Drawdown:        {risk['max_drawdown']:.1%}
Beta:                {risk.get('beta', 'N/A')}
Volatility:          {risk['annualized_volatility']:.1%}

EARNINGS ANALYSIS
━━━━━━━━━━━━━━━━━━
Events Analyzed:     {len(events_df)}
Positive 1d Rate:    {events_df['return_1d'].gt(0).mean():.0%}
Avg 1d Return:       {events_df['return_1d'].mean():+.2%}
EPS-Ret Correlation: {events_df['eps_surprise_pct'].corr(events_df['return_1d']):.3f}
"""
ax8.text(0.05, 0.95, metrics_text, transform=ax8.transAxes,
         fontsize=10, fontfamily='monospace', color='#e0e0e0',
         verticalalignment='top', bbox=dict(boxstyle='round', facecolor='#0d1b2a', alpha=0.8))
```

---

## 6. Phase 5: Report Generation

### HTML Report Structure

The report should be a self-contained HTML file with embedded CSS. Structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{TICKER} Earnings & Valuation Analysis</title>
    <style>
        /* Design: Editorial / data-journalism aesthetic */
        /* Typography: Serif headings (DM Serif Display) + monospace data */
        /* Color: Dark charcoal (#0d0d0d) bg, warm sand (#f0ece4) text, green accent */
        /* Layout: Single-column editorial flow, NOT card grid */
    </style>
</head>
<body>
    <!-- 1. Header: Ticker, date, one-line thesis -->
    <!-- 2. Executive Summary: Key numbers in a highlight strip -->
    <!-- 3. Analysis Chart: The comprehensive 8-panel figure -->
    <!-- 4. Narrative Sections: -->
    <!--    4a. Price Performance Around Earnings -->
    <!--    4b. Trading Strategy Simulation -->
    <!--    4c. Valuation Analysis -->
    <!--    4d. AI/Fundamental Market Context -->
    <!--    4e. Risks and Considerations -->
    <!-- 5. Investment Recommendations: By investor profile -->
    <!-- 6. Methodology: Data sources, period, statistical methods -->
</body>
</html>
```

### Key Content Sections

Each section follows this pattern:

```
[Section Title]
[Key Insight — 1-2 sentence takeaway]
[Supporting Data Table or Chart]
[Detailed Narrative — 2-4 paragraphs]
```

### Recommendation Framework

Structure recommendations by investor profile:

| Profile             | Time Horizon | Recommendation | Rationale                            |
|---------------------|-------------|----------------|--------------------------------------|
| Long-term investor  | 3+ years    | BUY / HOLD     | AI infrastructure early innings      |
| Growth investor     | 1-3 years   | ACCUMULATE     | Strong fundamentals, premium val.    |
| Short-term trader   | < 1 year    | AVOID TIMING   | Earnings strategy underperforms      |
| Income investor     | Any         | NOT SUITABLE   | Low/no dividend, high volatility     |

---

## 7. Phase 6: Validation & QA

### Pre-Delivery Checks

```python
def run_qa_checks(
    price_df: pd.DataFrame,
    events_df: pd.DataFrame,
    strategy_results: dict,
    valuation: dict,
    report_html: str,
    chart_path: str
) -> list[str]:
    """
    Final quality assurance. Return list of issues found.
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
    if not os.path.exists(chart_path):
        issues.append(f"ERROR: Chart file not found at {chart_path}")
    
    if '<html' not in report_html.lower():
        issues.append("ERROR: Report HTML appears malformed.")
    
    return issues
```

### Statistical Validity Checks

```python
def check_statistical_validity(events_df: pd.DataFrame) -> dict:
    """
    Verify that statistical claims in the report are supportable.
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

### Timezone Traps (The Silent Killer)

**Problem:** `yfinance` returns timezone-aware indices (`America/New_York`).
When you merge or do `.loc[date]` with a tz-naive timestamp, pandas does NOT
raise an error — it returns empty results silently.

**Fix:** Normalize to tz-naive IMMEDIATELY after every `yfinance` fetch.
See Section 3a.

### Weekend/Holiday Earnings

**Problem:** Some companies report earnings on weekends or market holidays.
The "next trading day" logic must handle this.

**Fix:** Always snap earnings dates to the nearest actual trading day using
the price DataFrame's index.

### Survivorship Bias in Strategy Backtests

**Problem:** We only analyze companies that still exist and are still
heavily traded. Companies that crashed and were delisted are invisible.

**Mitigation:** Acknowledge this in the methodology section. The analysis
is specifically about the queried ticker, not a general strategy.

### Look-Ahead Bias

**Problem:** Using future data in a "historical" simulation.

**Common mistake:** Using today's P/E to compare against historical prices.

**Fix:** Use point-in-time data. For P/E comparisons, use the P/E that
was available at each point in time, not the current one applied retroactively.

### The "Beating by Pennies" Problem

**Insight:** Most companies consistently beat EPS estimates by small margins
(managed expectations). A +14% surprise sounds impressive, but check:
- How does it compare to the sector average surprise?
- Is the beat growing or shrinking over time?
- Are revenue surprises also positive? (Revenue is harder to manage than EPS)

### Short Interest & Options Activity

**Augmented dimension:** For a complete picture, consider:
- Short interest ratio before earnings (squeeze potential)
- Implied volatility vs. realized volatility post-earnings
- Options market expected move vs. actual move

These explain why stocks sometimes move opposite to earnings surprise
(the surprise was already priced in via options market expectations).

### Macro Regime Awareness

**Augmented dimension:** The analysis period matters enormously.
- 2023-2025 was an extraordinary AI bull market
- The same analysis of NVDA in 2018-2020 would show a very different picture
- Always note the macro context: rate environment, risk appetite, sector rotation

### The "Forward P/E is Lower" Trap

**Insight:** When forward P/E < trailing P/E, the report says "growth justifies
the premium." But forward estimates are often optimistic. Consider:
- How accurate have forward estimates been historically?
- What happens to the stock if growth merely meets (not beats) estimates?
- Compute the implied growth rate and compare to historical growth.

---

## 9. Reusable Code Templates

### Template: Full Pipeline Orchestration

```python
def run_full_analysis(
    ticker: str,
    years: int = 2,
    sector_pe: float = 39.1,
    index_pe: float = 22.5,
    output_dir: str = "output"
) -> dict:
    """
    Complete pipeline from data acquisition to report generation.
    
    Usage:
        results = run_full_analysis("NVDA", years=2, sector_pe=39.1, index_pe=22.5)
    """
    os.makedirs(output_dir, exist_ok=True)
    
    # Phase 1: Data Acquisition
    print(f"[1/6] Fetching data for {ticker}...")
    price_df = fetch_price_history(ticker, years)
    stock_info = fetch_stock_info(ticker)
    earnings_hist, earnings_dates = fetch_earnings_data(ticker)
    benchmark_df = fetch_benchmark_data("SPY", years)
    
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
    valuation = valuation_analysis(stock_info, sector_pe, index_pe)
    risk = risk_metrics(price_df, benchmark_df)
    
    # Phase 4: Visualization
    print("[4/6] Creating charts...")
    chart_path = os.path.join(output_dir, f"{ticker}_analysis.png")
    create_analysis_charts(price_df, events_df, strategy, valuation, risk, ticker)
    
    # Phase 5: Report
    print("[5/6] Generating report...")
    # report_html = generate_html_report(...)  # implement with Jinja2
    
    # Phase 6: Validation
    print("[6/6] Running QA checks...")
    stat_validity = check_statistical_validity(events_df)
    
    return {
        'price_df': price_df,
        'events_df': events_df,
        'event_study': event_study,
        'strategy': strategy,
        'valuation': valuation,
        'risk': risk,
        'statistical_validity': stat_validity,
        'chart_path': chart_path,
    }
```

### Template: Quick Single-Ticker Check

```python
def quick_valuation_check(ticker: str) -> None:
    """
    Minimal check: current P/E vs sector. Useful for screening.
    """
    info = fetch_stock_info(ticker)
    pe = info.get('trailingPE')
    fwd_pe = info.get('forwardPE')
    print(f"{ticker}: Trailing P/E = {pe:.1f}x, Forward P/E = {fwd_pe:.1f}x")
    # Compare against hardcoded or fetched sector averages
```

---

## 10. Quality Checklist

Before delivering any analysis, verify:

- [ ] **Data Integrity**: No timezone conflicts, no duplicate dates, no NaN in critical columns
- [ ] **Sufficient Sample**: At least 8 earnings events for quarterly analysis
- [ ] **Statistical Claims Backed**: Every correlation or "X% of the time" has the sample size noted
- [ ] **Fair Comparisons**: Strategy backtests account for time-in-market; P/E comparisons note the source and date
- [ ] **Risk Metrics Present**: Sharpe ratio, max drawdown, and volatility are reported alongside returns
- [ ] **Visualizations Complete**: All 8 panels rendered, readable, and annotated
- [ ] **Recommendations Qualified**: Different advice for different investor profiles
- [ ] **Methodology Documented**: Data sources, time period, statistical methods, and limitations stated
- [ ] **Edge Cases Acknowledged**: Timezone issues, survivorship bias, macro context noted
- [ ] **Report Self-Contained**: HTML report has no broken images or missing data
- [ ] **Citations Present**: Web-sourced data (P/E ratios, market size) has source attribution
- [ ] **Warnings Surfaced**: Any data quality issues from validation are prominently noted

---

## Appendix: Common Pitfalls Summary

| Pitfall                        | Symptom                          | Fix                                      |
|--------------------------------|----------------------------------|------------------------------------------|
| Timezone mismatch              | Empty merge results              | `tz_localize(None)` immediately          |
| Future earnings dates included | NaN in analysis, inflated counts | `dropna(subset=['Reported EPS'])`         |
| Unfair strategy comparison     | Misleading underperformance %    | Compute Sharpe, annualize, note time-in-mkt |
| Missing risk metrics           | Incomplete investment thesis     | Always compute Sharpe, Sortino, max DD    |
| No significance testing        | False confidence in correlations | Report p-value alongside every correlation|
| Single P/E data source         | Stale or inaccurate benchmark    | Cross-reference multiple sources          |
| Ignoring macro context         | Analysis valid only in one regime| Note rate environment, sector rotation    |
| Revenue ignored                | Missed earnings quality signal   | Check revenue surprise alongside EPS      |

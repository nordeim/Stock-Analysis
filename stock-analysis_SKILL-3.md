---
name: stock-analysis-and-valuation
description: >
  Earnings-Event Stock Analysis & AI Sector Valuation
  Guide an AI agent through a rigorous, multi-phase pipeline that analyzes a stock's price behavior around earnings events,
  benchmarks its valuation against sector/indices, backtests an earnings-timing strategy, and produces a comprehensive report
  answering: "Is this stock's performance driven by sustainable fundamentals or short-term hype?"
version: 1.0
---

# Earnings-Event Stock Analysis & AI Sector Valuation

> **Purpose:** Guide an AI agent through a rigorous, multi-phase pipeline that analyzes
> a stock's price behavior around earnings events, benchmarks its valuation against
> sector/indices, backtests an earnings-timing strategy, and produces a comprehensive
> report answering: *"Is this stock's performance driven by sustainable fundamentals
> or short-term hype?"*

> **Applicable Tickers:** Any publicly traded equity (demonstrated with NVDA).
> **Time Horizon:** 2+ years of historical data recommended.
> **Skill Level:** Intermediate - requires comfort with pandas, matplotlib, financial APIs.

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
| Industry market size      | IDC, Gartner, McKinsey, Deloitte, WSTS              |
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
    stock = yf.Ticker(ticker)
    earnings_history = stock.quarterly_earnings
    earnings_dates = stock.earnings_dates
    if earnings_dates is not None:
        earnings_dates = earnings_dates.reset_index()
        earnings_dates.columns = ['Earnings Date', 'EPS Estimate', 'Reported EPS', 'Surprise(%)']
        earnings_dates = earnings_dates.dropna(subset=['Reported EPS'])
        earnings_dates['Earnings Date'] = pd.to_datetime(
            earnings_dates['Earnings Date']
        ).dt.tz_localize(None)
    return earnings_history, earnings_dates
```

### 2d. Benchmark / Sector Data

```python
def fetch_benchmark_data(ticker: str = "SPY", years: int = 2) -> pd.DataFrame:
    return fetch_price_history(ticker, years)


def get_sector_pe_from_search() -> dict:
    # Use web search tool to find current data
    # Search queries:
    # - "S&P 500 technology sector average P/E ratio {current_year}"
    # - "semiconductor industry average P/E ratio"
    # - "{ticker} sector valuation metrics"
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
    events = []
    trading_dates = price_df.index.sort_values()
    
    for _, row in earnings_dates.iterrows():
        earn_date = pd.Timestamp(row['Earnings Date'])
        future_dates = trading_dates[trading_dates >= earn_date]
        if len(future_dates) == 0:
            continue
        actual_date = future_dates[0]
        
        past_dates = trading_dates[trading_dates < earn_date]
        if len(past_dates) < window_before:
            continue
        date_before = past_dates[-1]
        date_window_before = past_dates[-window_before]
        
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
    df = price_df.copy()
    df['daily_return'] = df['Close'].pct_change()
    df['cumulative_return'] = (1 + df['daily_return']).cumprod() - 1
    df['volatility_20d'] = df['daily_return'].rolling(20).std() * np.sqrt(252)
    
    if benchmark_df is not None:
        bench_aligned = benchmark_df['Close'].reindex(df.index, method='ffill')
        df['benchmark_return'] = bench_aligned.pct_change()
        df['excess_return'] = df['daily_return'] - df['benchmark_return']
        df['cumulative_benchmark'] = (1 + df['benchmark_return']).cumprod() - 1
    
    df['ma_50'] = df['Close'].rolling(50).mean()
    df['ma_200'] = df['Close'].rolling(200).mean()
    df['volume_ma_20'] = df['Volume'].rolling(20).mean()
    df['volume_ratio'] = df['Volume'] / df['volume_ma_20']
    
    return df
```

### 3d. Data Quality Validation

```python
def validate_data(price_df: pd.DataFrame, earnings_df: pd.DataFrame) -> list[str]:
    warnings = []
    date_diffs = price_df.index.to_series().diff().dt.days
    large_gaps = date_diffs[date_diffs > 5]
    if len(large_gaps) > 0:
        warnings.append(f"Found {len(large_gaps)} gaps > 5 trading days.")
    
    zero_vol = (price_df['Volume'] == 0).sum()
    if zero_vol > 0:
        warnings.append(f"Found {zero_vol} zero-volume days.")
    
    if earnings_df is not None:
        missing_eps = earnings_df['eps_actual'].isna().sum()
        if missing_eps > 0:
            warnings.append(f"{missing_eps} earnings events missing actual EPS.")
    
    if len(price_df) < 252:
        warnings.append("Less than 1 year of trading data.")
    
    return warnings
```

---

## 4. Phase 3: Multi-Dimensional Analysis

### 4a. Event Study - Price Action Around Earnings

```python
def earnings_event_study(events_df: pd.DataFrame) -> dict:
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
| Low surprise-return correlation      | Stock moves aren't about the numbers - they're about guidance, narrative, sentiment |
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
    trading_dates = price_df.index.sort_values()
    
    trades = []
    for _, row in earnings_dates.iterrows():
        earn_date = pd.Timestamp(row['Earnings Date'])
        past_dates = trading_dates[trading_dates < earn_date]
        if len(past_dates) < days_before:
            continue
        buy_date = past_dates[-days_before]
        
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
    strategy_cumulative = (1 + trades_df['return']).prod() - 1
    
    first_trade = trades_df['buy_date'].min()
    last_trade = trades_df['sell_date'].max()
    bh_start = price_df.loc[first_trade, 'Close']
    bh_end = price_df.loc[last_trade, 'Close']
    bh_cumulative = (bh_end - bh_start) / bh_start
    
    total_holding_days = trades_df['holding_days'].sum()
    total_calendar_days = (last_trade - first_trade).days
    time_in_market = total_holding_days / total_calendar_days if total_calendar_days > 0 else 0
    
    years_held = total_calendar_days / 365.25
    strategy_annualized = (1 + strategy_cumulative) ** (1 / years_held) - 1 if years_held > 0 else 0
    bh_annualized = (1 + bh_cumulative) ** (1 / years_held) - 1 if years_held > 0 else 0
    
    risk_free = 0.045
    if len(trades_df) > 1:
        trades_per_year = len(trades_df) / (total_calendar_days / 365)
        mean_return = trades_df['return'].mean()
        std_return = trades_df['return'].std()
        sharpe = (mean_return * trades_per_year - risk_free) / (std_return * np.sqrt(trades_per_year)) if std_return > 0 else 0
    else:
        sharpe = 0
    
    portfolio_values = [initial_capital]
    for ret in trades_df['return']:
        portfolio_values.append(portfolio_values[-1] * (1 + ret))
    pv_series = pd.Series(portfolio_values)
    rolling_max = pv_series.cummax()
    drawdown = (pv_series - rolling_max) / rolling_max
    max_drawdown = drawdown.min()
    
    calmar = strategy_annualized / abs(max_drawdown) if max_drawdown != 0 else float('inf')
    
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
        'max_drawdown': max_drawdown,
        'calmar_ratio': calmar,
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

Always compute **Sharpe ratio**, **Calmar ratio**, and **time-in-market-adjusted returns** alongside
raw cumulative returns to give a fair picture.

### 4c. Valuation Benchmarking

```python
def valuation_analysis(
    stock_info: dict,
    sector_pe: float,
    index_pe: float
) -> dict:
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
    
    if trailing_pe and forward_pe and forward_pe < trailing_pe:
        results['earnings_growth_implied'] = (trailing_pe / forward_pe - 1) * 100
        results['valuation_note'] = "Forward P/E below trailing suggests expected earnings growth"
    
    return results
```

### 4d. Risk-Adjusted Performance (Augmented)

```python
def risk_metrics(price_df: pd.DataFrame, benchmark_df: pd.DataFrame = None) -> dict:
    returns = price_df['Close'].pct_change().dropna()
    
    total_return = (price_df['Close'].iloc[-1] / price_df['Close'].iloc[0]) - 1
    years = (price_df.index[-1] - price_df.index[0]).days / 365.25
    annualized_return = (1 + total_return) ** (1 / years) - 1
    
    annualized_vol = returns.std() * np.sqrt(252)
    
    risk_free = 0.045
    sharpe = (annualized_return - risk_free) / annualized_vol if annualized_vol > 0 else 0
    
    downside_returns = returns[returns < 0]
    downside_std = downside_returns.std() * np.sqrt(252)
    sortino = (annualized_return - risk_free) / downside_std if downside_std > 0 else 0
    
    cumulative = (1 + returns).cumprod()
    rolling_max = cumulative.cummax()
    drawdown = (cumulative - rolling_max) / rolling_max
    max_drawdown = drawdown.min()
    
    calmar = annualized_return / abs(max_drawdown) if max_drawdown != 0 else float('inf')
    
    beta = None
    alpha = None
    information_ratio = None
    if benchmark_df is not None:
        bench_returns = benchmark_df['Close'].pct_change().dropna()
        common = returns.index.intersection(bench_returns.index)
        if len(common) > 20:
            cov = np.cov(returns.loc[common], bench_returns.loc[common])
            beta = cov[0, 1] / cov[1, 1]
            bench_annual = (benchmark_df['Close'].iloc[-1] / benchmark_df['Close'].iloc[0]) ** (1/years) - 1
            alpha = annualized_return - (risk_free + beta * (bench_annual - risk_free))
            
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
    # Agent synthesizes based on all inputs
    # Evaluate: EARNINGS QUALITY, PRICE ACTION INTEGRITY, VALUATION REASONABLENESS,
    #           RISK PROFILE, INDUSTRY FUNDAMENTALS
    pass
```

---

## 5. Phase 4: Visualization

### Chart Architecture

```python
def create_analysis_charts(
    price_df: pd.DataFrame,
    events_df: pd.DataFrame,
    strategy_results: dict,
    valuation: dict,
    risk: dict,
    ticker: str
) -> str:
    fig, axes = plt.subplots(2, 4, figsize=(24, 12), facecolor='#1a1a2e')
    fig.suptitle(f'{ticker} Comprehensive Earnings & Valuation Analysis',
                 fontsize=18, fontweight='bold', color='#e0e0e0', y=0.98)
    
    for ax in axes.flat:
        ax.set_facecolor('#16213e')
        ax.tick_params(colors='#a0a0a0', labelsize=9)
        ax.spines['bottom'].set_color('#333')
        ax.spines['left'].set_color('#333')
        ax.spines['top'].set_visible(False)
        ax.spines['right'].set_visible(False)
        ax.grid(True, alpha=0.15, color='#555')
    
    # Panel 1: Price + Moving Averages + Earnings Markers
    ax1 = axes[0, 0]
    ax1.plot(price_df.index, price_df['Close'], color='#00d4aa', linewidth=1.5, label='Close')
    ax1.plot(price_df.index, price_df['ma_50'], color='#ffa500', linewidth=0.8, alpha=0.7, label='50-MA')
    ax1.plot(price_df.index, price_df['ma_200'], color='#ff6b6b', linewidth=0.8, alpha=0.7, label='200-MA')
    for _, event in events_df.iterrows():
        color = '#00ff88' if event['return_1d'] > 0 else '#ff4444'
        ax1.axvline(event['earnings_date'], color=color, alpha=0.3, linewidth=0.5)
    ax1.set_title('Price & Earnings Events', color='#e0e0e0', fontsize=12, fontweight='bold')
    ax1.legend(fontsize=8, loc='upper left', facecolor='#1a1a2e', edgecolor='#333', labelcolor='#a0a0a0')
    ax1.yaxis.set_major_formatter(plt.FuncFormatter(lambda x, p: f'${x:,.0f}'))
    
    # Panel 5: Valuation Comparison
    ax5 = axes[1, 0]
    categories = ['Trailing P/E', 'Forward P/E']
    stock_vals = [valuation['trailing_pe'], valuation['forward_pe']]
    sector_vals = [valuation['sector_pe'], valuation['sector_pe']]
    index_vals = [valuation['index_pe'], valuation['index_pe']]
    x = np.arange(len(categories))
    width = 0.25
    ax5.bar(x - width, stock_vals, width, label=ticker, color='#00d4aa', alpha=0.9)
    ax5.bar(x, sector_vals, width, label='Tech Sector', color='#ffa500', alpha=0.7)
    ax5.bar(x + width, index_vals, width, label='S&P 500', color='#666', alpha=0.7)
    ax5.set_title('Valuation Comparison', color='#e0e0e0', fontsize=12, fontweight='bold')
    ax5.set_xticks(x)
    ax5.set_xticklabels(categories, color='#a0a0a0')
    
    # Panel 8: Key Metrics Summary
    ax8 = axes[1, 3]
    ax8.axis('off')
    metrics_text = f"""
KEY METRICS SUMMARY
===================
Total Return:        {risk['total_return']:+.1%}
Annualized Return:   {risk['annualized_return']:+.1%}
Sharpe Ratio:        {risk['sharpe_ratio']:.2f}
Sortino Ratio:       {risk['sortino_ratio']:.2f}
Calmar Ratio:        {risk['calmar_ratio']:.2f}
Max Drawdown:        {risk['max_drawdown']:.1%}
Beta:                {risk.get('beta', 'N/A')}
Volatility:          {risk['annualized_volatility']:.1%}

EARNINGS ANALYSIS
===================
Events Analyzed:     {len(events_df)}
Positive 1d Rate:    {events_df['return_1d'].gt(0).mean():.0%}
Avg 1d Return:       {events_df['return_1d'].mean():+.2%}
EPS-Ret Correlation: {events_df['eps_surprise_pct'].corr(events_df['return_1d']):.3f}
"""
    ax8.text(0.05, 0.95, metrics_text, transform=ax8.transAxes,
             fontsize=10, fontfamily='monospace', color='#e0e0e0',
             verticalalignment='top', bbox=dict(boxstyle='round', facecolor='#0d1b2a', alpha=0.8))
    
    plt.tight_layout(rect=[0, 0, 1, 0.95])
    filepath = f'{ticker}_comprehensive_analysis.png'
    fig.savefig(filepath, dpi=150, bbox_inches='tight', facecolor=fig.get_facecolor())
    plt.close()
    
    return filepath
```

---

## 6. Phase 5: Report Generation

### HTML Report Structure

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
    <!-- 4. Narrative Sections -->
    <!-- 5. Investment Recommendations: By investor profile -->
    <!-- 6. Methodology: Data sources, period, statistical methods -->
</body>
</html>
```

### Recommendation Framework

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
    issues = []
    if len(price_df) < 252:
        issues.append("WARNING: Less than 1 year of price data.")
    if len(events_df) < 4:
        issues.append("WARNING: Fewer than 4 earnings events.")
    if strategy_results['strategy_cumulative_return'] > 10:
        issues.append("ERROR: Strategy return > 1000%. Likely a calculation bug.")
    if abs(strategy_results['buyhold_cumulative_return']) > 20:
        issues.append("WARNING: Buy-and-hold return > 2000%.")
    if valuation.get('trailing_pe') and valuation['trailing_pe'] > 500:
        issues.append("WARNING: P/E > 500x.")
    if not os.path.exists(chart_path):
        issues.append(f"ERROR: Chart file not found at {chart_path}")
    if '<html' not in report_html.lower():
        issues.append("ERROR: Report HTML appears malformed.")
    return issues
```

### Statistical Validity Checks

```python
def check_statistical_validity(events_df: pd.DataFrame) -> dict:
    n = len(events_df)
    return {
        'sample_size': n,
        'sample_adequate': n >= 8,
        'correlation_reliable': n >= 10,
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
raise an error - it returns empty results silently.

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

### Risk-Adjusted Metrics: The Full Picture

**Augmented dimension:** The original analysis only reported raw returns.
A complete assessment requires:

| Metric | Formula | What It Measures | Good Value |
|--------|---------|------------------|------------|
| **Sharpe Ratio** | (Rp - Rf) / sigma_p | Return per unit of total volatility | > 1.0 |
| **Sortino Ratio** | (Rp - Rf) / sigma_d | Return per unit of downside volatility | > 1.0 |
| **Calmar Ratio** | CAGR / Max DD | Return relative to worst drawdown | > 1.0 |
| **Information Ratio** | (Rp - Rb) / TE | Active return per unit of tracking error | > 0.5 |
| **Max Drawdown** | Peak-to-trough decline | Worst cumulative loss | Lower is better |

These metrics tell a more complete story than raw returns alone. A stock
with +276% return but -50% max drawdown is very different from one with
+200% return and -15% max drawdown.

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
            print(f"  WARNING: {w}")
    
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
    info = fetch_stock_info(ticker)
    pe = info.get('trailingPE')
    fwd_pe = info.get('forwardPE')
    print(f"{ticker}: Trailing P/E = {pe:.1f}x, Forward P/E = {fwd_pe:.1f}x")
```

---

## 10. Quality Checklist

Before delivering any analysis, verify:

- [ ] **Data Integrity**: No timezone conflicts, no duplicate dates, no NaN in critical columns
- [ ] **Sufficient Sample**: At least 8 earnings events for quarterly analysis
- [ ] **Statistical Claims Backed**: Every correlation or "X% of the time" has the sample size noted
- [ ] **Fair Comparisons**: Strategy backtests account for time-in-market; P/E comparisons note the source and date
- [ ] **Risk Metrics Present**: Sharpe ratio, Sortino ratio, Calmar ratio, max drawdown, and volatility are reported alongside returns
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
| Unfair strategy comparison     | Misleading underperformance %    | Compute Sharpe, Sortino, Calmar, annualize, note time-in-mkt |
| Missing risk metrics           | Incomplete investment thesis     | Always compute Sharpe, Sortino, Calmar, max DD |
| No significance testing        | False confidence in correlations | Report p-value alongside every correlation|
| Single P/E data source         | Stale or inaccurate benchmark    | Cross-reference multiple sources          |
| Ignoring macro context         | Analysis valid only in one regime| Note rate environment, sector rotation    |
| Revenue ignored                | Missed earnings quality signal   | Check revenue surprise alongside EPS      |

---

## DELIVER - Summary & Augmented Insights

The SKILL.md above distills the original NVIDIA analysis workflow into a **repeatable, agent-guided skill** with these key augmentations over the original:

**What was preserved from the original workflow:**
- Multi-phase data acquisition via `yfinance` + web search
- Earnings event alignment methodology
- Strategy backtest concept (buy before / sell after earnings)
- 8-panel visualization architecture
- Recommendation framework by investor profile

**What was augmented (new in SKILL.md):**

| Augmentation | Why It Matters |
|---|---|
| **Risk-adjusted returns** (Sharpe, Sortino, Calmar, max drawdown) | Raw returns don't tell you if the risk was worth it |
| **Fair backtest comparison** (time-in-market adjustment) | Original compared 22% invested time vs. 100% - misleading |
| **Statistical significance testing** (p-values, confidence intervals) | A correlation of 0.031 means nothing without a p-value |
| **Timezone normalization as a first-class concern** | #1 silent failure in financial data pipelines |
| **Data quality validation pipeline** | Prevents garbage-in-garbage-out analysis |
| **Look-ahead bias and survivorship bias warnings** | Essential intellectual honesty for backtests |
| **Forward P/E trap analysis** | Forward estimates are often wrong - don't just trust them |
| **Short interest and options awareness** | Explains "irrational" post-earnings moves |
| **Macro regime context** | Same analysis in different market conditions = different conclusion |
| **Revenue surprise alongside EPS** | EPS can be managed; revenue is the real signal |
| **Information Ratio** | Measures active return per unit of tracking error |
| **Calmar Ratio** | Annualized return relative to maximum drawdown |

The SKILL.md is designed to be **framework-agnostic at the instruction level** - an agent using any tool (Python, R, JavaScript) can follow the methodology. The code templates are Python-specific but the analytical logic transfers.

---
name: infographic-stock-analysis
description: >
  End-to-end pipeline for producing an impressive, self-contained interactive HTML infographic
  that analyzes stock price behavior around earnings announcements, benchmarks valuation
  against sector/indices with dynamic data, backtests earnings-timing strategies, and
  delivers an editorial-style "Hype vs. Sustainable" assessment.
  Combines quantitative finance, data journalism, web-search augmentation, and
  avant-garde UI/UX design into a single reusable workflow.
version: 1.0
domain: Quantitative Finance / Data Journalism / Avant-Garde UI/UX
---

# SKILL.md — Interactive Earnings-Event Stock Analysis Infographic

**Domain:** Quantitative Finance / Data Journalism / Avant-Garde UI/UX

---

## Table of Contents

1. [Overview & Philosophy](#1-overview--philosophy)
2. [Source Skills & Dependencies](#2-source-skills--dependencies)
3. [Environment Setup & Pitfalls](#3-environment-setup--pitfalls)
4. [Phase 1: Data Acquisition](#4-phase-1-data-acquisition)
5. [Phase 2: Data Processing & Feature Engineering](#5-phase-2-data-processing--feature-engineering)
6. [Phase 3: Multi-Dimensional Analysis](#6-phase-3-multi-dimensional-analysis)
7. [Phase 4: Web Research Augmentation](#7-phase-4-web-research-augmentation)
8. [Phase 5: Avant-Garde HTML Infographic Generation](#8-phase-5-avant-garde-html-infographic-generation)
9. [Phase 6: QA Validation & Delivery](#9-phase-6-qa-validation--delivery)
10. [Lessons Learned & Critical Pitfalls](#10-lessons-learned--critical-pitfalls)
11. [Design System Reference](#11-design-system-reference)
12. [Chart Architecture Reference](#12-chart-architecture-reference)
13. [Reusable Code Templates](#13-reusable-code-templates)
14. [Quality Checklist](#14-quality-checklist)

---

## 1. Overview & Philosophy

### Purpose

Guide an AI agent through a rigorous, multi-phase pipeline that:

1. Analyzes stock price behavior around earnings announcements using real market data
2. Benchmarks valuation against sector/indices with **dynamically sourced** (not hardcoded) data
3. Backtests earnings-timing strategies with **fair time-in-market adjustments**
4. Augments quantitative analysis with **web-researched industry context**
5. Produces an **interactive, self-contained HTML infographic** answering: *"Is this stock's performance driven by sustainable fundamentals or short-term hype?"*

### Anti-Generic Mandate

- **REJECT** static Matplotlib PNGs for web reports
- **REJECT** Bootstrap-style card grids and predictable layouts
- **REJECT** Inter/Roboto system fonts without intentional typographic hierarchy
- **REJECT** hardcoded sector P/E ratios that become stale
- **REJECT** unfair backtest comparisons (no time-in-market adjustment)
- **EMBRACE** interactive, bespoke visualizations with micro-interactions
- **EMBRACE** editorial/data-journalism aesthetic with intentional minimalism
- **EMBRACE** dark charcoal backgrounds, warm sand text, terminal green accents
- **EMBRACE** dynamic data sourcing via web search for benchmarks
- **EMBRACE** statistical rigor (p-values, confidence intervals, sample-size caveats)

### Technical Rigor

- **Timezone normalization** is mandatory (not optional) — the #1 source of silent failures
- **Fair backtesting** requires time-in-market adjustments
- **Statistical claims** require p-values and confidence intervals
- **Risk metrics** (Sharpe, Sortino, Calmar, max drawdown) are non-negotiable
- **Dynamic data sourcing** prevents stale benchmarks
- **Python venv awareness** — system Python and venv Python are different interpreters

### Applicable Tickers

Any publicly traded equity (demonstrated with NVDA, applicable to AAPL, MSFT, GOOGL, AMZN, etc.)

### Time Horizon

2+ years recommended (minimum 8 earnings events for statistical validity)

---

## 2. Source Skills & Dependencies

This skill synthesizes knowledge from multiple source skills. An agent executing this workflow MUST load and understand these skills:

### Primary Source Skill

| Skill | File | Purpose |
|-------|------|---------|
| **stock-analysis** | `stock-analysis_SKILL.md` | Core pipeline: data acquisition, analysis, visualization templates, QA checks. This is the foundation. |

### Augmentation Skills

| Skill | When to Load | Purpose |
|-------|-------------|---------|
| **charts** | Before Phase 5 (Visualization) | Color system rules, overlap prevention, typography hierarchy, export rules, anti-pattern reference. CRITICAL for avoiding generic chart aesthetics. |
| **web-search** | Phase 4 (Web Research) | Dynamic sourcing of sector P/E ratios, industry context, market size data. Prevents stale hardcoded benchmarks. |

### Python Dependencies

```bash
# Install into the correct Python environment (see Section 3 for pitfalls)
pip install yfinance pandas numpy scipy plotly jinja2 requests beautifulsoup4 lxml
```

| Library | Purpose | Version Constraint | Critical Notes |
|---------|---------|-------------------|----------------|
| `yfinance` | Historical prices, stock info, earnings | >=0.2.30 | Requires `curl_cffi` and `peewee` as deps. Install can be slow (11MB download). |
| `pandas` | Time series manipulation, alignment | >=2.0.0 | Timezone handling is critical — see pitfalls. |
| `numpy` | Numerical computation | >=1.24.0 | Used for covariance, sqrt, array ops. |
| `scipy` | Statistical tests (Pearson, linear regression) | >=1.10.0 | `stats.pearsonr()` and `stats.linregress()` are the workhorses. |
| `plotly` | Chart generation (Python-side, optional) | >=5.0.0 | Can be used for server-side chart generation. For this workflow, we use Plotly.js client-side. |
| `jinja2` | HTML templating (optional) | >=3.1.0 | Only if generating HTML via templates. We embed data directly as JSON. |
| `requests` / `beautifulsoup4` | Web scraping fallback | >=4.3.0 | For sector P/E if web-search API unavailable. |
| `lxml` | HTML/XML parsing | >=4.9.0 | Backend for BeautifulSoup. |

### Frontend Dependencies (Embedded via CDN)

- **Plotly.js** v2.27.0 — Interactive charts (loaded from CDN, not bundled)
- **Google Fonts**: DM Serif Display (headings), JetBrains Mono (data/metrics), Inter (body)
- **No Tailwind CSS** — Custom CSS is preferred for bespoke editorial aesthetic

---

## 3. Environment Setup & Pitfalls

### CRITICAL: Python Interpreter Mismatch

**The #1 environment issue**: The system `python3` and the venv `python3` are DIFFERENT interpreters with DIFFERENT package directories.

```
System Python:  /usr/bin/python3 → Python 3.13 (system-managed, read-only)
Venv Python:    /home/z/.venv/bin/python3 → Python 3.12 (user-managed)
```

**Symptoms:**
- `pip install --break-system-packages yfinance` installs to `~/.local/lib/python3.13/` but `python3` runs from `/home/z/.venv/` which is Python 3.12
- `import yfinance` fails with `ModuleNotFoundError` despite successful install
- Running `python3 -c "import sys; print(sys.path)"` reveals the mismatch

**Resolution:**
```bash
# ALWAYS use the venv's pip to install packages
/home/z/.venv/bin/pip3 install yfinance

# ALWAYS use the venv's python to run scripts
/home/z/.venv/bin/python3 script.py

# Or activate the venv first
source /home/z/.venv/bin/activate
pip install yfinance
python script.py
```

**Pre-flight Check:**
```python
import sys
print(f"Python: {sys.executable}")
print(f"Version: {sys.version}")
print(f"Paths: {sys.path[:3]}")
# Verify the path starts with /home/z/.venv/
```

### yfinance Installation Timeout

`yfinance` depends on `curl_cffi` (~11MB) which can be slow to download. Installation may exceed 60-second timeouts.

**Resolution:**
- Use a 180+ second timeout for the install command
- Use `--no-cache-dir` if cached version is corrupted
- Install in isolation: `pip install yfinance` alone, not with other packages

### yfinance API Rate Limiting

Yahoo Finance may rate-limit or block requests if too many are made in quick succession.

**Resolution:**
- Add small delays between sequential API calls
- Cache data locally as JSON after first fetch
- If `stock.history()` returns empty DataFrame, retry after 30 seconds

---

## 4. Phase 1: Data Acquisition

### 4a. Historical Price Data (OHLCV)

```python
def fetch_price_history(ticker: str, years: int = 2) -> pd.DataFrame:
    """
    Fetch OHLCV data with IMMEDIATE timezone normalization.

    WHY: yfinance returns tz-aware indices (America/New_York).
    Merging with tz-naive data causes SILENT failures (empty results).
    This is the #1 source of bugs in the entire pipeline.

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

    # CRITICAL: Normalize timezone IMMEDIATELY after fetch
    if df.index.tz is not None:
        df.index = df.index.tz_localize(None)
    df.index.name = 'Date'

    return df
```

**PITFALL:** If you forget `tz_localize(None)`, ALL subsequent operations that merge this DataFrame with tz-naive data (earnings dates, benchmark data) will produce **silently empty results**. No error is raised — you just get 0 rows.

### 4b. Stock Info & Fundamentals

```python
def fetch_stock_info(ticker: str) -> dict:
    """
    Fetch current valuation metrics and company info.

    Returns dict with keys:
    - trailingPE, forwardPE, marketCap
    - sector, industry, shortName
    - beta, priceToBook, dividendYield
    - fiftyTwoWeekHigh, fiftyTwoWeekLow
    - averageVolume, sharesOutstanding

    NOTE: Some fields may be None. Always validate before use.
    Common missing fields: priceToBook, dividendYield (for growth stocks).
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

**PITFALL:** `stock.info` can return `None` for many fields depending on the stock. Always use `.get(k)` with default checks, never assume a field exists.

### 4c. Earnings Data

```python
def fetch_earnings_data(ticker: str) -> pd.DataFrame:
    """
    Fetch quarterly earnings history and dates.

    CRITICAL:
    - earnings_dates includes FUTURE dates with NaN for Reported EPS
    - MUST filter these out before analysis: dropna(subset=['Reported EPS'])
    - MUST normalize timezone on the Earnings Date column
    - yfinance may return earnings_dates as None for some tickers

    Returns:
        DataFrame with columns: Earnings Date, EPS Estimate, Reported EPS, Surprise(%)
    """
    stock = yf.Ticker(ticker)

    earnings_dates = stock.earnings_dates
    if earnings_dates is None:
        return pd.DataFrame()

    earnings_dates = earnings_dates.reset_index()
    earnings_dates.columns = ['Earnings Date', 'EPS Estimate', 'Reported EPS', 'Surprise(%)']

    # Remove FUTURE dates (no reported EPS yet)
    earnings_dates = earnings_dates.dropna(subset=['Reported EPS'])

    # Normalize timezone
    earnings_dates['Earnings Date'] = pd.to_datetime(
        earnings_dates['Earnings Date']
    ).dt.tz_localize(None)

    return earnings_dates
```

### 4d. Benchmark Data

```python
def fetch_benchmark(ticker: str = "SPY", years: int = 2) -> pd.DataFrame:
    """Fetch benchmark index prices for relative performance comparison."""
    return fetch_price_history(ticker, years)
```

---

## 5. Phase 2: Data Processing & Feature Engineering

### 5a. Timezone Normalization (Mandatory)

**ALREADY DONE in Phase 1.** Apply `tz_localize(None)` immediately after every yfinance fetch.

**Failure Symptom:** Silent empty results on merges or `.loc[]` lookups.

### 5b. Earnings Event Window Alignment

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
    - MUST handle edge cases: not enough past/future trading days

    Returns DataFrame with columns:
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
            'close_before': float(price_df.loc[date_before, 'Close']),
            'close_day_of': float(price_df.loc[actual_date, 'Close']),
            'close_day_after': float(price_df.loc[date_after, 'Close']),
            'close_window_before': float(price_df.loc[date_window_before, 'Close']),
            'close_window_after': float(price_df.loc[date_window_after, 'Close']),
            'volume_day_of': float(price_df.loc[actual_date, 'Volume']),
            'eps_estimate': float(row.get('EPS Estimate', 0)) if pd.notna(row.get('EPS Estimate')) else None,
            'eps_actual': float(row.get('Reported EPS', 0)) if pd.notna(row.get('Reported EPS')) else None,
            'eps_surprise_pct': float(row.get('Surprise(%)', 0)) if pd.notna(row.get('Surprise(%)')) else None,
        })

    result = pd.DataFrame(events)

    if len(result) > 0:
        result['return_1d'] = (result['close_day_of'] - result['close_before']) / result['close_before']
        result['return_2d'] = (result['close_day_after'] - result['close_before']) / result['close_before']
        result['return_window'] = (
            (result['close_window_after'] - result['close_window_before']) /
            result['close_window_before']
        )

    return result
```

**PITFALL:** Converting pandas values to `float()` is essential when serializing to JSON. `numpy.float64` is not JSON-serializable. Always wrap with `float()` or `int()` when building output dicts.

### 5c. Derived Metrics

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
    - cumulative_benchmark
    """
    df = price_df.copy()

    df['daily_return'] = df['Close'].pct_change()
    df['cumulative_return'] = (1 + df['daily_return']).cumprod() - 1
    df['volatility_20d'] = df['daily_return'].rolling(20).std() * np.sqrt(252)
    df['ma_50'] = df['Close'].rolling(50).mean()
    df['ma_200'] = df['Close'].rolling(200).mean()
    df['volume_ma_20'] = df['Volume'].rolling(20).mean()
    df['volume_ratio'] = df['Volume'] / df['volume_ma_20']

    if benchmark_df is not None:
        bench_aligned = benchmark_df['Close'].reindex(df.index, method='ffill')
        df['benchmark_return'] = bench_aligned.pct_change()
        df['excess_return'] = df['daily_return'] - df['benchmark_return']
        df['cumulative_benchmark'] = (1 + df['benchmark_return']).cumprod() - 1

    return df
```

### 5d. Data Quality Validation

```python
def validate_data(price_df, events_df) -> list[str]:
    """
    Run sanity checks. Return list of warnings (empty = all good).

    Checks:
    - Gaps > 5 days (holidays are 1-3 days)
    - Zero-volume days
    - Missing EPS data
    - Sufficient data length (>= 252 trading days)
    """
    warnings = []

    date_diffs = price_df.index.to_series().diff().dt.days
    large_gaps = date_diffs[date_diffs > 5]
    if len(large_gaps) > 0:
        warnings.append(f"Found {len(large_gaps)} gaps > 5 trading days.")

    zero_vol = (price_df['Volume'] == 0).sum()
    if zero_vol > 0:
        warnings.append(f"Found {zero_vol} zero-volume days.")

    if events_df is not None and len(events_df) > 0:
        missing_eps = events_df['eps_actual'].isna().sum()
        if missing_eps > 0:
            warnings.append(f"{missing_eps} earnings events missing actual EPS.")

    if len(price_df) < 252:
        warnings.append("Less than 1 year of trading data.")

    return warnings
```

---

## 6. Phase 3: Multi-Dimensional Analysis

### 6a. Event Study — Price Action Around Earnings

```python
def earnings_event_study(events_df: pd.DataFrame) -> dict:
    """
    Analyze stock returns around earnings announcements.

    Returns dict with:
    - total_events, mean_1d_return, median_1d_return, std_1d_return
    - positive_rate (% of events with positive return)
    - mean_2d_return, mean_window_return
    - best/worst_earnings_return
    - correlation_surprise_return (Pearson r)
    - correlation_p_value (statistical significance)
    - regression_slope, regression_r_squared
    """
    results = {
        'total_events': int(len(events_df)),
        'mean_1d_return': float(events_df['return_1d'].mean()),
        'median_1d_return': float(events_df['return_1d'].median()),
        'std_1d_return': float(events_df['return_1d'].std()),
        'positive_rate': float((events_df['return_1d'] > 0).mean()),
        'mean_2d_return': float(events_df['return_2d'].mean()),
        'mean_window_return': float(events_df['return_window'].mean()),
        'best_earnings_return': float(events_df['return_1d'].max()),
        'worst_earnings_return': float(events_df['return_1d'].min()),
    }

    clean = events_df.dropna(subset=['return_1d', 'eps_surprise_pct'])
    if len(clean) >= 3:
        corr, p_value = stats.pearsonr(clean['eps_surprise_pct'], clean['return_1d'])
        slope, intercept, r_value, p_val, std_err = stats.linregress(
            clean['eps_surprise_pct'], clean['return_1d']
        )
        results['correlation_surprise_return'] = float(corr)
        results['correlation_p_value'] = float(p_value)
        results['correlation_significant'] = bool(p_value < 0.05)
        results['regression_slope'] = float(slope)
        results['regression_r_squared'] = float(r_value ** 2)
        results['regression_p_value'] = float(p_val)

    return results
```

**Interpretation Guide:**

| Finding | Meaning |
|---------|---------|
| High positive rate (>70%) | Market consistently rewards earnings |
| Low positive rate (<40%) | Market "sells the news" routinely |
| Low surprise-return correlation (p > 0.05) | Stock moves on guidance/narrative, not just numbers |
| Negative correlation | Counter-intuitive: bigger beats → bigger sell-offs ("sell the news") |
| High volatility on earnings days | Earnings are a significant risk event |
| Window return >> 1-day return | The move extends beyond the announcement |

### 6b. Earnings-Timing Strategy Backtest (Fair Comparison)

**CRITICAL:** Always compare fairly. The naive comparison of cumulative returns is misleading because the earnings strategy is only invested ~23% of the time.

**MUST compute:**
- Time-in-market adjustment
- Annualized returns
- Sharpe ratio, Sortino ratio, Calmar ratio
- Maximum drawdown
- Win rate

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

    Returns dict with all metrics including risk-adjusted measures.
    See source skill for full implementation.
    """
    # [Full implementation in source skill — Section 4b]
    pass
```

**KEY INSIGHT FROM NVDA ANALYSIS:** The earnings-timing strategy produced **-42.4% cumulative returns** vs. buy-and-hold's **+76.4%**. This is because NVDA's gains are driven by sustained momentum between earnings, not quarterly event volatility. Missing 77% of trading days meant missing the bulk of the appreciation. This is a powerful, counter-intuitive finding that makes the infographic compelling.

### 6c. Valuation Benchmarking

```python
def valuation_analysis(
    stock_info: dict,
    sector_pe: float,   # MUST be dynamically sourced, NOT hardcoded
    index_pe: float      # MUST be dynamically sourced, NOT hardcoded
) -> dict:
    """
    Compare current valuation to sector and market benchmarks.

    CRITICAL: sector_pe and index_pe MUST come from web research
    (see Phase 4). Hardcoded values become stale within weeks.

    Returns dict with:
    - trailing_pe, forward_pe, sector_pe, index_pe
    - premium_vs_sector, premium_vs_market
    - earnings_growth_implied (if forward < trailing)
    - valuation_note
    """
    trailing_pe = stock_info.get('trailingPE')
    forward_pe = stock_info.get('forwardPE')

    results = {
        'trailing_pe': trailing_pe,
        'forward_pe': forward_pe,
        'sector_pe': sector_pe,
        'index_pe': index_pe,
        'trailing_premium_vs_sector': ((trailing_pe / sector_pe) - 1) * 100 if trailing_pe and sector_pe else None,
        'forward_premium_vs_sector': ((forward_pe / sector_pe) - 1) * 100 if forward_pe and sector_pe else None,
        'trailing_premium_vs_market': ((trailing_pe / index_pe) - 1) * 100 if trailing_pe and index_pe else None,
    }

    if trailing_pe and forward_pe and forward_pe < trailing_pe:
        results['earnings_growth_implied'] = (trailing_pe / forward_pe - 1) * 100
        results['valuation_note'] = (
            f"Forward P/E below trailing suggests expected earnings growth. "
            f"Implies {(trailing_pe / forward_pe - 1) * 100:.1f}% earnings growth."
        )

    return results
```

**PITFALL:** The source skill hardcoded `sector_pe = 39.1`. The ACTUAL semiconductor sector trailing P/E as of January 2026 is **100.2x** (Damodaran/NYU Stern). Using the hardcoded value would show NVDA at a 19% discount to sector, when in reality it's at a **69% discount**. This completely changes the valuation narrative.

### 6d. Risk-Adjusted Performance Metrics

```python
def risk_metrics(
    price_df: pd.DataFrame,
    benchmark_df: pd.DataFrame = None,
    risk_free_rate: float = 0.045
) -> dict:
    """
    Compute comprehensive risk-adjusted return metrics.

    Non-negotiable metrics:
    - total_return, annualized_return, annualized_volatility
    - sharpe_ratio, sortino_ratio, calmar_ratio
    - max_drawdown
    - beta, alpha, information_ratio (if benchmark provided)
    - positive_days_pct, best_day, worst_day

    Returns dict with all metrics as float values.
    """
    # [Full implementation in source skill — Section 4d]
    pass
```

**KEY INSIGHT:** NVDA had a strong +56.9% total return but **negative alpha (-6.2%)**. This means the stock underperformed on a risk-adjusted basis relative to its beta exposure. The high beta (2.01) means the stock amplifies market moves 2x, and the total return is largely explained by this beta exposure during a bull market, not by idiosyncratic alpha. This nuance is critical for honest analysis.

### 6e. Fundamental Assessment Synthesis

The synthesis should evaluate across 5 dimensions:

| Dimension | Criteria | Ratings |
|-----------|----------|---------|
| Earnings Quality | Beat consistency, surprise magnitude, market reaction rationality | High / Medium / Low |
| Price Action Integrity | Sustained gains between earnings? Momentum vs fundamental-driven? | Sustainable / Speculative |
| Valuation Reasonableness | Premium justified by growth? Historical context? | Reasonable / Premium / Expensive |
| Risk Profile | Max drawdown, Sharpe > 1.0?, Alpha generation | Conservative / Moderate / Aggressive |
| Industry Fundamentals | Real demand, competitive moat, structural tailwinds | Strong / Moderate / Weak |

**Overall Verdict:** Sustainable / Mixed / Hype

---

## 7. Phase 4: Web Research Augmentation

### Why Web Research Is Non-Negotiable

The source skill provided placeholder functions for web research. In practice, this phase is **essential** for:

1. **Current sector P/E ratios** — Hardcoded values become stale within weeks
2. **Industry market size** — Provides context for growth narratives
3. **Competitive landscape** — Validates moat claims
4. **AI adoption metrics** — Grounds hype assessments in real data

### Search Queries

```python
SEARCH_QUERIES = {
    "sector_pe": "S&P 500 {sector} sector trailing P/E ratio {year}",
    "sector_pe_alt": "{sector} industry average P/E ratio {year} Damodaran",
    "ai_market_size": "global AI market size {year} forecast",
    "infrastructure_spending": "AI infrastructure spending forecast {year} IDC Gartner",
    "competitive_landscape": "{company} market share GPU AI {year}",
    "adoption_rate": "enterprise AI adoption rate {year} McKinsey survey",
    "revenue_growth": "{company} data center revenue growth {year}"
}
```

### Key Data Sources (Verified in Practice)

| Source | Data Type | URL Pattern | Reliability |
|--------|-----------|-------------|-------------|
| **Damodaran/NYU Stern** | Sector P/E, industry-level valuation | `pages.stern.nyu.edu/~adamodar/` | Excellent — updated annually, academic rigor |
| **multpl.com** | S&P 500 P/E (Shiller data) | `multpl.com` | Excellent — real-time, historical context |
| **Precedence Research** | AI market size, CAGR | `precedenceresearch.com` | Good — detailed forecasts |
| **IDC** | AI spending forecasts | `idc.com` | Good — enterprise-focused |
| **McKinsey** | Enterprise AI adoption | `mckinsey.com` | Good — survey-based |
| **Jon Peddie Research** | GPU market share | `jonpeddie.com` | Good — hardware-specific |
| **Macrotrends** | Revenue history | `macrotrends.net` | Good — clean historical data |

### Practical Resolution When Web Search Fails

During the NVDA analysis, the web-search API timed out. The subagent fell back to **direct page reading** of authoritative sources. This produced accurate data from:

- Damodaran's pedata.html (sector P/E ratios for 66 semiconductor firms)
- multpl.com (S&P 500 Shiller P/E)
- Precedence Research (AI market size)
- McKinsey Global AI Survey 2025

**Lesson:** Always have a fallback strategy. Direct page reading is a viable alternative when search APIs fail.

### Data Extracted for NVDA (June 2026 Reference)

| Metric | Value | Source |
|--------|-------|--------|
| S&P 500 Trailing P/E | 32.01 | multpl.com |
| Semiconductor Sector Trailing P/E | 100.18 | Damodaran/NYU Stern |
| Semiconductor Sector Forward P/E | 37.29 | Damodaran/NYU Stern |
| Global AI Market (2025) | $757.58B | Precedence Research |
| Global AI Market (2026) | $900B | Precedence Research |
| AI Market CAGR (2026-2035) | 18.73% | Precedence Research |
| Hyperscaler AI Capex (2025) | $300B+ | Company filings |
| NVIDIA Discrete GPU Share | 92% | Jon Peddie Research |
| NVIDIA AI GPU Share | >80% | Industry reports |
| Enterprise AI Scaling Rate | 33% | McKinsey 2025 |
| NVIDIA FY2024 Revenue | $60.92B | Macrotrends |
| NVIDIA FY2025 Revenue | $130.50B | Macrotrends |
| NVIDIA FY2026 Revenue | $215.94B | Macrotrends |

---

## 8. Phase 5: Avant-Garde HTML Infographic Generation

### Design Philosophy

This is where the **charts skill** guidance converges with the **stock-analysis** template. The goal is NOT to use the source skill's HTML template verbatim — it's a starting point that must be elevated.

### Design Decisions & Rationale

| Decision | Source Skill Default | Our Enhancement | Why |
|----------|---------------------|-----------------|-----|
| Background color | `#0d0d0d` | `#0a0a0f` | Darker, more cinematic, reduces eye strain on large monitors |
| Background type | Flat gradient | Ambient radial gradients (fixed position) | Creates depth without distraction; doesn't scroll with content |
| Section headers | Plain `<h2>` with border-bottom | Numbered (01, 02, 03...) + horizontal rule | Editorial convention; numbers aid scanning; line creates visual rhythm |
| Metric display | Card grid with hover transforms | Grid with 1px gap (shared border) + subtle hover | Eliminates card-shadow cliché; 1px gap creates unified surface |
| Insight boxes | One style (orange/warning) | Three variants: green (positive), red (negative), amber (warning) | Color-coded semantic meaning; faster comprehension |
| Recommendation cards | Single style with border | Distinct buy/hold/avoid with badge + color-coded borders | Investor-type specific; badges provide instant categorization |
| Chart layout | Single container style | Container with monospaced title label | Adds editorial precision; "CHART TITLE" format feels data-journalistic |
| Animations | None | Scroll-reveal (opacity + translateY) | Progressive disclosure; reduces cognitive load on first view |
| Verdict banner | Simple highlighted box | Left-accent bar + gradient background + strong typography | First thing user reads after hero — must command attention |
| Context cards | Not in source skill | 2x2 grid with big number + label + detail | "Big number" journalism technique — scan first, read second |
| Revenue chart | Not in source skill | Bar chart with gradient opacity | Shows NVIDIA's explosive growth trajectory — essential context |

### Typography System

```css
--font-serif: 'DM Serif Display', Georgia, serif;     /* Headings, verdict text */
--font-mono: 'JetBrains Mono', 'Courier New', monospace; /* Data, metrics, labels */
--font-sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif; /* Body text */
```

**Hierarchy:**
- **H1 (hero):** DM Serif Display, 3.75rem, weight 400 — editorial gravitas
- **H2 (sections):** DM Serif Display, 1.75rem, weight 400 — with section number
- **Metric values:** JetBrains Mono, 1.75rem, weight 700 — precision and authority
- **Metric labels:** JetBrains Mono, 0.7rem, uppercase, tracking 0.1em — technical labeling
- **Body text:** Inter, 0.95rem, weight 400 — clean readability
- **Insight text:** Inter, 0.95rem — with bold highlights for key phrases

### Color System

```css
--bg-primary: #0a0a0f;           /* Deepest background */
--bg-secondary: #12121e;         /* Card/cell backgrounds */
--bg-tertiary: #1a1a2e;          /* Hover states */
--bg-card: rgba(26, 26, 46, 0.6); /* Semi-transparent cards */
--text-primary: #f0ece4;         /* Warm sand — not pure white */
--text-secondary: #8a8a9a;       /* Body text */
--text-muted: #5a5a6a;           /* Labels, metadata */
--accent-positive: #00d4aa;      /* Terminal green — positive values */
--accent-negative: #ff4757;      /* Desaturated crimson — negative values */
--accent-warning: #ffa502;       /* Muted amber — warnings */
--accent-blue: #3498db;          /* Scatter points, regression lines */
--border-subtle: rgba(255, 255, 255, 0.06);  /* Barely visible borders */
--border-accent: rgba(0, 212, 170, 0.3);     /* Active/selected borders */
```

**Key Rule:** Never use pure white (#fff) for text — it creates harsh contrast against dark backgrounds. Warm sand (#f0ece4) is more readable and editorial.

### Chart Architecture (9 Charts)

The infographic contains **9 interactive Plotly.js charts**, each with specific design requirements:

| # | Chart | Plotly Type | Key Design Choices |
|---|-------|-------------|-------------------|
| 1 | Price + MAs + Earnings | scatter (lines) + shapes (vlines) | Green price line, dashed MA lines, colored earnings vlines with ▲/▼ annotations |
| 2 | Cumulative Returns | scatter (lines) + fill | Area fill for NVDA, dashed line for benchmark, unified hover |
| 3 | Earnings-Day Returns | bar | Color-coded green/red bars with auto text labels |
| 4 | EPS Surprise vs Return | scatter (markers) + trend line | Green/red markers by return direction, dashed regression with R² |
| 5 | Strategy Backtest | scatter (lines+markers) | Red for strategy (losses), green dashed for buy-and-hold |
| 6 | Valuation Comparison | bar (grouped) | NVDA green, sector amber, S&P gray — grouped by metric |
| 7 | Rolling Volatility | scatter (lines) + fill | Amber line with subtle fill, emphasizes risk spikes |
| 8 | Volume Comparison | bar (2 bars) | Gray for average, green for earnings day — dramatic contrast |
| 9 | Revenue Trajectory | bar | Gradient opacity (light→dark) showing growth acceleration |

### Plotly.js Configuration (Shared)

```javascript
const plotlyConfig = {
    displayModeBar: true,
    displaylogo: false,
    responsive: true,
    modeBarButtonsToRemove: ['lasso2d', 'select2d']
};

const darkLayout = {
    paper_bgcolor: 'rgba(0,0,0,0)',      // Transparent — uses CSS background
    plot_bgcolor: 'rgba(0,0,0,0)',       // Transparent
    font: { family: "'Inter', sans-serif", color: '#8a8a9a', size: 12 },
    xaxis: { gridcolor: 'rgba(255,255,255,0.04)', zerolinecolor: 'rgba(255,255,255,0.06)' },
    yaxis: { gridcolor: 'rgba(255,255,255,0.04)', zerolinecolor: 'rgba(255,255,255,0.06)' },
    margin: { l: 60, r: 30, t: 40, b: 50 },
    hoverlabel: {
        bgcolor: '#1a1a2e',
        bordercolor: '#333',
        font: { color: '#f0ece4', family: "'JetBrains Mono', monospace", size: 12 }
    }
};
```

### Data Embedding Strategy

**DO NOT** use Plotly's Python `fig.to_html()` to embed charts. This creates massive HTML bloat because each chart includes a full Plotly.js bundle reference.

**Instead:** Embed raw data as JSON arrays in the HTML, then use client-side JavaScript to construct Plotly charts. This:
- Produces smaller HTML files (~82KB vs potentially 500KB+)
- Allows shared Plotly.js CDN reference (loaded once)
- Enables consistent dark theme across all charts
- Makes the HTML more maintainable

```python
# In Python: Serialize data as JSON strings for embedding
price_dates = json.dumps([p['date'] for p in data['price_data']])
price_close = json.dumps([round(p['close'], 2) for p in data['price_data']])
# ... then inject into HTML template via f-string
```

### Scroll Reveal Animation

```javascript
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            entry.target.classList.add('visible');
        }
    });
}, { threshold: 0.1, rootMargin: '0px 0px -40px 0px' });

document.querySelectorAll('.reveal').forEach(el => observer.observe(el));
```

```css
.reveal {
    opacity: 0;
    transform: translateY(24px);
    transition: opacity 0.6s ease, transform 0.6s ease;
}
.reveal.visible {
    opacity: 1;
    transform: translateY(0);
}
```

---

## 9. Phase 6: QA Validation & Delivery

### Pre-Delivery Checks

```python
def run_qa_checks(data: dict, html: str, html_path: str) -> list[str]:
    """
    Final quality assurance. Return list of issues found.

    Checks:
    1. Data integrity (price data length, earnings event count)
    2. Strategy sanity (unrealistic returns)
    3. Valuation sanity (extreme P/E)
    4. HTML structure (valid tags, all charts present)
    5. Design compliance (fonts, colors, dark theme)
    6. Statistical validity (sample size, significance)
    7. File size reasonableness
    """
    issues = []

    # Data integrity
    if len(data['price_data']) < 252:
        issues.append("WARNING: Less than 1 year of price data.")
    if len(data['events_data']) < 4:
        issues.append("WARNING: Fewer than 4 earnings events.")

    # Strategy sanity
    sr = data['strategy_results']
    if sr['strategy_cumulative_return'] > 10:
        issues.append("ERROR: Strategy return > 1000%. Likely a calculation bug.")
    if abs(sr['buyhold_cumulative_return']) > 20:
        issues.append("WARNING: Buy-and-hold return > 2000%. Verify data period.")

    # Valuation sanity
    val = data['valuation']
    if val.get('trailing_pe') and val['trailing_pe'] > 500:
        issues.append("WARNING: P/E > 500x. Verify data.")

    # HTML structure
    if '<html' not in html.lower():
        issues.append("ERROR: Report HTML appears malformed.")
    if 'plotly' not in html.lower():
        issues.append("ERROR: Plotly.js not referenced.")

    # Chart containers (9 charts required)
    required_charts = [
        'chart-price', 'chart-cumulative', 'chart-earnings',
        'chart-scatter', 'chart-strategy', 'chart-valuation',
        'chart-volatility', 'chart-volume', 'chart-revenue'
    ]
    for chart_id in required_charts:
        if chart_id not in html:
            issues.append(f"ERROR: Chart container '{chart_id}' missing.")

    # Design compliance
    if 'DM Serif Display' not in html:
        issues.append("WARNING: Serif font not referenced.")
    if 'JetBrains Mono' not in html:
        issues.append("WARNING: Mono font not referenced.")
    if '00d4aa' not in html:
        issues.append("WARNING: Accent color not applied.")
    if 'disclaimer' not in html.lower():
        issues.append("WARNING: Disclaimer may be missing.")

    # Statistical validity
    es = data['event_study']
    if es['total_events'] < 8:
        issues.append(f"NOTE: Only {es['total_events']} events. Correlations are directional.")

    return issues
```

---

## 10. Lessons Learned & Critical Pitfalls

### Environment Pitfalls

| # | Pitfall | Symptom | Resolution |
|---|---------|---------|------------|
| 1 | **Python interpreter mismatch** | `pip install` succeeds but `import` fails | Always use venv's pip (`/home/z/.venv/bin/pip3`) and venv's python (`/home/z/.venv/bin/python3`) |
| 2 | **yfinance install timeout** | pip install exceeds 60s timeout | Use 180s+ timeout; install yfinance alone, not with other packages |
| 3 | **System Python is read-only** | `--break-system-packages` needed but installs to wrong dir | Never use system Python. Always use venv. |
| 4 | **Missing yfinance deps** | yfinance needs curl_cffi, peewee, protobuf | Let pip resolve deps automatically; don't use `--no-deps` |

### Data Pipeline Pitfalls

| # | Pitfall | Symptom | Resolution |
|---|---------|---------|------------|
| 5 | **Timezone mismatch** | Empty merge results, 0 events found | `tz_localize(None)` IMMEDIATELY after every yfinance fetch |
| 6 | **Future earnings dates** | NaN in analysis, inflated event count | `dropna(subset=['Reported EPS'])` before processing |
| 7 | **numpy types not JSON-serializable** | `TypeError: Object of type float64 is not JSON serializable` | Wrap all values with `float()` or `int()` when building output dicts |
| 8 | **NaN values in JSON** | `json.dumps` fails on NaN | Check `pd.notna()` before including; convert to `None` |
| 9 | **Missing stock info fields** | `TypeError` when formatting None as float | Always use `.get(key)` with None checks; format strings conditionally |
| 10 | **Earnings dates return None** | `stock.earnings_dates` is None for some tickers | Check for None before processing; return empty DataFrame |

### Analysis Pitfalls

| # | Pitfall | Symptom | Resolution |
|---|---------|---------|------------|
| 11 | **Hardcoded sector P/E** | Valuation analysis uses stale numbers | ALWAYS dynamically source via web research; cite source and date |
| 12 | **Unfair backtest comparison** | Strategy appears to underperform because it's invested less | Compute time-in-market, annualized returns, Sharpe/Sortino/Calmar |
| 13 | **No significance testing** | False confidence in correlations | Report p-value alongside every correlation; flag non-significance |
| 14 | **Negative alpha misinterpretation** | Negative alpha on a stock with positive total return | Explain: high beta during bull market explains return; alpha is risk-adjusted |
| 15 | **"Sell the news" oversight** | Low positive earnings rate seems contradictory | Explicitly explain this common market dynamic; it's not a bug |
| 16 | **Look-ahead bias** | Using today's P/E for historical decisions | Use point-in-time data only; note limitations |

### Design Pitfalls

| # | Pitfall | Symptom | Resolution |
|---|---------|---------|------------|
| 17 | **Using fig.to_html() for each chart** | Massive HTML bloat (500KB+) | Embed raw JSON data; construct charts client-side with shared Plotly.js CDN |
| 18 | **Pure white text on dark bg** | Harsh contrast, eye strain | Use warm sand (#f0ece4) for primary text |
| 19 | **Bootstrap-style card grids** | Generic, template-looking | Use 1px-gap grid (shared borders) for metrics; no box shadows |
| 20 | **Single insight-box style** | All insights look the same | Three variants: green (positive), red (negative), amber (warning) |
| 21 | **No scroll animations** | User overwhelmed by content density | IntersectionObserver reveal animations with 24px translateY |
| 22 | **Missing revenue chart** | AI narrative lacks visual evidence | Add revenue trajectory chart — it's the most compelling visual |
| 23 | **Flat background** | Lifeless, despite dark theme | Add ambient radial gradients (fixed position) for depth |

### Workflow Pitfalls

| # | Pitfall | Symptom | Resolution |
|---|---------|---------|------------|
| 24 | **Running full pipeline in one script** | Timeout or memory issues | Split into phases: (1) data fetch + analysis → JSON, (2) HTML generation from JSON |
| 25 | **Web search API timeouts** | Subagent can't fetch sector data | Fall back to direct page reading of authoritative sources |
| 26 | **Not validating data before analysis** | Events_df is empty, silent downstream failures | Check `len(events_df) > 0` before running event study/backtest |
| 27 | **Rounded JSON values** | Loss of precision in chart data | Round to 2 decimal places — sufficient for charts, keeps file size manageable |

---

## 11. Design System Reference

### CSS Custom Properties

```css
:root {
    /* Backgrounds */
    --bg-primary: #0a0a0f;           /* Page background */
    --bg-secondary: #12121e;         /* Card/cell backgrounds */
    --bg-tertiary: #1a1a2e;          /* Hover states */
    --bg-card: rgba(26, 26, 46, 0.6); /* Semi-transparent cards */

    /* Text */
    --text-primary: #f0ece4;         /* Headlines, key data */
    --text-secondary: #8a8a9a;       /* Body text, descriptions */
    --text-muted: #5a5a6a;           /* Labels, metadata */

    /* Accents */
    --accent-positive: #00d4aa;      /* Terminal green — gains, beats, buy */
    --accent-negative: #ff4757;      /* Desaturated crimson — losses, sells */
    --accent-warning: #ffa502;       /* Muted amber — warnings, caution */
    --accent-blue: #3498db;          /* Secondary data, regression lines */
    --accent-purple: #a855f7;        /* Tertiary accent (sparingly) */

    /* Borders */
    --border-subtle: rgba(255, 255, 255, 0.06);   /* Between cells, sections */
    --border-accent: rgba(0, 212, 170, 0.3);      /* Active states, verdict */

    /* Typography */
    --font-serif: 'DM Serif Display', Georgia, serif;
    --font-mono: 'JetBrains Mono', 'Courier New', monospace;
    --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
}
```

### Responsive Breakpoints

```css
/* Tablet */
@media (max-width: 768px) {
    .container { padding: 0 1.25rem; }
    .hero h1 { font-size: 2rem; }
    .metric-strip { grid-template-columns: repeat(2, 1fr); }
    .context-grid { grid-template-columns: 1fr; }
}

/* Mobile */
@media (max-width: 480px) {
    .metric-strip { grid-template-columns: 1fr; }
    .hero-meta { flex-direction: column; gap: 0.5rem; }
}
```

### Component Patterns

**Section Header:**
```html
<div class="section-header">
    <span class="section-number">01</span>
    <h2 class="section-title">Section Title</h2>
    <div class="section-line"></div>
</div>
```

**Insight Box (3 variants):**
```html
<!-- Positive (green) -->
<div class="insight-box green">
    <div class="insight-label">Key Insight</div>
    <p class="insight-text">Content with <strong>highlights</strong></p>
</div>

<!-- Negative (red) -->
<div class="insight-box red">
    <div class="insight-label">Warning</div>
    <p class="insight-text">Content with <strong>highlights</strong></p>
</div>

<!-- Neutral (amber) -->
<div class="insight-box">
    <div class="insight-label">Note</div>
    <p class="insight-text">Content</p>
</div>
```

**Context Card (Big Number):**
```html
<div class="context-card">
    <div class="context-card-value">$757B</div>
    <div class="context-card-label">Global AI Market (2025)</div>
    <div class="context-card-detail">Supporting context and detail</div>
</div>
```

**Recommendation Card:**
```html
<div class="rec-card buy">  <!-- or "hold" or "avoid" -->
    <div class="rec-badge">Long-Term (3+ Years)</div>
    <div class="rec-title">BUY — With Conviction</div>
    <ul class="rec-list">
        <li>Reason 1</li>
        <li>Reason 2</li>
    </ul>
</div>
```

---

## 12. Chart Architecture Reference

### Chart Data Serialization (Python Side)

```python
# Serialize price data for embedding
price_dates = json.dumps([p['date'] for p in data['price_data']])
price_close = json.dumps([round(p['close'], 2) for p in data['price_data']])
price_ma50 = json.dumps([round(p['ma_50'], 2) if p['ma_50'] else 'null' for p in data['price_data']])
price_ma200 = json.dumps([round(p['ma_200'], 2) if p['ma_200'] else 'null' for p in data['price_data']])
price_cum_ret = json.dumps([round(p['cumulative_return']*100, 2) if p['cumulative_return'] is not None else 'null' for p in data['price_data']])
price_cum_bench = json.dumps([round(p['cumulative_benchmark']*100, 2) if p['cumulative_benchmark'] is not None else 'null' for p in data['price_data']])
price_vol20d = json.dumps([round(p['volatility_20d']*100, 2) if p['volatility_20d'] else 'null' for p in data['price_data']])

events_dates = json.dumps([e['earnings_date'] for e in data['events_data']])
events_ret1d = json.dumps([round(e['return_1d']*100, 2) for e in data['events_data']])
events_surprise = json.dumps([round(e['eps_surprise_pct'], 2) if e['eps_surprise_pct'] else 'null' for e in data['events_data']])
events_vol = json.dumps([e['volume_day_of'] for e in data['events_data']])

# Strategy cumulative returns (computed from trade returns)
trades_cum_ret = []
cum = 0
for t in data['trades_data']:
    cum = (1 + cum) * (1 + t['return']) - 1
    trades_cum_ret.append(round(cum * 100, 2))
trades_cum_ret_json = json.dumps(trades_cum_ret)
```

### Earnings Markers on Price Chart (Plotly.js)

```javascript
// Vertical lines for each earnings event
const priceShapes = earningsDates.map((d, i) => ({
    type: 'line', x0: d, x1: d, y0: 0, y1: 1, yref: 'paper',
    line: {
        color: earningsReturns[i] > 0 ? 'rgba(0,212,170,0.35)' : 'rgba(255,71,87,0.35)',
        width: 1, dash: 'dot'
    }
}));

// Triangle annotations (▲ for positive, ▼ for negative)
const priceAnnotations = earningsDates.map((d, i) => ({
    x: d, y: 1.02, yref: 'paper',
    text: earningsReturns[i] > 0 ? '▲' : '▼',
    showarrow: false,
    font: { color: earningsReturns[i] > 0 ? '#00d4aa' : '#ff4757', size: 10 }
}));
```

### Regression Line on Scatter Chart (Client-side)

```javascript
// Compute linear regression in JavaScript (avoids Python dependency for this)
if (cleanX.length >= 3) {
    const n = cleanX.length;
    const sumX = cleanX.reduce((a, b) => a + b, 0);
    const sumY = cleanY.reduce((a, b) => a + b, 0);
    const sumXY = cleanX.reduce((a, x, i) => a + x * cleanY[i], 0);
    const sumX2 = cleanX.reduce((a, x) => a + x * x, 0);
    const slope = (n * sumXY - sumX * sumY) / (n * sumX2 - sumX * sumX);
    const intercept = (sumY - slope * sumX) / n;
    // Add trend line trace
}
```

---

## 13. Reusable Code Templates

### Full Pipeline Orchestration

```python
def run_full_analysis(
    ticker: str,
    years: int = 2,
    output_dir: str = "/home/z/my-project/download"
) -> dict:
    """
    Complete pipeline from data acquisition to HTML infographic generation.

    Usage:
        results = run_full_analysis("NVDA", years=2)

    Returns:
        {
            'analysis_data': dict (JSON-serializable),
            'html_path': str,
            'qa_issues': list[str]
        }
    """
    os.makedirs(output_dir, exist_ok=True)

    # Phase 1: Data Acquisition
    print(f"[1/6] Fetching data for {ticker}...")
    price_df = fetch_price_history(ticker, years)
    stock_info = fetch_stock_info(ticker)
    earnings_dates = fetch_earnings_data(ticker)
    benchmark_df = fetch_benchmark("SPY", years)

    # Phase 2: Processing
    print("[2/6] Processing data...")
    warnings = validate_data(price_df, earnings_dates)
    for w in warnings:
        print(f"  WARNING: {w}")

    price_df = compute_derived_metrics(price_df, benchmark_df)
    events_df = align_earnings_to_prices(earnings_dates, price_df)

    # Phase 3: Analysis
    print("[3/6] Running analyses...")
    event_study = earnings_event_study(events_df)
    strategy = backtest_earnings_strategy(price_df, earnings_dates)
    risk = risk_metrics(price_df, benchmark_df)

    # Phase 4: Web Research (sector P/E, industry context)
    print("[4/6] Conducting web research...")
    # Agent should use web-search skill or subagent here
    # sector_data = get_sector_pe_via_web_search(...)
    # industry_context = extract_industry_context(...)

    # Phase 5: HTML Generation
    print("[5/6] Generating HTML infographic...")
    # Serialize data to JSON
    # Generate HTML from template with embedded data
    # Save to output_dir

    # Phase 6: QA
    print("[6/6] Running QA checks...")
    # qa_issues = run_qa_checks(...)

    return {
        'analysis_data': {},  # JSON-serializable analysis results
        'html_path': '',      # Path to generated HTML
        'qa_issues': []       # List of QA issues
    }
```

### JSON Serialization Helper

```python
def convert_for_json(obj):
    """Convert numpy/pandas types to JSON-serializable Python types."""
    if isinstance(obj, (np.integer,)):
        return int(obj)
    elif isinstance(obj, (np.floating, float)):
        if np.isnan(obj) or np.isinf(obj):
            return None
        return float(obj)
    elif isinstance(obj, (np.bool_,)):
        return bool(obj)
    elif isinstance(obj, pd.Timestamp):
        return obj.strftime('%Y-%m-%d')
    elif isinstance(obj, datetime):
        return obj.strftime('%Y-%m-%d')
    return str(obj)
```

---

## 14. Quality Checklist

### Before delivering any analysis, verify:

**Data Integrity**
- [ ] Timezone normalized for all yfinance data (`tz_localize(None)`)
- [ ] No future earnings dates included (`dropna(subset=['Reported EPS'])`)
- [ ] No duplicate dates in price DataFrame
- [ ] No NaN in critical columns (Close, Volume, EPS) — or properly handled
- [ ] At least 252 trading days of data
- [ ] At least 8 earnings events (4+ for annual analysis)

**Statistical Validity**
- [ ] Every correlation has p-value reported
- [ ] Sample size noted in all statistical claims
- [ ] Non-significant correlations explicitly flagged
- [ ] Confidence intervals computed where appropriate

**Backtesting Fairness**
- [ ] Time-in-market calculated and reported
- [ ] Annualized returns computed (not just cumulative)
- [ ] Sharpe ratio calculated
- [ ] Sortino ratio calculated
- [ ] Calmar ratio calculated
- [ ] Maximum drawdown reported
- [ ] Win rate reported

**Valuation Accuracy**
- [ ] Sector P/E sourced dynamically (NOT hardcoded)
- [ ] Source and date cited for all benchmarks
- [ ] Forward P/E vs trailing P/E comparison included
- [ ] Implied growth rate calculated if forward < trailing
- [ ] Premium/discount vs sector AND vs market reported

**Visualization Quality**
- [ ] All 9 interactive Plotly.js charts rendered
- [ ] Charts use transparent backgrounds (not white)
- [ ] Dark theme with proper color palette
- [ ] Typography: DM Serif Display + JetBrains Mono + Inter
- [ ] Responsive design (mobile-friendly)
- [ ] Hover states and tooltips functional
- [ ] Earnings markers on price chart (▲/▼)
- [ ] Revenue trajectory chart included (not in source skill — added value)

**Report Completeness**
- [ ] Executive verdict banner with key insight
- [ ] Metric strip with 6+ key numbers
- [ ] All narrative sections present with 150+ words each
- [ ] Three insight-box variants used (green/red/amber)
- [ ] Investment recommendations by investor profile (buy/hold/avoid)
- [ ] AI industry context section with big-number cards
- [ ] Methodology documented with sources
- [ ] Disclaimer present
- [ ] HTML self-contained (no broken images/links)

**Edge Cases**
- [ ] Macro context noted (rate environment, sector trends)
- [ ] Negative alpha explained (not just reported)
- [ ] "Sell the news" dynamic acknowledged
- [ ] Look-ahead bias prevented
- [ ] Survivorship bias acknowledged
- [ ] Small sample caveat (if < 10 events)

**Code Quality**
- [ ] All values wrapped with `float()` or `int()` for JSON serialization
- [ ] `pd.notna()` checks before including values
- [ ] `.get()` with None checks for stock_info fields
- [ ] Empty DataFrame checks before analysis
- [ ] Consistent use of venv Python interpreter

---

## Appendix A: File Structure

```
/home/z/my-project/download/
├── NVDA_Earnings_Valuation_Analysis.html   # Main infographic (self-contained, ~82KB)
├── nvda_analysis_data.json                  # Serialized analysis data
└── SKILL_Infographic_Stock_Analysis.md      # This skill document
```

## Appendix B: Execution Timeline (Reference)

| Phase | Duration | Notes |
|-------|----------|-------|
| Environment Setup | 5-10 min | Install yfinance (slow download), verify venv |
| Data Acquisition | 1-2 min | yfinance API calls (4 requests) |
| Data Processing | <1 min | Pandas operations on ~500 rows |
| Analysis | <1 min | Statistical computations |
| Web Research | 3-5 min | Sector P/E, industry context (subagent) |
| HTML Generation | 1-2 min | Data serialization + template rendering |
| QA Validation | <1 min | Automated checks |
| **Total** | **~15-20 min** | |

## Appendix C: Extending to Other Tickers

To analyze a different stock (e.g., AAPL, MSFT, GOOGL):

1. Change `TICKER = "NVDA"` to desired ticker
2. Web search for that company's specific industry context
3. Adjust sector P/E to the correct sector (not Technology for all)
4. Revenue trajectory data will differ — research company-specific numbers
5. Recommendation cards should reflect company-specific fundamentals
6. The "AI narrative" section may not apply — replace with relevant industry theme

**Sectors other than Technology:**
- Healthcare: Use Healthcare sector P/E, replace AI context with drug pipeline/FDA dynamics
- Financials: Use Financials sector P/E, replace AI context with rate sensitivity/loan growth
- Energy: Use Energy sector P/E, replace AI context with commodity price dynamics
- Consumer: Use Consumer Discretionary sector P/E, replace AI context with spending trends

---

*Version 1.0 | Created: June 2026 | Domain: Quantitative Finance + Data Journalism + Avant-Garde UI/UX | Based on: stock-analysis SKILL.md v3.0, charts skill, web-search skill, and practical execution experience*

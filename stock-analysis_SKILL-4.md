---
name: stock-analysis-sustainable-earnings
description: >
  Earnings‑Event Stock Analysis & AI Sector Valuation.
  Guides an AI agent through a rigorous, multi‑phase pipeline that analyzes a stock's price behaviour around earnings events,
  benchmarks valuation against sector/indices, backtests an earnings‑timing strategy (with fair time‑in‑market adjustment),
  and produces a comprehensive report answering: "Is this stock's performance driven by sustainable fundamentals or short‑term hype?"
  Applicable Tickers: Any publicly traded equity.
  Time Horizon: 2+ years of historical data recommended.
  Requires: Python, pandas, yfinance, matplotlib, seaborn, scipy, jinja2.
version: 2.0
---

## Table of Contents

1. [Prerequisites & Environment](#1-prerequisites--environment)
2. [Phase 1: Data Acquisition](#2-phase-1-data-acquisition)
3. [Phase 2: Data Processing & Feature Engineering](#3-phase-2-data-processing--feature-engineering)
4. [Phase 3: Multi‑Dimensional Analysis](#4-phase-3-multi-dimensional-analysis)
5. [Phase 4: Visualisation](#5-phase-4-visualisation)
6. [Phase 5: Report Generation](#6-phase-5-report-generation)
7. [Phase 6: Validation & QA](#7-phase-6-validation--qa)
8. [Edge Cases, Pitfalls & Augmented Insights](#8-edge-cases-pitfalls--augmented-insights)
9. [Reusable Code Templates](#9-reusable-code-templates)
10. [Quality Checklist](#10-quality-checklist)

---

## 1. Prerequisites & Environment

### Python Dependencies

```bash
pip install yfinance pandas numpy matplotlib seaborn scipy jinja2 requests beautifulsoup4
```

| Library           | Purpose                                      |
|-------------------|----------------------------------------------|
| `yfinance`        | Historical prices, stock info, earnings     |
| `pandas`          | Time series manipulation, alignment         |
| `numpy`           | Numerical operations                         |
| `matplotlib`/`seaborn` | Static visualisations                     |
| `scipy`           | Statistical tests (t‑test, Pearson)         |
| `jinja2`          | HTML report templating                       |
| `requests`/`bs4`  | Web search fallback for sector P/E           |

### Environment Verification

```python
import yfinance as yf
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
    # CRITICAL: Normalise timezone immediately
    df.index = df.index.tz_localize(None)
    df.index.name = 'Date'
    return df
```

**Checkpoint:** Save as `{ticker}_prices_{years}y.csv`.

### 2b. Stock Info & Fundamentals

```python
def fetch_stock_info(ticker: str) -> dict:
    stock = yf.Ticker(ticker)
    info = stock.info
    keys = ['shortName', 'sector', 'industry', 'marketCap',
            'trailingPE', 'forwardPE', 'trailingEps', 'forwardEps',
            'priceToBook', 'dividendYield', 'beta']
    return {k: info.get(k) for k in keys}
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
        earnings_dates['Earnings Date'] = pd.to_datetime(earnings_dates['Earnings Date']).dt.tz_localize(None)
    return earnings_history, earnings_dates
```

### 2d. Benchmark & Sector P/E

```python
def fetch_benchmark(ticker: str = "SPY", years: int = 2) -> pd.DataFrame:
    return fetch_price_history(ticker, years)

def get_sector_pe_via_web_search(company_sector: str, current_year: int) -> dict:
    """
    Use web search to find current sector P/E.
    Example queries:
    - "S&P 500 technology sector average P/E ratio {current_year}"
    - "{company_sector} industry average P/E ratio"
    Return dict with keys: sector_pe, index_pe, source, as_of.
    """
    # Agent should perform web search and extract numeric values
    pass
```

### 2e. Industry Context (Web Search)

```python
INDUSTRY_QUERIES = [
    "{company} AI market size {year}",
    "{sector} infrastructure spending forecast",
    "{company} market share {technology}",
    "AI adoption enterprise statistics {year}"
]
```

Extract: market size, growth rates, competitive positioning, adoption metrics.

---

## 3. Phase 2: Data Processing & Feature Engineering

### 3a. Timezone Normalisation (Mandatory)

Apply `tz_localize(None)` immediately after every yfinance fetch (see 2a).  
**Failure symptom:** silent empty results on merges / `.loc[]`.

### 3b. Earnings Event Window Alignment

```python
def align_earnings_to_prices(earnings_dates, price_df, window_before=7, window_after=7):
    # Returns DataFrame with columns:
    # earnings_date, actual_trade_date, close_before, close_day_of, close_after,
    # return_1d, return_window, eps_surprise_pct, etc.
    # Uses trading days only (index of price_df).
```

### 3c. Derived Metrics

```python
def compute_derived_metrics(price_df, benchmark_df):
    df = price_df.copy()
    df['daily_return'] = df['Close'].pct_change()
    df['cumulative_return'] = (1 + df['daily_return']).cumprod() - 1
    df['volatility_20d'] = df['daily_return'].rolling(20).std() * np.sqrt(252)
    df['ma_50'] = df['Close'].rolling(50).mean()
    df['ma_200'] = df['Close'].rolling(200).mean()
    if benchmark_df is not None:
        bench_aligned = benchmark_df['Close'].reindex(df.index, method='ffill')
        df['excess_return'] = df['daily_return'] - bench_aligned.pct_change()
    return df
```

### 3d. Data Quality Validation

```python
def validate_data(price_df, earnings_df) -> list:
    warnings = []
    # Check gaps >5 days, zero volume, missing EPS, less than 252 days
    return warnings
```

---

## 4. Phase 3: Multi‑Dimensional Analysis

### 4a. Event Study – Price Action Around Earnings

```python
def earnings_event_study(events_df):
    results = {
        'total_events': len(events_df),
        'mean_1d_return': events_df['return_1d'].mean(),
        'positive_rate': (events_df['return_1d'] > 0).mean(),
        'mean_window_return': events_df['return_window'].mean(),
        'correlation_surprise_return': events_df['eps_surprise_pct'].corr(events_df['return_1d']),
        'correlation_p_value': stats.pearsonr(...)[1],
    }
    return results
```

**Interpretation:** High positive rate (>70%) → market consistently rewards earnings.  
Low correlation → stock moves on narrative/guidance, not just numbers.

### 4b. Earnings‑Timing Strategy Backtest (Fair Comparison)

**Critical correction:** Always compute time‑in‑market, annualised returns, Sharpe, Sortino, Calmar, and max drawdown.

```python
def backtest_earnings_strategy(price_df, earnings_dates, days_before=7, days_after=7, initial_capital=10000):
    # 1. Generate trades (buy N days before, sell N days after)
    # 2. Compute strategy cumulative return, annualised return
    # 3. Compute buy‑and‑hold over the same total calendar period
    # 4. Compute metrics:
    #    - time_in_market = total_holding_days / total_calendar_days
    #    - strategy_annualised = (1+strategy_cumulative)^(1/years) - 1
    #    - sharpe = (mean_trade_return * trades_per_year - rf) / (std_trade_return * sqrt(trades_per_year))
    #    - sortino = (annualised_return - rf) / downside_std
    #    - calmar = annualised_return / abs(max_drawdown)
    # 5. Return dict with all metrics plus final values
```

### 4c. Valuation Benchmarking

```python
def valuation_analysis(stock_info, sector_pe, index_pe):
    trailing_pe = stock_info.get('trailingPE')
    forward_pe = stock_info.get('forwardPE')
    return {
        'trailing_pe': trailing_pe,
        'forward_pe': forward_pe,
        'sector_pe': sector_pe,
        'index_pe': index_pe,
        'trailing_premium_vs_sector': (trailing_pe/sector_pe - 1)*100 if sector_pe else None,
        'forward_discount_vs_sector': (forward_pe/sector_pe - 1)*100 if sector_pe else None,
    }
```

### 4d. Risk‑Adjusted Performance Metrics

```python
def risk_metrics(price_df, benchmark_df=None, risk_free_rate=0.045):
    returns = price_df['Close'].pct_change().dropna()
    total_return = price_df['Close'].iloc[-1]/price_df['Close'].iloc[0] - 1
    years = (price_df.index[-1] - price_df.index[0]).days / 365.25
    annualised_return = (1 + total_return) ** (1/years) - 1
    annualised_vol = returns.std() * np.sqrt(252)
    sharpe = (annualised_return - risk_free_rate) / annualised_vol
    downside_std = returns[returns < 0].std() * np.sqrt(252)
    sortino = (annualised_return - risk_free_rate) / downside_std if downside_std > 0 else 0
    cumulative = (1 + returns).cumprod()
    drawdown = (cumulative / cumulative.cummax()) - 1
    max_drawdown = drawdown.min()
    calmar = annualised_return / abs(max_drawdown) if max_drawdown != 0 else float('inf')
    # Beta & information ratio if benchmark provided
    return {...}
```

### 4e. Fundamental Assessment Synthesis

Combine event study, backtest, valuation, risk, and industry context to answer: **Hype or sustainable?**  
Evaluate earnings quality, price action integrity, valuation reasonableness, risk profile, industry fundamentals.

---

## 5. Phase 4: Visualisation

**8‑panel figure** (2 rows × 4 cols) – dark theme, annotated, consistent date formatting.

| Panel | Content |
|-------|---------|
| 1 | Price + 50/200 MA + earnings markers (green/red vertical lines) |
| 2 | Cumulative returns (stock vs benchmark) |
| 3 | Earnings‑day returns (bar chart per event) |
| 4 | EPS surprise vs 1‑day return scatter + regression line |
| 5 | Valuation comparison (trailing/forward P/E vs sector/index) |
| 6 | Strategy backtest (cumulative returns: strategy vs buy‑hold) |
| 7 | Volume around earnings (pre‑ vs post‑event) |
| 8 | Key metrics summary (text panel) |

Use `matplotlib` with `seaborn` styling. Save as `{ticker}_analysis.png`.

---

## 6. Phase 5: Report Generation

**HTML report** with editorial / data‑journalism aesthetic:

- Dark charcoal background, warm sand text, green accent.
- Single‑column editorial flow (not card grid).
- Embedded CSS, no external dependencies except fonts.
- Sections:
  1. Header: ticker, date, one‑line thesis
  2. Executive summary (highlight strip)
  3. Analysis chart (8‑panel image)
  4. Narrative: price performance, strategy backtest, valuation, AI market context, risks
  5. Investment recommendations by investor profile (table)
  6. Methodology & data sources
  7. Disclaimer

**Recommendation framework:**

| Profile          | Horizon | Recommendation | Rationale |
|------------------|---------|----------------|------------|
| Long‑term        | 3+ yrs  | BUY / HOLD     | Infrastructure early innings |
| Growth           | 1‑3 yrs | ACCUMULATE     | Strong fundamentals, premium |
| Short‑term trader| <1 yr   | AVOID TIMING   | Earnings strategy underperforms |
| Income           | Any     | NOT SUITABLE   | High volatility, no dividend |

---

## 7. Phase 6: Validation & QA

### Pre‑Delivery Checks

```python
def run_qa_checks(price_df, events_df, strategy, valuation, report_html, chart_path):
    issues = []
    if len(price_df) < 252: issues.append("Less than 1 year of data")
    if len(events_df) < 4: issues.append("Fewer than 4 earnings events")
    if strategy['strategy_cumulative_return'] > 10: issues.append("Strategy return > 1000%")
    if valuation.get('trailing_pe', 0) > 500: issues.append("P/E > 500x")
    if not os.path.exists(chart_path): issues.append("Chart missing")
    if '<html' not in report_html.lower(): issues.append("Malformed HTML")
    return issues
```

### Statistical Validity

```python
def check_statistical_validity(events_df):
    n = len(events_df)
    return {
        'sample_adequate': n >= 8,
        'correlation_reliable': n >= 10,
        'confidence_note': f"Based on {n} events. {'Adequate' if n>=8 else 'Caution: small sample'}"
    }
```

---

## 8. Edge Cases, Pitfalls & Augmented Insights

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| **Timezone mismatch** | Empty merge results | `tz_localize(None)` after every yfinance fetch |
| **Future earnings dates** | NaN in analysis | `dropna(subset=['Reported EPS'])` |
| **Unfair backtest comparison** | Misleading underperformance | Compute Sharpe, annualised, time‑in‑market |
| **Missing risk metrics** | Incomplete thesis | Always include Sharpe, Sortino, Calmar, max DD |
| **No significance testing** | False confidence in correlations | Report p‑value alongside each correlation |
| **Look‑ahead bias** | Using today’s P/E for historical decisions | Use point‑in‑time data only |
| **Forward P/E trap** | Assuming growth will materialise | Compare historical estimate accuracy |
| **Macro regime blindness** | Conclusions valid only in one market regime | Note rate environment, sector rotation |

**Additional augmented insights:**
- Always check **revenue surprise** alongside EPS (revenue is harder to manage).
- Consider **short interest** and **options‑implied moves** to understand pre‑earnings pricing.
- For AI stocks, include **infrastructure spending forecasts** and **adoption metrics** from IDC/Gartner.

---

## 9. Reusable Code Templates

### Full Pipeline Orchestration

```python
def run_full_analysis(ticker, years=2, sector_pe=None, index_pe=22.5, output_dir="output"):
    os.makedirs(output_dir, exist_ok=True)
    price_df = fetch_price_history(ticker, years)
    stock_info = fetch_stock_info(ticker)
    earnings_hist, earnings_dates = fetch_earnings_data(ticker)
    benchmark_df = fetch_benchmark("SPY", years)
    price_df = compute_derived_metrics(price_df, benchmark_df)
    events_df = align_earnings_to_prices(earnings_dates, price_df)
    event_study = earnings_event_study(events_df)
    strategy = backtest_earnings_strategy(price_df, earnings_dates)
    if sector_pe is None:
        sector_pe = get_sector_pe_via_web_search(stock_info.get('sector'), datetime.now().year)['sector_pe']
    valuation = valuation_analysis(stock_info, sector_pe, index_pe)
    risk = risk_metrics(price_df, benchmark_df)
    chart_path = create_analysis_charts(price_df, events_df, strategy, valuation, risk, ticker)
    # report_html = generate_html_report(...)
    return {'price_df': price_df, 'events_df': events_df, 'strategy': strategy, 'valuation': valuation, 'risk': risk}
```

### Quick Single‑Ticker Check

```python
def quick_valuation_check(ticker):
    info = fetch_stock_info(ticker)
    print(f"{ticker}: Trailing P/E = {info.get('trailingPE'):.1f}x, Forward P/E = {info.get('forwardPE'):.1f}x")
```

---

## 10. Quality Checklist

Before delivering any analysis:

- [ ] Timezone normalised for all yfinance data
- [ ] No future earnings dates included
- [ ] At least 8 earnings events (4+ for annual)
- [ ] Backtest includes time‑in‑market, annualised returns, Sharpe, Sortino, Calmar, max drawdown
- [ ] Correlation p‑values reported
- [ ] Valuation sourced with date and reference
- [ ] Macro context noted (rate environment, sector trends)
- [ ] Revenue surprise examined (if available)
- [ ] Visualisations all 8 panels rendered, readable
- [ ] HTML report self‑contained, no broken images
- [ ] Recommendations differentiated by investor profile
- [ ] Methodology and limitations documented
- [ ] Disclaimer present

---

## Appendix: Agent Web Search Guidance

To obtain sector P/E and industry context, use the following search patterns:

```python
SEARCH_QUERIES = {
    "sector_pe": "S&P 500 {sector} sector trailing P/E ratio {year}",
    "ai_market_size": "global AI market size {year} IDC Gartner",
    "adoption_rate": "enterprise AI adoption rate {year} survey"
}
```

Extract numeric values and always note the source and retrieval date. When numbers conflict, prefer the most recent official source (e.g., S&P Global, IDC).

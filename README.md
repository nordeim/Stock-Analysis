<div align="center">

# Earnings-Event Stock Analysis

**Interactive Infographic Pipeline for Quantitative Finance & Data Journalism**

[![Domain: Quant Finance](https://img.shields.io/badge/Domain-Quantitative%20Finance%20%2F%20Data%20Journalism-00d4aa?style=flat-square)](https://github.com)
[![Skill Version](https://img.shields.io/badge/Skill-1.0-ffa502?style=flat-square)](https://github.com)
[![Python 3.12+](https://img.shields.io/badge/Python-3.12+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Plotly.js](https://img.shields.io/badge/Charts-Plotly.js%20v2.27-3f4f75?style=flat-square)](https://plotly.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-8a8a9a?style=flat-square)](LICENSE)

_An AI-driven pipeline that transforms raw market data into publication-grade, self-contained interactive HTML infographics — answering the question: **"Is this stock's performance driven by sustainable fundamentals or short-term hype?"**_

[Overview](#-overview) · [Live Artifact](#-live-artifact-nvidia-analysis) · [Pipeline](#-pipeline-architecture) · [Design System](#-design-system) · [Skill Document](#-skill-document) · [Pitfalls & Lessons](#-pitfalls--lessons-learned) · [Quick Start](#-quick-start)

</div>

---

## Overview

This project is an **end-to-end, AI-orchestrated pipeline** that performs deep-dive earnings-event stock analysis and produces a stunning, self-contained HTML infographic with 9 interactive charts. It combines quantitative finance, data journalism, and avant-garde UI/UX design into a single reproducible workflow.

### What It Does

1. **Fetches real market data** via yfinance — OHLCV prices, earnings history, fundamentals
2. **Aligns earnings events** with trading calendars and computes event-window returns
3. **Backtests earnings-timing strategies** with fair time-in-market adjustments and risk metrics (Sharpe, Sortino, Calmar)
4. **Dynamically sources sector benchmarks** via web research — no stale hardcoded P/E ratios
5. **Generates a self-contained HTML infographic** with 9 interactive Plotly.js charts, editorial narrative, and investment recommendations

### Anti-Generic Philosophy

This pipeline explicitly rejects generic approaches:

| Rejected | Embraced |
|----------|----------|
| Static Matplotlib PNGs | Interactive Plotly.js charts with hover/zoom |
| Bootstrap card grids | Bespoke editorial layout with 1px-gap metric grids |
| Inter/Roboto system fonts | DM Serif Display + JetBrains Mono + Inter hierarchy |
| White/light backgrounds | Dark charcoal (#0a0a0f) with ambient gradients |
| Hardcoded sector P/E ratios | Dynamic web-researched benchmarks with citations |
| Cumulative-only backtests | Time-in-market adjusted, annualized, risk-adjusted returns |

---

## Live Artifact: NVIDIA Analysis

The pipeline's first production output is a comprehensive NVIDIA (NVDA) analysis covering 2 years of data with 8 quarterly earnings events.

### Key Findings

| Metric | Value | Insight |
|--------|-------|---------|
| **Buy-and-Hold Return** | +76.4% | Strong absolute performance over 2 years |
| **Earnings-Timing Strategy** | -42.4% | Attempting to trade around earnings destroyed value |
| **Time-in-Market** | 12.1% | Strategy was invested only ~1 month per year |
| **Earnings Beat Rate** | 75% | 6 of 8 quarters beat EPS estimates |
| **Positive Earnings-Day Rate** | 37.5% | Classic "sell the news" dynamic despite beats |
| **Sector P/E (Semiconductors)** | 100.2x | NVDA trades at 69% discount to sector |
| **Beta** | 2.20 | High systematic risk — amplifies market moves |
| **Sharpe Ratio (B&H)** | 0.87 | Decent risk-adjusted performance |

### 9 Interactive Charts

The infographic contains 9 client-side rendered Plotly.js charts:

| # | Chart | Purpose |
|---|-------|---------|
| 1 | **Price + MAs + Earnings** | 2-year price trajectory with 50/200-day moving averages and earnings event markers |
| 2 | **Cumulative Returns** | Stock vs SPY benchmark with alpha visualization |
| 3 | **Earnings-Day Returns** | Bar chart of 1-day, 3-day, and 7-day returns around each earnings event |
| 4 | **EPS Surprise Scatter** | EPS surprise % vs next-day return with regression line and p-value |
| 5 | **Strategy Backtest** | Earnings-timing strategy vs buy-and-hold cumulative returns |
| 6 | **Valuation Comparison** | NVDA P/E vs sector vs S&P 500 with premium/discount indicators |
| 7 | **Rolling Volatility** | 20-day annualized volatility with earnings event markers |
| 8 | **Volume Comparison** | Earnings-day volume vs 20-day average volume |
| 9 | **Revenue Trajectory** | NVIDIA's revenue growth from $27B (FY2024) to $148B (FY2026) |

---

## Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    EARNINGS-EVENT STOCK ANALYSIS PIPELINE                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Phase 1: DATA ACQUISITION                                                 │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │
│  │  yfinance     │ │  yfinance     │ │  yfinance     │ │  yfinance     │      │
│  │  OHLCV Data   │ │  Stock Info   │ │  Earnings     │ │  Benchmark    │      │
│  │  (2yr daily)  │ │  (PE, beta)   │ │  (EPS, dates) │ │  (SPY)        │      │
│  └──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘      │
│         │                │                │                │               │
│         ▼                ▼                ▼                ▼               │
│  Phase 2: PROCESSING & FEATURE ENGINEERING                                 │
│  ┌─────────────────────────────────────────────────────────────┐           │
│  │  tz_localize(None)  →  Derived metrics  →  Earnings alignment │        │
│  │  (CRITICAL step)       (MAs, returns)     (event windows)     │        │
│  └─────────────────────────────┬───────────────────────────────┘           │
│                                │                                           │
│                                ▼                                           │
│  Phase 3: MULTI-DIMENSIONAL ANALYSIS         ┌─────────────────────┐      │
│  ┌────────────────┐ ┌───────────────────┐    │  Phase 4: WEB RESEARCH   │
│  │  Event Study    │ │  Strategy Backtest │    │  ┌───────────────────┐  │
│  │  (CAR, t-test)  │ │  (Sharpe, Sortino) │    │  │  Sector P/E       │  │
│  └───────┬────────┘ └────────┬──────────┘    │  │  (Damodaran/NYU)  │  │
│          │                   │               │  │  Industry Context │  │
│          │    ┌──────────────┘               │  │  Market Size Data │  │
│          │    │                              │  └─────────┬─────────┘  │
│          ▼    ▼                              └────────────┼────────────┘
│  ┌──────────────────┐                                     │               │
│  │  Risk Metrics     │◄────────────────────────────────────┘               │
│  │  (Beta, Alpha,    │                                                     │
│  │   Max Drawdown)   │                                                     │
│  └────────┬─────────┘                                                      │
│           │                                                                │
│           ▼                                                                │
│  Phase 5: HTML INFOGRAPHIC GENERATION                                      │
│  ┌─────────────────────────────────────────────────────────────┐           │
│  │  JSON Serialization  →  Plotly.js Client-Side Rendering       │        │
│  │  Design System       →  Scroll-Reveal Animations              │        │
│  │  Narrative Sections  →  Insight Boxes (green/red/amber)       │        │
│  └─────────────────────────────┬───────────────────────────────┘           │
│                                │                                           │
│                                ▼                                           │
│  Phase 6: QA VALIDATION                                                    │
│  ┌─────────────────────────────────────────────────────────────┐           │
│  │  Chart presence  ·  Design compliance  ·  Statistical rigor   │        │
│  │  Data integrity  ·  Backtest fairness  ·  JSON validity       │        │
│  └─────────────────────────────────────────────────────────────┘           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Why Two-Phase Generation?

The pipeline deliberately splits computation from rendering:

1. **Python phase** — Fetch data, run analysis, serialize to JSON (~179KB)
2. **JavaScript phase** — Client-side Plotly.js renders charts from embedded JSON

This architecture avoids `fig.to_html()` bloat (500KB+ per chart) and produces a single 81KB self-contained HTML file with all 9 charts.

---

## Design System

The infographic follows an intentional, bespoke design system — no off-the-shelf CSS frameworks.

### Color Palette

```
┌────────────────────────────────────────────────────────────┐
│  BACKGROUND                                                │
│  ██████  #0a0a0f  Primary     (deep charcoal)             │
│  ██████  #12121e  Secondary   (elevated surfaces)          │
│  ██████  #1a1a2e  Tertiary    (hover states)               │
│                                                            │
│  TEXT                                                      │
│  ██████  #f0ece4  Primary     (warm sand — NOT white)      │
│  ██████  #8a8a9a  Secondary   (body text)                  │
│  ██████  #5a5a6a  Muted       (labels, metadata)           │
│                                                            │
│  ACCENTS                                                   │
│  ██████  #00d4aa  Positive    (terminal green — gains)     │
│  ██████  #ff4757  Negative    (desaturated crimson)        │
│  ██████  #ffa502  Warning     (muted amber — caution)      │
│  ██████  #3498db  Secondary   (blue — data lines)          │
│  ██████  #a855f7  Tertiary    (purple — sparse accents)    │
└────────────────────────────────────────────────────────────┘
```

### Typography Stack

| Role | Font | Fallback | Usage |
|------|------|----------|-------|
| **Headings** | DM Serif Display | Georgia, serif | Section titles, hero headline |
| **Data/Metrics** | JetBrains Mono | Courier New, monospace | Numbers, percentages, tickers |
| **Body** | Inter | -apple-system, sans-serif | Narrative text, descriptions |

### Key Components

| Component | Pattern | Purpose |
|-----------|---------|---------|
| **Section Header** | Numbered with decorative line | Visual hierarchy and scannability |
| **Insight Box** | 3 variants: green / red / amber | Semantic meaning differentiation |
| **Metric Strip** | 1px-gap grid (shared borders) | Dense data without box shadows |
| **Context Card** | Big number + label + detail | Industry-scale data points |
| **Recommendation Card** | buy/hold/avoid badges | Investor-profile-specific guidance |
| **Verdict Banner** | Full-width with accent border | Executive summary at top |

### Scroll-Reveal Animations

All content sections animate in via `IntersectionObserver`:

```css
.reveal {
    opacity: 0;
    transform: translateY(24px);
    transition: opacity 0.7s ease-out, transform 0.7s ease-out;
}
.reveal.active {
    opacity: 1;
    transform: translateY(0);
}
```

---

## Skill Document

The comprehensive skill document ([`SKILL_Infographic_Stock_Analysis.md`](SKILL_Infographic_Stock_Analysis.md)) is the core artifact — a 1,360-line, 55KB reference that distills the exact workflow, methodology, and lessons from this project.

### What's Inside

| Section | Lines | Content |
|---------|-------|---------|
| Overview & Philosophy | 1-82 | Anti-generic mandate, technical rigor principles |
| Source Skills & Dependencies | 83-123 | stock-analysis (primary), charts + web-search (augmentation) |
| Environment Setup & Pitfalls | 124-183 | Python venv, yfinance timeouts, rate limiting |
| Phase 1: Data Acquisition | 184-296 | OHLCV, stock info, earnings, benchmarks — with tz_localize |
| Phase 2: Processing | 297-420 | Feature engineering, earnings alignment |
| Phase 3: Analysis | 421-600 | Event study, backtest, risk metrics |
| Phase 4: Web Research | 601-700 | Dynamic sector P/E, industry context |
| Phase 5: HTML Generation | 701-900 | JSON serialization, Plotly.js client-side, design system |
| Phase 6: QA Validation | 901-922 | Automated checks for charts, design, statistics |
| Pitfalls & Lessons | 924-977 | 27 pitfalls across 5 categories with symptoms and resolutions |
| Design System Reference | 980-1085 | CSS custom properties, responsive breakpoints, components |
| Chart Architecture | 1088-1151 | Data serialization patterns, Plotly.js rendering |
| Reusable Templates | 1155-1243 | Full pipeline orchestration, JSON helper |
| Quality Checklist | 1247-1316 | 50+ verification items across 7 categories |
| Appendices | 1319-1360 | File structure, execution timeline, extending to other tickers |

### Source Skills Synthesized

This skill document synthesizes knowledge from three source skills:

```
SKILL_Infographic_Stock_Analysis.md (1,360 lines)
├── stock-analysis_SKILL.md (2,188 lines)     ← Primary: data pipeline, analysis, QA
├── charts skill                              ← Augmentation: color systems, anti-patterns
└── web-search skill                          ← Augmentation: dynamic data sourcing
```

---

## Pitfalls & Lessons Learned

The pipeline encountered **27 pitfalls** across 5 categories. Here are the most critical:

### The #1 Silent Killer: Timezone Mismatch

yfinance returns timezone-aware DataFrames (`America/New_York`). Merging with timezone-naive data produces **0 rows with no error**.

```python
# WRONG — silently produces empty results
price_df = yf.Ticker("NVDA").history(...)
earnings['date'] = pd.to_datetime(earnings['date'])
merged = price_df.loc[earnings['date']]  # → 0 rows, no error!

# CORRECT — normalize immediately after every yfinance fetch
df = stock.history(start=start, end=end, auto_adjust=True)
if df.index.tz is not None:
    df.index = df.index.tz_localize(None)  # Strip timezone
```

### The #2 Value Destroyer: Unfair Backtest Comparison

An earnings-timing strategy is only invested ~12% of the time. Comparing its cumulative return directly against buy-and-hold is misleading.

```python
# WRONG — apples to oranges comparison
strategy_return = -42.4%
buy_hold_return = +76.4%
# Conclusion: "Strategy is terrible" — MISLEADING

# CORRECT — fair comparison with time-in-market adjustment
strategy_annualized = -87.8%  # annualized
buy_hold_annualized = +33.3%  # annualized
strategy_sharpe = -1.42       # risk-adjusted
buy_hold_sharpe = 0.87        # risk-adjusted
# Conclusion: "Strategy destroys value even on risk-adjusted basis" — ACCURATE
```

### The #3 Stale Data Trap: Hardcoded Sector P/E

The original skill document had `sector_pe = 39.1`. The actual semiconductor trailing P/E is **100.2x** (Damodaran/NYU Stern, Jan 2026). This completely changes the valuation narrative:

```
Hardcoded (39.1x):  NVDA at 39% PREMIUM to sector  → "Overvalued"
Dynamic  (100.2x):  NVDA at 69% DISCOUNT to sector → "Undervalued relative to peers"
```

### The #4 Environment Gotcha: Python Interpreter Mismatch

```
System Python:  /usr/bin/python3 → Python 3.13 (system-managed, read-only)
Venv Python:    /home/z/.venv/bin/python3 → Python 3.12 (user-managed)

# pip install --break-system-packages installs to Python 3.13
# but python3 runs from venv (Python 3.12) → ModuleNotFoundError!
```

### The #5 HTML Bloat: fig.to_html()

```python
# WRONG — 500KB+ per chart with fig.to_html()
html = fig.to_html(include_plotlyjs=True)  # Embeds entire Plotly.js library!

# CORRECT — embed raw JSON, render client-side with shared CDN
# Python: serialize chart data to JSON arrays
price_dates = json.dumps([p['date'] for p in data['price_data']])
price_close = json.dumps([round(p['close'], 2) for p in data['price_data']])

# HTML: single CDN link + JavaScript chart construction
# <script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>
# <script>
#   Plotly.newPlot('chart-price', [{ x: priceDates, y: priceClose, ... }], layout);
# </script>
```

### Full Pitfall Summary

| Category | Count | Top Issue |
|----------|-------|-----------|
| Environment | 4 | Python interpreter mismatch |
| Data Pipeline | 6 | Timezone mismatch (silent failures) |
| Analysis | 6 | Hardcoded sector P/E |
| Design | 7 | fig.to_html() bloat |
| Workflow | 4 | Full pipeline timeout |

---

## Repository Structure

```
stock-earnings-analysis/
│
├── README.md                                   ← You are here
├── SKILL_Infographic_Stock_Analysis.md          ← Comprehensive skill document (1,360 lines)
│
├── NVDA_Earnings_Valuation_Analysis.html        ← Self-contained interactive infographic (81KB)
├── nvda_analysis_data.json                      ← Serialized analysis data (499 price points, 8 events)
│
├── skills_archive.tar.gz                        ← Source skills archive (39MB)
│                                                  Includes: stock-analysis, charts, web-search, etc.
│
└── upload/
    └── stock-analysis_SKILL.md                  ← Original source skill (2,188 lines)
```

### File Details

| File | Size | Description |
|------|------|-------------|
| `NVDA_Earnings_Valuation_Analysis.html` | 81 KB | Self-contained HTML infographic with 9 Plotly.js charts, editorial narrative, and investment recommendations |
| `nvda_analysis_data.json` | 179 KB | JSON-serialized analysis data: 499 OHLCV data points, 8 earnings events, event study results, backtest results, valuation metrics, risk metrics |
| `SKILL_Infographic_Stock_Analysis.md` | 55 KB | Comprehensive skill document with 14 sections + 3 appendices, 27 pitfalls, design system reference, QA checklist |
| `skills_archive.tar.gz` | 39 MB | Complete source skills folder for skill supplementation |
| `stock-analysis_SKILL.md` | ~60 KB | Original source skill — the foundation pipeline |

---

## Quick Start

### Prerequisites

```bash
# Use the correct Python environment (venv, not system Python)
python3 -c "import sys; print(sys.executable)"
# Should output: /home/z/.venv/bin/python3

# Install dependencies (use venv pip with extended timeout for yfinance)
/home/z/.venv/bin/pip3 install yfinance pandas numpy scipy plotly requests beautifulsoup4 lxml
```

### Run the Pipeline

```python
from datetime import datetime, timedelta
import yfinance as yf
import pandas as pd
import numpy as np
import json

TICKER = "NVDA"
YEARS = 2

# Phase 1: Fetch data
stock = yf.Ticker(TICKER)
end = datetime.now()
start = end - timedelta(days=YEARS * 365)

price_df = stock.history(start=start, end=end, auto_adjust=True)
if price_df.index.tz is not None:
    price_df.index = price_df.index.tz_localize(None)  # CRITICAL

stock_info = stock.info
earnings = stock.earnings_dates
benchmark = yf.Ticker("SPY").history(start=start, end=end, auto_adjust=True)
if benchmark.index.tz is not None:
    benchmark.index = benchmark.index.tz_localize(None)  # CRITICAL

# Phase 2-5: See SKILL_Infographic_Stock_Analysis.md for complete pipeline
```

### View the Infographic

Simply open `NVDA_Earnings_Valuation_Analysis.html` in any modern browser. No server required — it's fully self-contained with embedded data and CDN-loaded Plotly.js.

---

## Extending to Other Tickers

The pipeline works with any publicly traded equity. To analyze a different stock:

1. Change `TICKER = "NVDA"` to your desired ticker (e.g., `"AAPL"`, `"MSFT"`, `"GOOGL"`)
2. Web-search for that company's sector P/E ratio (Damodaran/NYU Stern is authoritative)
3. Adjust the industry context section (AI narrative → healthcare/financials/energy)
4. Research company-specific revenue data for the trajectory chart
5. Update recommendation cards based on company-specific fundamentals

**Sector-specific adjustments:**

| Sector | P/E Source | Context Theme |
|--------|-----------|---------------|
| Technology | Semiconductor / Software P/E | AI, cloud, digital transformation |
| Healthcare | Pharma / Biotech P/E | Drug pipeline, FDA, demographics |
| Financials | Banks / Insurance P/E | Rate sensitivity, loan growth |
| Energy | Oil & Gas P/E | Commodity prices, transition |
| Consumer | Discretionary P/E | Spending trends, e-commerce |

---

## Technical Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Data** | yfinance API | Historical OHLCV, earnings, fundamentals |
| **Processing** | pandas, numpy | Time series alignment, feature engineering |
| **Statistics** | scipy | Pearson correlation, linear regression, t-tests |
| **Web Research** | web-search skill + direct source reading | Dynamic sector benchmarks, industry context |
| **Visualization** | Plotly.js v2.27 (CDN) | Interactive charts, client-side rendering |
| **Typography** | Google Fonts CDN | DM Serif Display, JetBrains Mono, Inter |
| **Styling** | Custom CSS | Bespoke design system, no frameworks |
| **Animation** | IntersectionObserver | Scroll-reveal with 24px translateY |
| **Serialization** | JSON (Python → JavaScript) | Data bridge between analysis and rendering |

---

## Key Design Decisions

### Client-Side Chart Rendering over Server-Side

Embedding raw JSON data and constructing charts in JavaScript via shared Plotly.js CDN produces an 81KB file vs. 500KB+ per chart with `fig.to_html()`. This 6-10x size reduction makes the infographic practical for sharing via email, chat, or static hosting.

### Warm Sand over Pure White

Pure white text (#FFFFFF) on dark backgrounds creates harsh contrast and eye strain. The warm sand (#f0ece4) is intentional — it provides excellent readability while being comfortable for extended viewing of dense analytical content.

### 1px-Gap Grid over Box Shadows

Traditional card-based layouts with box shadows look generic and template-like. The 1px-gap grid creates shared borders between cells, producing a dense, data-journalism aesthetic that feels editorial rather than dashboard-like.

### Three Insight-Box Variants

A single insight-box style makes all insights look the same regardless of sentiment. Three variants (green for positive findings, red for warnings, amber for neutral notes) provide immediate visual differentiation and semantic meaning.

### Dynamic Data Sourcing over Hardcoding

The original skill had `sector_pe = 39.1` which was dangerously wrong (actual: 100.2x). Dynamic web research with source citations prevents stale benchmarks that could completely reverse analytical conclusions.

---

## QA Checklist Highlights

The skill document includes a 50+ item QA checklist. The most critical checks:

- **Timezone normalized** for all yfinance data (`tz_localize(None)`)
- **No future earnings dates** included (`dropna(subset=['Reported EPS'])`)
- **Every correlation** has p-value reported
- **Time-in-market** calculated and reported for backtests
- **Sector P/E sourced dynamically** with citation
- **All 9 charts** render with transparent backgrounds
- **HTML self-contained** — no broken images or external dependencies
- **Disclaimer present** — not financial advice

---

## License

This project is provided for educational and research purposes. The analysis and visualizations are **not financial advice**. Always conduct your own research and consult qualified professionals before making investment decisions.

---

<div align="center">

_Built with quantitative rigor, editorial ambition, and an aversion to the generic._

**Skill v1.0** · June 2026 · Quantitative Finance + Data Journalism + Avant-Garde UI/UX

</div>

---
title: Factor Library — All Factors
nav_order: 20
permalink: /factors/revenue/
---

# Factor Library

This page collects all factor examples learned so far.  
Each section includes:
- **Code snippet** (ready to run on JoinQuant)
- **Comments** (explaining the code)
- **Finance intuition** (why this factor matters)

---

## 1. Revenue Factors

### (1) Year-on-Year Revenue Growth
```python
# Select stocks with revenue growth > 300% YoY
df = get_fundamentals(
    query( 
        indicator.code,
        indicator.inc_revenue_year_on_year
    ).filter(
        indicator.inc_revenue_year_on_year > 300
    ).order_by(
        indicator.inc_revenue_year_on_year.desc()
    ),
    date='2022-09-01'
)
print(df)
```
**Finance Notes**  
- Measures revenue growth compared with the same quarter last year.  
- Captures high-growth companies.  
- Caveat: sometimes driven by one-off events or low base effects.

---

### (2) Quarterly Revenue Acceleration
```python
# Select stocks with quarterly annualized revenue growth > 900%
df = get_fundamentals(
    query(
        indicator.code,
        indicator.inc_revenue_annual
    ).filter(
        indicator.inc_revenue_annual > 900
    ).order_by(
        indicator.inc_revenue_annual.desc()
    ),
    date='2019-06-01'
)
print(df)
```
**Finance Notes**  
- Tracks sequential sales momentum (quarter-on-quarter, annualized).  
- Useful for detecting turning points earlier than YoY measures.

---

### (3) Net Profit Margin (Net Profit / Revenue)
```python
# Rank stocks by Net Profit Margin
df = get_fundamentals(
    query(
        indicator.code,
        indicator.net_profit_to_total_revenue
    ).order_by(
        indicator.net_profit_to_total_revenue.desc()
    ),
    date='2022-09-01'
)
print(df)
```
**Finance Notes**  
- Profitability per unit of sales.  
- Higher margins suggest stronger pricing power or cost control.

---

## 2. Profitability Factors

### (1) Net Profit Growth YoY
```python
# Select stocks with net profit YoY growth > 300%
df = get_fundamentals(
    query(
        indicator.code,
        indicator.inc_net_profit_year_on_year
    ).filter(
        indicator.inc_net_profit_year_on_year > 300
    ).order_by(
        indicator.inc_net_profit_year_on_year.desc()
    ),
    date='2022-09-01'
)
print(df)
```
**Finance Notes**  
- Focuses on bottom-line growth.  
- Strong net profit growth often drives re-rating, but verify quality (non-recurring items).

---

### (2) Operating Profit Margin
```python
# Select stocks with operating profit margin > 200%
df = get_fundamentals(
    query(
        indicator.code,
        indicator.operation_profit_to_total_revenue
    ).filter(
        indicator.operation_profit_to_total_revenue > 200
    ).order_by(
        indicator.operation_profit_to_total_revenue.desc()
    ),
    date='2022-09-01'
)
print(df[:5])
```
**Finance Notes**  
- Efficiency of core operations before non-operating items.  
- High OPM can reflect scale, mix, or cost discipline.

---

### (3) Net Profit Margin (Top 5)
```python
# Top 5 companies by net profit margin
df = get_fundamentals(
    query(
        indicator.code,
        indicator.net_profit_margin
    ).order_by(
        indicator.net_profit_margin.desc()
    ),
    date='2022-09-01'
)
print(df[:5])
```
**Finance Notes**  
- Bottom-line profitability after all expenses.  
- Sustained high NPM is rare; check stability across cycles.

---

### (4) Gross Profit Margin (Top 5)
```python
# Top 5 companies by gross profit margin
df = get_fundamentals(
    query(
        indicator.code,
        indicator.gross_profit_margin
    ).order_by(
        indicator.gross_profit_margin.desc()
    ),
    date='2022-09-01'
)
print(df[:5])
```
**Finance Notes**  
- (Revenue − COGS) / Revenue.  
- Strong unit economics; often seen in brands, software, or high-value niches.

---

## 3. Size Factors

### (1) Market Capitalization
```python
# Top 5 by Market Cap > 100 billion
df = get_fundamentals(
    query(
        valuation.code,
        valuation.market_cap
    ).filter(
        valuation.market_cap > 10000
    ).order_by(
        valuation.market_cap.desc()
    )
)
print(df[:5])
```
**Finance Notes**  
- Price × shares outstanding.  
- Size proxies risk/liquidity; large-caps differ from small-caps.

---

### (2) Circulating Market Cap (Free-Float Market Cap)
```python
# Top 5 by circulating market cap > 500 billion
df = get_fundamentals(
    query(
        valuation.code,
        valuation.circulating_market_cap
    ).filter(
        valuation.circulating_market_cap > 5000
    ).order_by(
        valuation.circulating_market_cap.desc()
    )
)
print(df[:5])
```
**Finance Notes**  
- Tradable portion of market cap.  
- Important in markets with restricted shares (affects liquidity and index inclusion).

---

### (3) Total Shares Outstanding + Market Cap
```python
# Large share base and large market cap
df = get_fundamentals(
    query(
        valuation.code,
        valuation.market_cap,
        valuation.capitalization
    ).filter(
        valuation.market_cap > 8000,
        valuation.capitalization > 10000000
    ).order_by(
        valuation.capitalization.desc()
    )
)
print(df[:5])
```
**Finance Notes**  
- Mature profiles often have both high shares outstanding and high market cap.

---

### (4) Circulating Shares Outstanding + Free-Float Market Cap
```python
# Large free float (circulating shares) and high free-float market cap
df = get_fundamentals(
    query(
        valuation.code,
        valuation.circulating_market_cap,
        valuation.circulating_cap
    ).filter(
        valuation.circulating_market_cap > 5000,
        valuation.circulating_cap > 10000000
    ).order_by(
        valuation.circulating_cap.desc()
    )
)
print(df[:5])
```
**Finance Notes**  
- Larger floats can reduce price impact; small floats are more volatile and squeeze-prone.

---

## 4. Value Factors

### (1) Price-to-Book (PB)
```python
# PB < 1.5 and Market Cap > 80 billion
df = get_fundamentals(
    query(
        valuation.code,
        valuation.pb_ratio,
        valuation.market_cap
    ).filter(
        valuation.market_cap > 8000,
        valuation.pb_ratio < 1.5
    ).order_by(
        valuation.pb_ratio.asc()
    ),
    date='2022-09-01'
)
print(df[:5])
```
**Finance Notes**  
- Classic value screen; low PB can indicate undervaluation vs. net assets.  
- Beware value traps (declining business quality).

---

### (2) Price-to-Sales (PS) + PB
```python
# PS < 0.5 and PB < 1.5
df = get_fundamentals(
    query(
        valuation.code,
        valuation.pb_ratio,
        valuation.ps_ratio
    ).filter(
        valuation.ps_ratio < 0.5,
        valuation.pb_ratio < 1.5
    ).order_by(
        valuation.pb_ratio.asc()
    ),
    date='2019-09-01'
)
print(df[:5])
```
**Finance Notes**  
- Sales-based valuation works when earnings are noisy/negative.  
- Combining PB and PS reduces single-metric bias.

---

### (3) Multi-Value Screen (PS + PCF + PE band)
```python
# PS < 0.5, P/CF < 6, and PE between 3 and 5
df = get_fundamentals(
    query(
        valuation.code,
        valuation.pcf_ratio,
        valuation.pe_ratio,
        valuation.ps_ratio
    ).filter(
        valuation.ps_ratio < 0.5,
        valuation.pcf_ratio < 6,
        valuation.pe_ratio > 3,
        valuation.pe_ratio < 5
    ).order_by(
        valuation.pe_ratio.asc()
    ),
    date='2019-09-01'
)
print(df[:5])
```
**Finance Notes**  
- Cross-checking value metrics improves robustness.  
- Tight PE band focuses on deep-value candidates.

---

## 5. Quality Factors

### (1) Return on Equity (ROE)
```python
# ROE > 50 (example threshold)
df = get_fundamentals(
    query(
        indicator.code,
        indicator.roe
    ).filter(
        indicator.roe > 50
    ).order_by(
        indicator.roe.asc()
    ),
    date='2019-03-01'
)
print(df[:10])
```
**Finance Notes**  
- Net Income / Shareholder Equity.  
- Captures capital efficiency; combine with leverage checks.

---

### (2) Return on Assets (ROA)
```python
# ROA > 10 (example threshold)
df = get_fundamentals(
    query(
        indicator.code,
        indicator.roa
    ).filter(
        indicator.roa > 10
    ).order_by(
        indicator.roa.asc()
    ),
    date='2019-03-01'
)
print(df[:10])
```
**Finance Notes**  
- Net Income / Total Assets.  
- Cross-industry efficiency measure; less affected by leverage than ROE.

---

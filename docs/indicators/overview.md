---
title: Indicators (MACD / EMV / UOS)
nav_order: 3
permalink: /indicators/overview/
---

# Indicators (MACD / EMV / UOS)

Quick links: [MACD](#macd) • [EMV](#emv) • [UOS](#uos)

---

## MACD

**Definition**: Trend-following momentum indicator using two EMAs.  
**Formula**:  
- **DIF (MACD line)** = EMA(SHORT) − EMA(LONG)  
- **DEA (Signal line)** = EMA(DIF, MID)  
- **Histogram** = (DIF − DEA) × 2

**How to use**  
- DIF crosses **above** DEA → bullish bias  
- DIF crosses **below** DEA → bearish bias  
- Histogram divergence may warn of momentum shift

**JoinQuant API (single date)**
```python
# Get MACD of Ping An Bank on a specific date
from jqlib.technical_analysis import MACD

security = '000001.XSHE'
DIF, DEA, MACD_hist = MACD(
    security_list=security,
    check_date='2022-09-01',
    SHORT=12, LONG=26, MID=9
)
print(DIF, DEA, MACD_hist)
```

**Notes**  
- Parameter name is `check_date` (not `date`).  
- Ticker suffix: `.XSHE` (SZ), `.XSHG` (SH).

---

## EMV (Ease of Movement)

**Definition**: Volume-based oscillator measuring how “easily” price moves.

**How to use**
- EMV > 0 → upside moves are easier  
- EMV < 0 → downside moves are easier  
- Simple rule: cross **above 0** = buy; cross **below 0** = sell

**JoinQuant API (multiple symbols)**
```python
from jqlib.technical_analysis import EMV

security_list = ['000001.XSHE', '000002.XSHE', '601211.XSHG']
EMV_val, MAEMV_val = EMV(
    security_list=security_list,
    check_date='2022-09-01',
    N=14, M=9
)
print(EMV_val)
print(MAEMV_val)
```

**Notes**  
- `EMV` returns the raw EMV and its moving average (MAEMV).  
- Avoid curly quotes or extra characters in tickers.

---

## UOS (Ultimate Oscillator)

**Definition**: Momentum oscillator combining 3 timeframes to reduce false signals.

**How to use**
- > 70 often considered overbought; < 30 oversold  
- Divergence with price can signal potential reversal  
- Works best with a trend filter (e.g., 200-day SMA)

**JoinQuant API (single symbol)**
```python
from jqlib.technical_analysis import UOS

security = ['000001.XSHE']  # list is accepted
uos_val, mauos_val = UOS(
    security_list=security,
    check_date='2022-09-01',
    N1=7, N2=14, N3=28, M=6
)
print(uos_val)
print(mauos_val)
```

**Notes**  
- Returns UOS and its smoothed value (MAUOS).  
- Use consistent date format `YYYY-MM-DD`.

---

## Quick Takeaways

- **MACD** → trend + momentum shifts  
- **EMV** → price-volume “ease” / zero-line crosses  
- **UOS** → multi-timeframe momentum; fewer whipsaws  
Combine them: **trend (MACD)** + **confirmation (EMV)** + **timing (UOS)**.

---

## Common Pitfalls (fast)

- Using `date=` instead of **`check_date=`**.  
- Ticker typos or curly quotes (e.g., `’` vs `'`).  
- Forgetting to import from `jqlib.technical_analysis`.

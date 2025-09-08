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
- MACD Line = EMA(12) − EMA(26)  
- Signal Line = EMA(9) of MACD Line  
- Histogram = MACD − Signal

**Usage**:  
- MACD crosses above Signal → Bullish bias  
- MACD crosses below Signal → Bearish bias  
- Histogram divergence may hint momentum shift

**Pros**: Captures trend changes.  
**Cons**: Lags; choppy in ranges.

### Code (JoinQuant-style; pandas-based)
```python
# Get OHLCV and compute MACD
def calc_macd(close, fast=12, slow=26, signal=9):
    # EMAs (no de-meaning; standard implementation)
    ema_fast = close.ewm(span=fast, adjust=False).mean()
    ema_slow = close.ewm(span=slow, adjust=False).mean()
    macd = ema_fast - ema_slow
    signal_line = macd.ewm(span=signal, adjust=False).mean()
    hist = macd - signal_line
    return macd, signal_line, hist

# Example inside JoinQuant
def run_example_macd(security='000001.XSHE'):
    df = get_price(security, start_date='2018-01-01', end_date='2019-12-31',
                   frequency='daily', fields=['close'])
    macd, sig, hist = calc_macd(df['close'])
    df['MACD'], df['MACD_signal'], df['MACD_hist'] = macd, sig, hist
    return df.tail(5)

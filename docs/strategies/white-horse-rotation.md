---
title: Strategy Case — White-Horse Rotation
nav_order: 40
permalink: /strategies/white-horse-rotation/
---

# Strategy Case — White‑Horse Rotation

A modular, fundamentals‑driven rotation strategy. Rebalances **every 100 days**, holding **20 large, consistently profitable “white‑horse” stocks** that satisfy **ROE > 20%**, **Gross Profit Margin > 20%**, and **Market Cap > 50 (CNY 100M units)**.

**Covers**: `initialize` • stock selection • rebalance logic • IPO‑age filter • paused filter • `order_value` execution.

---

## Logic Overview

- **Universe**: A‑share stocks available on JoinQuant.  
- **Quality filters** (yearly fundamentals via `get_fundamentals`):
  - `indicator.roe > 20`
  - `indicator.gross_profit_margin > 20`
  - `valuation.market_cap > 50`  *(JoinQuant unit is 亿元 CNY)*
- **Ranking**: Sort by `valuation.market_cap` **desc**, pick top 100, then after filters keep **top 20**.  
- **Operational filters**:  
  - Exclude **IPO age < 750 days**  
  - Exclude **paused** stocks
- **Rebalance**: Every **100 calendar days**, equal‑cash allocation to reach 20 names.

> ⚠️ The sample below triggers on a fixed **daily time** and checks whether 100 **calendar** days have passed since last rebalance. You can switch to **trading‑day counting** (see “Notes & Variants”).

---

## Full JoinQuant Implementation (runnable)

```python
# -*- coding: utf-8 -*-
from jqdata import *                      # JoinQuant API
from datetime import datetime, timedelta

# =======================
# Initialization
# =======================
def initialize(context):
    set_benchmark('000300.XSHG')                     # CSI 300
    set_option('use_real_price', True)
    set_option('order_volume_ratio', 1)

    set_order_cost(OrderCost(
        open_tax=0,
        close_tax=0.001,
        open_commission=0.0003,
        close_commission=0.0003,
        close_today_commission=0,
        min_commission=5
    ), type='stock')

    g.stocknum = 20                   # target number of holdings
    g.refresh_days = 100              # rebalance every 100 days (calendar)
    g.last_rebalance_date = None      # track last rebalance date

    # Run once per day at market open; the function itself guards the 100-day interval
    run_daily(rebalance, time='09:35')


# =======================
# Core Stock Selection
# =======================
def pick_white_horses(context):
    # Build fundamental query (yearly statement by statDate)
    q = query(
        indicator.code,
        valuation.market_cap,             # 亿元
        indicator.roe,
        indicator.gross_profit_margin
    ).filter(
        valuation.market_cap > 50,        # > 50 亿元
        indicator.gross_profit_margin > 20,
        indicator.roe > 20
    ).order_by(
        valuation.market_cap.desc()
    ).limit(100)

    # Pull latest annual data by the current year (JoinQuant uses statDate='YYYY')
    df = get_fundamentals(q, statDate=str(context.current_dt.year))

    if df is None or df.empty:
        return []

    # Initial list
    codes = list(df['code'])

    # Operational filters
    codes = exclude_recently_listed(codes, context.current_dt.date(), min_days=750)
    codes = exclude_paused(codes)

    # Take top 20 after filters
    return codes[:20]


# -----------------------
# Helper: Exclude paused
# -----------------------
def exclude_paused(stock_list):
    current_data = get_current_data()
    return [s for s in stock_list if (s in current_data and not current_data[s].paused)]


# -----------------------
# Helper: Exclude IPOs newer than N days
# -----------------------
def exclude_recently_listed(stocks, ref_date, min_days=180):
    res = []
    for s in stocks:
        info = get_security_info(s)
        if not info:
            continue
        ipo_date = info.start_date
        if ipo_date and ipo_date <= (ref_date - timedelta(days=min_days)):
            res.append(s)
    return res


# =======================
# Rebalance Logic
# =======================
def rebalance(context):
    # 1) Check the 100-day condition (calendar days)
    today = context.current_dt.date()
    if g.last_rebalance_date is not None:
        days_since = (today - g.last_rebalance_date).days
        if days_since < g.refresh_days:
            return  # not time yet

    # 2) Select target list
    target_list = pick_white_horses(context)

    # 3) Determine sells (names not in target list)
    current_positions = list(context.portfolio.positions.keys())
    to_sell = list(set(current_positions) - set(target_list))
    for s in to_sell:
        order_target_value(s, 0)

    # 4) Determine buys (names in target but not held)
    current_positions = list(context.portfolio.positions.keys())
    to_buy = [s for s in target_list if s not in current_positions]

    # 5) Equal-cash allocation for new buys
    slots_left = max(0, g.stocknum - len(current_positions))
    if slots_left > 0:
        cash_per_name = context.portfolio.cash / max(1, slots_left)
    else:
        cash_per_name = 0

    for s in to_buy:
        if len(context.portfolio.positions) >= g.stocknum:
            break
        if cash_per_name > 0:
            order_value(s, cash_per_name)

    # 6) Stamp the last rebalance date
    g.last_rebalance_date = today
```

---

## Notes & Variants

- **Trading‑day interval**: For exact trading‑day spacing, precompute a series with `get_trade_days` and advance an index; or store a counter you increment in a function scheduled by `run_daily` and only count on trading days.
- **Universe hygiene**: In production, also exclude `ST/*ST`, and consider removing limit‑up/limit‑down, suspended, or risk‑warning boards.
- **Capitalization vs. Market Cap**: This implementation filters by `valuation.market_cap` (units: 亿元). Using `valuation.capitalization` will filter by **shares outstanding** instead of market value.
- **Equal weight**: The example uses equal **cash** per new position; you can convert to **equal weight** on total portfolio using `order_target_percent`.
- **Sector neutrality / max exposure**: Add sector caps or max weight per name to reduce concentration risk.

---

## Why this can work (intuition)

- **Profitability screen** (ROE, gross margin) favors stable, entrenched firms.  
- **Size tilt** reduces idiosyncratic risk and improves liquidity.  
- **Slow rotation** (100‑day) avoids over‑trading on noisy accounting updates.

> This is an educational template, not investment advice. Backtest thoroughly (fees, slippage, survivorship, look‑ahead).

---

## Common Pitfalls (quick)

- Using `valuation.capitalization > 50` when you actually want **market cap** filter.  
- Confusing `date` vs. `statDate`: for fundamentals use `statDate='YYYY'`; for indicators use `check_date='YYYY-MM-DD'`.  
- Scheduling `every_bar` and incrementing a “day” counter → causes much more frequent rebalances than intended.

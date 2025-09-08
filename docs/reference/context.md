---
title: System & Core Objects
nav_order: 10
---

# System & Core Objects

This page shows the minimum building blocks you’ll use in every JoinQuant strategy:
- the runtime lifecycle (`initialize`, scheduled functions, handlers)
- the `context` object (time, universe, portfolio)
- orders/trades APIs
- portfolio/positions/subportfolios

---

## 1) Runtime & `context` basics

**What it does:** prints current time, previous trading day, universe, and total account value.

~~~python
def initialize(context):
    # choose a target security for demos
    g.security = "000001.XSHE"

def handle_data(context, data):
    print("now:", context.current_dt)                 # datetime of the current bar/day
    print("previous date:", context.previous_date)    # previous trading date
    print("universe:", context.universe)              # if set via set_universe(...)
    print("total value:", context.portfolio.total_value)  # cash + holdings
~~~

> **Why it matters:** `context` is your global state during backtests/live runs.  
> You read data/state from it and also store your globals (e.g., `g.security`).

---

## 2) Orders & Trades

**Place an order**, then **inspect trades after close**.

~~~python
def initialize(context):
    set_benchmark('000300.XSHG')
    g.security = "000001.XSHE"
    # run once shortly after open; and a post-close hook
    run_daily(opening_trade, time='09:35')
    run_daily(after_market_close, time='15:30')

def opening_trade(context):
    # buy 100 shares if we don't hold it
    if g.security not in context.portfolio.positions:
        order_obj = order(g.security, 100)
        if order_obj:
            print("commission:", order_obj.commission)
            print("is_buy:", order_obj.is_buy)
            print("status:", order_obj.status)  # e.g., 'filled'
            print("avg price:", order_obj.price)
        else:
            print("order failed")

def after_market_close(context):
    # trade fills are available after close
    trades = get_trades()
    for tr in trades.values():
        print("trade:", tr)
        print(" time:", tr.time)
        print(" order_id:", tr.order_id)
        print(" price:", tr.price, " amount:", tr.amount)
~~~

> **Tip:** an **Order** is your request; a **Trade** is what actually got filled.

---

## 3) Portfolio & Positions

**Read positions** and **cash fields** from the account.

~~~python
def initialize(context):
    g.security = "000001.XSHE"

def handle_data(context, data):
    # open a demo position (idempotent checks omitted)
    order(g.security, 1000)

    # iterate long positions
    for pos in list(context.portfolio.long_positions.values()):
        print("holding:", pos.security)
        print(" total_amount:", pos.total_amount)
        print(" market_value:", pos.value)
        print(" init_time:", pos.init_time)

    # sub-portfolio cash fields (index 0 for stock account)
    sp = context.subportfolios[0]
    print("available_cash:", sp.available_cash)
    print("inout_cash:", sp.inout_cash)
    print("transferable_cash:", sp.transferable_cash)
    print("locked_cash:", sp.locked_cash)

    # portfolio-level aggregates
    print("long_positions:", context.portfolio.long_positions)
    print("short_positions:", context.portfolio.short_positions)
    print("total_value:", context.portfolio.total_value)
    print("returns:", context.portfolio.returns)
    print("starting_cash:", context.portfolio.starting_cash)
    print("positions_value:", context.portfolio.positions_value)
~~~

> **Why it matters:** every allocation/resize decision depends on these fields
> (how much cash left, how much value in positions, realized/unrealized P&L).

---

## 4) Common pitfalls (quick checklist)

- **Don’t reuse multiple `initialize` in one file** unless they belong to *separate* examples.  
- **Orders vs trades:** fills are visible *after close* via `get_trades()`.  
- **Suspensions:** filter paused stocks with `get_current_data()[s].paused`.  
- **Commas & quotes:** many Python errors are from missing commas or smart quotes.

---

## Copy-paste snippets (quick reference)

### Print core timestamps & portfolio value
~~~python
print(context.current_dt, context.previous_date, context.portfolio.total_value)
~~~

### Basic order then check trades after close
~~~python
order("000001.XSHE", 100)
for tr in get_trades().values():
    print(tr.time, tr.price, tr.amount)
~~~

### Iterate positions safely
~~~python
for pos in context.portfolio.long_positions.values():
    print(pos.security, pos.total_amount, pos.value)
~~~

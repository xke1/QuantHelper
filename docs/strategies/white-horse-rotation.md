---
title: Strategy Case — White-Horse Rotation
nav_order: 40
permalink: /strategies/white-horse-rotation/
---

# Strategy Case: White-Horse Rotation

A classic **factor-based strategy** using financial quality filters.  
- Rebalances every **100 trading days**  
- Selects companies with **ROE > 20%**, **Gross Profit Margin > 20%**, **Market Cap > 50B RMB**  
- Holds up to **20 stocks**  
- Implements **filters for IPO age & suspension**  
- Uses **order_value** to allocate cash equally  

---

## Full Code

```python
from jqdata import *   # Import JoinQuant API
from datetime import datetime, timedelta  # For date operations

# ---------------- Initialization ----------------
def initialize(context):
    # Set benchmark to CSI300
    set_benchmark('000300.XSHG')
    
    # Use real price to calculate PnL
    set_option('use_real_price', True)
    
    # Allow full position usage
    set_option('order_volume_ratio', 1)
    
    # Set trading costs
    set_order_cost(OrderCost(
        open_tax=0,
        close_tax=0.001,
        open_commission=0.0003,
        close_commission=0.0003,
        close_today_commission=0,
        min_commission=5
    ), type='stock')
    
    # Global parameters
    g.stocknum = 20              # Max number of holdings
    g.days = 0                   # Counter for trading days
    g.refresh_rate = 100         # Rebalance every 100 days
    
    # Run trading function every bar
    run_daily(trade, 'every_bar')


# ---------------- Stock Selection ----------------
def check_stocks(context):
    q = query(
        indicator.code,
        valuation.capitalization,
        valuation.market_cap,
        indicator.roe,
        indicator.gross_profit_margin
    ).filter(
        valuation.capitalization > 50,        # Large capitalization
        indicator.gross_profit_margin > 20,   # High gross margin
        indicator.roe > 20                    # High ROE
    ).order_by(
        valuation.market_cap.desc()
    ).limit(100)

    df = get_fundamentals(q, statDate=str(context.current_dt.year))
    buylist = list(df['code'])
    
    # Apply filters: IPO age & suspension
    buylist = delect_stock(buylist, context.current_dt, 750)
    buylist = filter_paused_stock(buylist)[:20]
    
    return buylist


# ---------------- Filters ----------------
def filter_paused_stock(stock_list):
    """Remove suspended stocks"""
    current_data = get_current_data()
    return [stock for stock in stock_list if not current_data[stock].paused]


def delect_stock(stocks, beginDate, n=180):
    """Remove IPOs listed less than n days"""
    stockList = []
    for stock in stocks:
        start_date = get_security_info(stock).start_date
        if start_date < (beginDate - timedelta(days=n)).date():
            stockList.append(stock)
    return stockList


# ---------------- Trading Logic ----------------
def trade(context):
    g.days += 1
    if g.days % g.refresh_rate == 0:
        stock_list = check_stocks(context)
        hold_list = list(context.portfolio.positions.keys())
        
        # Sell stocks not in the new buylist
        sells = list(set(hold_list).difference(set(stock_list)))
        for stock in sells:
            order_target_value(stock, 0)
        
        # Determine position size
        if len(context.portfolio.positions) < g.stocknum:
            num = g.stocknum - len(context.portfolio.positions)
            cash = context.portfolio.cash / num
        else:
            cash = 0
        
        # Buy new stocks
        for stock in stock_list:
            if len(context.portfolio.positions) < g.stocknum and stock not in context.portfolio.positions:
                order_value(stock, cash)
## Finance Notes

- **ROE (Return on Equity)** → Measures how efficiently equity is used to generate profit.  
- **Gross Profit Margin** → Higher margins indicate stronger business fundamentals.  
- **Market Cap Filter** → Focuses on large-cap stable companies.  
- **Rebalancing** → Periodic updates capture new winners while removing laggards.  
- **IPO Age Filter** → Excludes new listings with unstable financials.  
- **Suspension Filter** → Ensures liquidity and tradability.  

📌 This is a **long-term rotation strategy**, suitable for investors seeking **stable compound growth** rather than short-term speculation.

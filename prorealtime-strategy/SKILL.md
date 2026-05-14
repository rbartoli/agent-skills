---
name: prorealtime-strategy
description: Write automated trading strategies for ProRealTime using ProBuilder language (ProBacktest/ProOrder). Use this skill whenever the user asks to create, fix, optimize, or modify a ProRealTime trading strategy, mentions ProBuilder code, ProOrder, ProBacktest, or asks for help with PRT/ProRealTime coding. Also trigger when the user describes a trading idea and wants it coded for ProRealTime — even if they just say "write me a strategy for..." or describe entry/exit conditions. Covers forex (LOTS) and stocks (SHARES), trend-following, mean-reversion, and any systematic strategy logic.
---

# ProRealTime Strategy Writer

You write ProBuilder code for ProRealTime's ProBacktest and ProOrder engines. The user knows trading concepts — they need correct, complete, paste-ready code that avoids the platform's many gotchas.

## Output Format

Always output a single fenced code block containing the complete strategy. No explanations outside the code unless the user asks. The code must be immediately loadable into ProRealTime's strategy editor.

```
// Strategy Name
// Description: one-line summary
// Timeframe: recommended timeframe
// Instrument: target market type

DEFPARAM CumulateOrders = False
...strategy code...
```

## Variable Naming — Read This First

ProBuilder's parser is picky about variable names. The official documentation consistently uses short, clear names. Follow these patterns to avoid "undefined variable" errors:

**Safe patterns from official examples:**
- Short descriptive: `c1`, `c2`, `n`, `x`, `y`, `val`
- CamelCase: `BuyLevel`, `SellLevel`, `OrderSize`, `DayDistance`
- "My" prefix: `MyTarget`, `MyResistance`, `MySupport`, `MyADX12`, `MyMM20`
- Numbered conditions: `Condition1`, `Condition2`, `Condition3`

**Rules:**
- No underscores `_` or hyphens `-` in variable names (only in comments)
- No special characters — only letters and digits
- Language is case-insensitive (`myVar` = `MYVAR`)
- Don't name variables identically to built-in functions (`Close`, `RSI`, `ADX`, `Volume`, etc.)
- No variable declaration needed — just assign: `n = 14`

**ATR is NOT a valid function name.** The correct function is `AverageTrueRange[N](price)`. This is the single most common error. Never write `ATR[14]` — always write `AverageTrueRange[14](Close)`.

## Code Structure Rules

Every strategy follows this skeleton:

```
// 1. DEFPARAM declarations (MUST be first executable lines)
// 2. Parameters as named variables
// 3. Indicator calculations
// 4. Entry conditions with position guards
// 5. Exit conditions
// 6. Risk management (SET STOP / SET TARGET)
```

Constraints that reflect how ProRealTime actually parses and executes:

1. **DEFPARAM lines must be the very first executable lines** — before any variable assignment or logic. The engine rejects code that places DEFPARAM after other statements.

2. **One position at a time** unless the user explicitly requests pyramiding. Always include `DEFPARAM CumulateOrders = False`.

3. **Guard entries** with `IF NOT LongOnMarket` / `IF NOT ShortOnMarket` to prevent duplicate orders. Without guards, ProOrder silently rejects orders when CumulateOrders is False, making it look like the strategy "doesn't trade."

4. **AT MARKET orders execute at next bar's open** — this is how ProRealTime works. Don't promise same-bar execution.

5. **No semicolons.** ProBuilder doesn't use them. Comments use `//` for single-line.

6. **All quantity keywords are interchangeable:** SHARES = SHARE = LOT = LOTS = CONTRACT = CONTRACTS = PERPOINT. Use whatever matches the asset type for clarity.

7. **SET STOP / SET TARGET accept only simple variables or constants, NOT expressions.** Pre-calculate any formula into a variable first:
```
// WRONG — will fail:
SET STOP pLOSS AverageTrueRange[14](Close) * 2

// CORRECT:
MyATR = AverageTrueRange[14](Close)
StopDist = MyATR * 2 / pipsize
SET STOP pLOSS StopDist
```

8. **SET STOP / SET TARGET CAN be inside IF blocks** for conditional risk management. They can also be placed at the bottom outside any blocks for global application.

9. **TIMEFRAME blocks**: always return to `TIMEFRAME(DEFAULT)` after reading a higher timeframe. Secondary timeframe must be a multiple of the base chart timeframe. Maximum 10 intraday TIMEFRAME instructions. Variables assigned in a TIMEFRAME block retain their value after switching back but cannot be reassigned in a different timeframe.

10. **ONCE executes during the PRELOADBARS period too** — not just on the first "real" bar. Keep this in mind when initializing counters or flags.

11. **GRAPH, GRAPHONPRICE, and PRINT cannot be used in ProOrder** (live automatic trading). They only work in ProBacktest.

12. **Indicators not usable in ProOrder:** ZigZag, ZigZagPoint, DPO. Avoid these in strategies intended for live trading.

## Asset-Specific Defaults

**Forex:**
- Position sizing: `BUY n LOTS AT MARKET`
- Typical stops: pip-based (`SET STOP pLOSS 30`) or percentage (`SET STOP %LOSS 1`)
- Session filter: consider `DEFPARAM FlatBefore` / `DEFPARAM FlatAfter` for session boundaries
- Minimum effective timeframe is 15 minutes
- Use `pipsize` in calculations that reference price distances
- `pipsize` = `PointSize`, `PipValue` = `PointValue` (synonyms)

**Stocks:**
- Position sizing: `BUY n SHARES AT MARKET` or `BUY n CASH AT MARKET`
- Typical stops: percentage-based (`SET STOP %LOSS 2`) or ATR-based
- Session filter: usually `DEFPARAM FlatBefore = 093000` and `DEFPARAM FlatAfter = 155000`

## Money Management (Position Sizing)

When the user asks for risk-based position sizing:

```
Capital = 10000
Risk = 0.01
MyATR = AverageTrueRange[14](Close)
MyStop = 2 * MyATR

equity = Capital + StrategyProfit
maxrisk = ROUND(equity * Risk)
PositionSize = ABS(ROUND((maxrisk / MyStop) / PointValue) * pipsize)

IF NOT LongOnMarket AND buyCondition THEN
    BUY PositionSize CONTRACTS AT MARKET
ENDIF
```

Key variables:
- `StrategyProfit` — cumulative P&L of closed trades (built-in)
- `PointValue` — monetary value of one point/pip in the instrument's currency
- `pipsize` — size of one pip (0.0001 for most forex, 1 for indices)

## All SET STOP / SET TARGET Variants

### Stop Loss
```
SET STOP LOSS x          // x units from entry (absolute price distance)
SET STOP pLOSS x         // x points from entry
SET STOP %LOSS x         // x% loss from entry price
SET STOP $LOSS x         // x currency units loss
SET STOP BREAKEVEN       // Move stop to entry price
SET STOP PRICE x         // Stop at specific price level
```

### Profit Target
```
SET TARGET PROFIT x      // x units from entry
SET TARGET pPROFIT x     // x points from entry
SET TARGET %PROFIT x     // x% gain from entry price
SET TARGET $PROFIT x     // x currency units gain
SET TARGET PRICE x       // Target at specific price level
```

### Trailing Stop
```
SET STOP TRAILING x      // Trail x units from highest profit
SET STOP pTRAILING x     // Trail x points from highest profit
SET STOP %TRAILING x     // Trail x% from highest profit
SET STOP $TRAILING x     // Trail x currency units from highest profit
```

## Platform Gotchas (Critical)

### Backtesting vs Live Divergence
- **Spread is fixed in backtest, variable in live.** During news, live spreads widen massively.
- **Preload bars matter.** Default is only 200. If your strategy uses long-period indicators (MA 200, etc.), increase with `DEFPARAM PreLoadBars = 2000`. Insufficient preload causes garbage indicator values at backtest start.

### Order Rejections in Live Trading
- **Minimum distance rules:** Brokers (especially IG) reject stops/limits placed too close to current price. Use stops of at least 10-15 pips on forex, or ATR-based stops that adapt.
- **ProOrder retries 10 times** — after 10 fails, the order is cancelled and the strategy may be paused.

### Execution Model
- Code runs once per bar close in ProBacktest/ProOrder
- Multiple orders in the same bar: only the first valid one executes
- `SELL AT MARKET` + `SELLSHORT AT MARKET` in same bar: only SELL fires; SELLSHORT executes next bar

## Avoiding Overfitting

1. **Fewer parameters is better.** 3-5 optimizable parameters max.
2. **Walk Forward Efficiency (WFE) of 50%+** indicates robustness.
3. **At least 100+ trades** in backtest for statistical significance.
4. **Out-of-sample testing:** Optimize on 70% of data, verify on remaining 30%.

## When the User is Vague

Fill in reasonable defaults:
- Standard indicator periods (RSI 14, MA 20/50, Bollinger 20)
- Default to 1-hour timeframe for forex, daily for stocks
- Include risk management (1-2% risk per trade via stops)
- Add `DEFPARAM CumulateOrders = False`
- Include both entry AND exit logic
- Add a time filter for intraday strategies

State assumptions in a comment block at the top.

## Common Strategy Templates

### Trend-Following (Moving Average Crossover)
- Fast MA crosses over slow MA: BUY
- Fast MA crosses under slow MA: SELL / SELLSHORT
- ADX > 20 filter to confirm trend exists
- Trailing stop to let winners run

### Mean-Reversion (Bollinger / RSI)

**Entry — use a rejection candle, not a simple band touch:**
- Previous bar's low pierced below lower Bollinger AND current bar closes back inside + RSI was < 30 in last 3 bars: BUY
- Previous bar's high pierced above upper Bollinger AND current bar closes back inside + RSI was > 70 in last 3 bars: SELLSHORT
- The rejection candle (close back inside after piercing) is critical — simple band-touch entries have much lower win rates in live testing

**Filters:**
- ADX < 25-30 to avoid trading during strong trends
- Bollinger Band width filter: only trade when `BandWidth < Average[50](BandWidth)` — narrow bands indicate low-volatility consolidation where mean reversion works best. This filter alone dramatically improves signal quality and can replace session-based time filters
- Prefer volatility-based filters (band width, ADX) over session/time filters — they adapt to actual market conditions rather than assuming fixed session behavior

**Exit — require dual confirmation, not just middle band:**
- Exit when price reaches middle band AND RSI has recovered past 50 (e.g., RSI > 55 for longs, RSI < 45 for shorts). Requiring both conditions lets winners run longer and significantly improves the reward-to-risk ratio (~2.5:1 vs ~1:1 with single exit)
- Add a time stop (10-15 bars) as safety net — if price hasn't reverted by then, a trend may have started

**Stops:**
- 1.5x AverageTrueRange for stop distance — wide enough to survive noise, tight enough to limit losses
- Pre-calculate: `StopDist = MyATR * 1.5 / pipsize` then `SET STOP pLOSS StopDist`

### Breakout
- Price breaks above highest[n]: BUY
- Price breaks below lowest[n]: SELLSHORT
- ATR-based stops (2x AverageTrueRange typically)

## Debugging Help

When the user shares code that isn't working:
1. Check DEFPARAM placement (must be first executable lines)
2. Check for `ATR` — must be `AverageTrueRange[N](price)`
3. Check for underscores or special characters in variable names
4. Check for missing ENDIF / NEXT / WEND
5. Verify position guards (NOT LongOnMarket / NOT ShortOnMarket)
6. Check TIMEFRAME(DEFAULT) is restored after higher-TF reads
7. Look for typos in indicator syntax — brackets `[n]` for period, parentheses `(price)` for data source
8. Check quantity keywords are plural: LOTS, SHARES, CONTRACTS
9. Check SET STOP value isn't too small — broker may reject for minimum distance
10. Verify the strategy has exit logic — without it, positions stay open forever

## Reference

For the complete ProBuilder language specification (all functions, operators, indicators, and order commands), read `references/probuilder-reference.md`.

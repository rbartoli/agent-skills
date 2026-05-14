# ProBuilder Language Reference (ProBacktest & ProOrder)

Source: Official ProRealTime documentation (probuilder.pdf v6.1.0, probacktest.pdf v6.1.0)

## Variable Naming Rules

- Case-insensitive language (no difference between `myVar` and `MYVAR`)
- No declaration or typing needed — just assign: `x = 10`
- Variables are "historised" (store one value per bar, accessible via `x[1]`, `x[2]`, etc.)
- **No underscores `_` or hyphens `-` in variable names** (only allowed in comments)
- **No special characters** — only letters, digits
- Avoid naming variables identically to built-in functions (e.g., don't name a variable `Close`, `RSI`, `ADX`)
- Unassigned variables appear as optimizable parameters in the ProBacktest UI

## Price & Volume Constants

| Constant | Description |
|----------|-------------|
| `Open`, `High`, `Low`, `Close`, `Volume` | Current bar OHLCV |
| `Close[1]`, `Close[n]` | Value n bars ago |
| `DOpen(n)`, `DHigh(n)`, `DLow(n)`, `DClose(n)` | Daily-level values (n=0 is current day) |
| `Range` | High - Low of current candle |
| `TypicalPrice` | (High + Low + Close) / 3 |
| `TotalPrice` | (Close + Open + High + Low) / 4 |
| `WeightedClose` | (2 * Close + High + Low) / 4 |
| `MedianPrice` | (High + Low) / 2 |
| `CustomClose` | Customizable in chart settings (default: Close) |
| `Ticksize` | Minimum price variation of the instrument |
| `TR(price)` | True Range |
| `Variation(price)` | % difference between last bar close and current close |

## Time Constants

| Constant | Format/Description |
|----------|-------------------|
| `Date` | YYYYMMDD |
| `Time` | HHMMSS |
| `Today` | Today's date YYYYMMDD |
| `Yesterday` | Yesterday[N] — date N periods before current bar |
| `Hour`, `Minute`, `Second` | Bar time components |
| `Day`, `Month`, `Year` | Bar date components |
| `DayOfWeek` | 0=Sunday..6=Saturday |
| `OpenDay`, `OpenHour`, `OpenMinute`, `OpenMonth`, `OpenSecond`, `OpenTime`, `OpenWeek`, `OpenYear` | Opening values of bar |
| `CurrentDayOfWeek`, `CurrentHour`, `CurrentMinute`, `CurrentMonth`, `CurrentSecond`, `CurrentTime`, `CurrentYear` | Real-time clock values |
| `BarIndex` | Sequential bar counter from start of loaded data |
| `IntradayBarIndex` | Resets to 0 each trading day |
| `Days` | Counter of days since 1900 |
| `Timestamp` | UNIX timestamp of bar close |
| `OpenTimestamp` | UNIX timestamp of bar open |
| `GetTimeFrame` | Returns current period in seconds |

## Control Structures

```
// Conditional
IF condition THEN
  ...
ELSIF condition THEN
  ...
ELSE
  ...
ENDIF

// FOR loop (ascending)
FOR i = 0 TO n DO
  ...
NEXT

// FOR loop (descending)
FOR i = n DOWNTO 0 DO
  ...
NEXT

// WHILE loop
WHILE condition DO
  ...
WEND

// Execute only on first bar (also runs during PRELOADBARS period)
ONCE variableName = value

// Exit loop
BREAK

// Skip to next iteration
CONTINUE
```

## Operators

### Comparison
`<`, `<=`, `>`, `>=`, `=`, `<>`

### Logical
`AND`, `OR`, `NOT`, `XOR`

### Arithmetic
`+`, `-`, `*`, `/`, `MOD`

### Crossing
`CROSSES OVER`, `CROSSES UNDER`

## Math Functions

| Function | Description |
|----------|-------------|
| `ABS(a)` | Absolute value |
| `CEIL(N,M)` | Smallest number >= N at decimal M |
| `COS(a)` | Cosine (degrees) |
| `EXP(a)` | Exponential |
| `FLOOR(N,M)` | Largest number <= N at decimal M |
| `LOG(a)` | Natural logarithm |
| `MAX(a,b)` | Maximum |
| `MIN(a,b)` | Minimum |
| `POW(N,P)` | N to the power P |
| `RANDOM(Min,Max)` | Random integer between Min and Max |
| `ROUND(a)` | Round to nearest whole number |
| `SGN(a)` | Sign of a (-1, 0, or 1) |
| `SIN(a)` | Sine (degrees) |
| `SQRT(a)` | Square root |
| `SQUARE(a)` | a squared |
| `TAN(a)` | Tangent (degrees) |
| `ACOS(a)` | Arc cosine (degrees) |
| `ASIN(a)` | Arc sine (degrees) |
| `ATAN(a)` | Arc tangent (degrees) |

## Statistical / Aggregation Functions

| Function | Description |
|----------|-------------|
| `STD[n](data)` | Standard deviation over n bars |
| `STE[n](data)` | Standard error over n bars |
| `highest[n](data)` | Highest value over n bars |
| `lowest[n](data)` | Lowest value over n bars |
| `Summation[n](data)` | Sum over n bars |
| `cumsum(price)` | Cumulative sum over all loaded data |
| `BarsSince(condition,n)` | Bars since nth occurrence of condition (n=0 default) |

## Built-in Indicators — Complete List

Syntax: `IndicatorName[period](price)` or `IndicatorName[p1,p2](price)`

### Trend / Moving Averages
| Indicator | Syntax | Description |
|-----------|--------|-------------|
| `Average` | `Average[N](price)` | Simple Moving Average |
| `ExponentialAverage` | `ExponentialAverage[N](price)` | Exponential Moving Average |
| `WeightedAverage` | `WeightedAverage[N](price)` | Weighted Moving Average |
| `WilderAverage` | `WilderAverage[N](price)` | Wilder Moving Average |
| `TriangularAverage` | `TriangularAverage[N](price)` | Triangular Moving Average |
| `EndPointAverage` | `EndPointAverage[N](price)` | End Point Moving Average |
| `TimeSeriesAverage` | `TimeSeriesAverage[N](price)` | Time Series (Least Squares) MA |
| `HullAverage` | `HullAverage[N](close)` | Hull Moving Average |
| `DEMA` | `DEMA[N](price)` | Double Exponential MA |
| `TEMA` | `TEMA[N](price)` | Triple Exponential MA |
| `TRIX` | `TRIX[N](price)` | Triple Smoothed EMA |
| `VolumeAdjustedAverage` | `VolumeAdjustedAverage[N](price)` | Volume Adjusted MA |
| `AdaptiveAverage` | `AdaptiveAverage[x,y,z](price)` | Adaptive Average |
| `LinearRegression` | `LinearRegression[N](price)` | Linear Regression |
| `LinearRegressionSlope` | `LinearRegressionSlope[N](price)` | Slope of Linear Regression |

### Directional / Trend Strength
| Indicator | Syntax | Description |
|-----------|--------|-------------|
| `ADX` | `ADX[N]` | Average Directional Index |
| `ADXR` | `ADXR[N]` | ADX Rate |
| `DIplus` | `DIplus[N](price)` | DI+ indicator |
| `DIminus` | `DIminus[N](price)` | DI- indicator |
| `DI` | `DI[N](price)` | Demand Index |
| `SAR` | `SAR[At,St,Lim]` | Parabolic SAR |
| `SARatdmf` | `SARatdmf[At,St,Lim](price)` | Smoothed Parabolic SAR |
| `Supertrend` | `Supertrend[STF,N]` | Super Trend |
| `R2` | `R2[N](price)` | R-Squared (error rate of linear regression) |

### Ichimoku
| Indicator | Syntax | Description |
|-----------|--------|-------------|
| `TenkanSen` | `TenkanSen[T,K,S]` | Tenkan-Sen line |
| `KijunSen` | `KijunSen[T,K,S]` | Kijun-Sen line |
| `SenkouSpanA` | `SenkouSpanA[T,K,S]` | Senkou Span A |
| `SenkouSpanB` | `SenkouSpanB[T,K,S]` | Senkou Span B |

### Momentum / Oscillators
| Indicator | Syntax | Description |
|-----------|--------|-------------|
| `RSI` | `RSI[N](price)` | Relative Strength Index |
| `Stochastic` | `Stochastic[N,K](price)` | %K line of Stochastic |
| `Stochasticd` | `Stochasticd[N,K,D](price)` | %D line of Stochastic |
| `SmoothedStochastic` | `SmoothedStochastic[N,K](price)` | Smoothed Stochastic |
| `SMI` | `SMI[N,SS,DS](price)` | Stochastic Momentum Index |
| `MACD` | `MACD[S,L,Si](price)` | MACD histogram |
| `MACDLine` | `MACDLine[S,L,Si](price)` | MACD line |
| `MACDSignal` | `MACDSignal[S,L,Si](price)` | MACD signal line |
| `CCI` | `CCI[N](price)` | Commodity Channel Index |
| `Momentum` | `Momentum[I](price)` | Momentum |
| `ROC` | `ROC[N](price)` | Rate of Change |
| `PriceOscillator` | `PriceOscillator[S,L](price)` | Price Oscillator |
| `Williams` | `Williams[N](close)` | Williams %R |
| `AroonUp` | `AroonUp[P]` | Aroon Up |
| `AroonDown` | `AroonDown[P]` | Aroon Down |
| `Chandle` | `Chandle[N](price)` | Chande Momentum Oscillator |
| `DPO` | `DPO[N](price)` | Detrended Price Oscillator (NOT usable in ProOrder) |
| `Cycle` | `Cycle(price)` | Cycle indicator |
| `RocnRoll` | `RocnRoll(price)` | RocnRoll indicator |

### Volatility
| Indicator | Syntax | Description |
|-----------|--------|-------------|
| `AverageTrueRange` | `AverageTrueRange[N](price)` | ATR (Wilder smoothing). **NOT `ATR` — that is invalid** |
| `BollingerUp` | `BollingerUp[N](price)` | Upper Bollinger Band |
| `BollingerDown` | `BollingerDown[N](price)` | Lower Bollinger Band |
| `BollingerBandWidth` | `BollingerBandWidth[N](price)` | Bollinger Band Width |
| `KeltnerBandCenter` | `KeltnerBandCenter[N]` | Keltner Channel center |
| `KeltnerBandUp` | `KeltnerBandUp[N]` | Keltner Channel upper |
| `KeltnerBandDown` | `KeltnerBandDown[N]` | Keltner Channel lower |
| `DonchianChannelCenter` | `DonchianChannelCenter[N]` | Donchian Channel center |
| `DonchianChannelUp` | `DonchianChannelUp[N]` | Donchian Channel upper |
| `DonchianChannelDown` | `DonchianChannelDown[N]` | Donchian Channel lower |
| `ChandeKrollStopUp` | `ChandeKrollStopUp[Pp,Qq,X]` | Chande Kroll Stop (long) |
| `ChandeKrollStopDown` | `ChandeKrollStopDown[Pp,Qq,X]` | Chande Kroll Stop (short) |
| `HistoricVolatility` | `HistoricVolatility[N](price)` | Historic Volatility |
| `Volatility` | `Volatility[S,L]` | Chaikin Volatility |
| `FractalDimensionIndex` | `FractalDimensionIndex[N](close)` | Fractal Dimension Index |
| `PRTBandsUp` | `PRTBandsUp` | PRT Bands upper |
| `PRTBandsDown` | `PRTBandsDown` | PRT Bands lower |
| `PRTBandShortTerm` | `PRTBandShortTerm` | PRT Bands short term |
| `PRTBandMediumTerm` | `PRTBandMediumTerm` | PRT Bands medium term |
| `MassIndex` | `MassIndex[N]` | Mass Index |

### Volume
| Indicator | Syntax | Description |
|-----------|--------|-------------|
| `OBV` | `OBV(price)` | On Balance Volume |
| `MoneyFlow` | `MoneyFlow[N](price)` | Money Flow (-1 to 1) |
| `MoneyFlowIndex` | `MoneyFlowIndex[N]` | Money Flow Index |
| `AccumDistr` | `AccumDistr(price)` | Accumulation/Distribution |
| `ChaikinOsc` | `ChaikinOsc[Ch1,Ch2](price)` | Chaikin Oscillator |
| `ForceIndex` | `ForceIndex(price)` | Force Index |
| `EaseOfMovement` | `EaseOfMovement[l]` | Ease of Movement |
| `EMV` | `EMV[N]` | Ease of Movement Value |
| `NegativeVolumeIndex` | `NegativeVolumeIndex[N]` | Negative Volume Index |
| `PositiveVolumeIndex` | `PositiveVolumeIndex(price)` | Positive Volume Index |
| `PVT` | `PVT(price)` | Price Volume Trend |
| `VolumeOscillator` | `VolumeOscillator[S,L]` | Volume Oscillator |
| `VolumeROC` | `VolumeROC[N]` | Volume Rate of Change |
| `WilliamsAccumDistr` | `WilliamsAccumDistr(price)` | Williams Accumulation/Distribution |

### Other
| Indicator | Syntax | Description |
|-----------|--------|-------------|
| `Repulse` | `Repulse[N](price)` | Repulse indicator |
| `RepulseMM` | `RepulseMM[N,Period,factor](price)` | Repulse moving average |
| `SmoothedRepulse` | `SmoothedRepulse[N](price)` | Smoothed Repulse |
| `ElderrayBearPower` | `ElderrayBearPower[N](close)` | Elder Ray Bear Power |
| `ElderrayBullPower` | `ElderrayBullPower[N](close)` | Elder Ray Bull Power |
| `DivergenceCCI` | `DivergenceCCI[Div1,Div2,Div3,Div4]` | CCI Divergence |
| `DivergenceMACD` | `DivergenceMACD[Div1,Div2,Div3,Div4](close)` | MACD Divergence |
| `DynamicZoneRSIUp` | `DynamicZoneRSIUp[rsiN,N]` | Dynamic Zone RSI upper |
| `DynamicZoneRSIDown` | `DynamicZoneRSIDown[rsiN,N]` | Dynamic Zone RSI lower |
| `DynamicZoneStochasticUp` | `DynamicZoneStochasticUp[N]` | Dynamic Zone Stochastic upper |
| `DynamicZoneStochasticDown` | `DynamicZoneStochasticDown[N]` | Dynamic Zone Stochastic lower |
| `ViPlus` | `ViPlus[N]` | Vortex Indicator positive |
| `ViMinus` | `ViMinus[N]` | Vortex Indicator negative |
| `ZigZag` | `ZigZag[Zr](price)` | ZigZag (NOT usable in ProOrder) |
| `ZigZagPoint` | `ZigZagPoint[Zp](price)` | ZigZag Point (NOT usable in ProOrder) |

## DEFPARAM (Strategy Parameters)

Must be the FIRST executable lines, before any variable assignment or logic.

```
DEFPARAM CumulateOrders = False     // False = one position at a time
DEFPARAM PreLoadBars = 200          // Bars preloaded for indicator warmup (0-5000)
DEFPARAM FlatBefore = 090000        // Close positions before this time (HHMMSS)
DEFPARAM FlatAfter = 173000         // No new positions after this time (HHMMSS)
DEFPARAM NoCashUpdate = true        // Don't update cash with P&L during backtest
```

## Order Commands

### Entry Orders
```
BUY n SHARES AT MARKET              // Market buy
BUY n LOTS AT MARKET                // Same (LOTS = SHARES = CONTRACTS = LOT = SHARE = CONTRACT = PERPOINT)
BUY n CONTRACTS AT MARKET           // Same
BUY n CASH AT MARKET                // Buy with cash amount
BUY n SHARES AT price LIMIT         // Limit buy (price must be BELOW market)
BUY n SHARES AT price STOP          // Stop buy (price must be ABOVE market)

SELLSHORT n SHARES AT MARKET        // Short entry
SELLSHORT n SHARES AT price LIMIT   // Short limit (price must be ABOVE market)
SELLSHORT n SHARES AT price STOP    // Short stop (price must be BELOW market)
```

### Exit Orders
```
SELL AT MARKET                       // Close long position
SELL n SHARES AT MARKET              // Partial close long
SELL AT price LIMIT                  // Limit exit long (price above market)
SELL AT price STOP                   // Stop exit long (price below market)

EXITSHORT AT MARKET                  // Close short position
EXITSHORT n SHARES AT MARKET         // Partial close short
EXITSHORT AT price LIMIT             // Limit exit short (price below market)
EXITSHORT AT price STOP              // Stop exit short (price above market)
```

### Order Type Rules
- **LIMIT**: Entry price more favorable than current (buy lower, short higher)
- **STOP**: Entry price less favorable than current (buy higher, short lower)
- **MARKET**: Execute at next bar's open
- All quantity keywords are interchangeable: SHARES = SHARE = LOT = LOTS = CONTRACT = CONTRACTS = PERPOINT

## Risk Management — All Variants

### Stop Loss
| Syntax | Description |
|--------|-------------|
| `SET STOP LOSS x` | Stop at x units (absolute price distance) from entry |
| `SET STOP pLOSS x` | Stop at x points from entry |
| `SET STOP %LOSS x` | Stop at x% loss from entry price |
| `SET STOP $LOSS x` | Stop at x currency units loss |
| `SET STOP BREAKEVEN` | Move stop to entry price (break-even) |
| `SET STOP PRICE x` | Stop/target at specific price level x |

### Profit Target
| Syntax | Description |
|--------|-------------|
| `SET TARGET PROFIT x` | Target at x units from entry |
| `SET TARGET pPROFIT x` | Target at x points from entry |
| `SET TARGET %PROFIT x` | Target at x% gain from entry price |
| `SET TARGET $PROFIT x` | Target at x currency units gain |
| `SET TARGET PRICE x` | Target at specific price level x |

### Trailing Stop
| Syntax | Description |
|--------|-------------|
| `SET STOP TRAILING x` | Trail x units from highest profit |
| `SET STOP pTRAILING x` | Trail x points from highest profit |
| `SET STOP %TRAILING x` | Trail x% from highest profit |
| `SET STOP $TRAILING x` | Trail x currency units from highest profit |

### Important Notes
- SET STOP / SET TARGET accept only simple variables or constants, NOT expressions. Pre-calculate into a variable first.
- CAN be placed inside IF blocks for conditional stops.
- `QUIT` stops the trading system entirely.

## Position Management Variables

| Variable | Description |
|----------|-------------|
| `LongOnMarket` | 1 if long position open |
| `ShortOnMarket` | 1 if short position open |
| `NotOnMarket` | 1 if no position open |
| `OnMarket` | 1 if any position open |
| `CountOfPosition` | Number of units in position (positive=long, negative=short) |
| `CountOfLongShares` | Number of securities in long position |
| `CountOfShortShares` | Number of securities in short position |
| `PositionPrice` | Average entry price of current position |
| `PositionPerf(n)` | Performance of nth closed position (% gain/loss, n=1 is most recent) |
| `TradePrice(n)` | Price of nth last executed order |
| `TradeIndex(n)` | BarIndex of nth last executed order (n=0 is most recent) |
| `StrategyProfit` | Cumulative P&L of all closed trades |
| `LongTriggered[N]` | 1 if a buy order was executed N bars ago |
| `ShortTriggered[N]` | 1 if a short order was executed N bars ago |

## Multi-Timeframe

```
TIMEFRAME(period, mode)

// Period examples: 1 minute, 5 minutes, 15 minutes, 1 hour, 1 day, 1 week, 1 month
// Also: X Ticks, X Seconds, X Hours, X Days, X Weeks, X Months, X Years
// Mode: DEFAULT (update each tick), UPDATEONCLOSE (update at period close)

TIMEFRAME(1 hour, UPDATEONCLOSE)
myHourlyMA = Average[20](Close)

TIMEFRAME(DEFAULT)  // Return to chart timeframe
// Use myHourlyMA in logic below
```

**Rules:**
- Secondary timeframe must be a multiple of the base chart timeframe
- Maximum 10 intraday TIMEFRAME instructions per strategy
- Variables assigned inside a TIMEFRAME block retain value after switching back
- Variables CANNOT be reassigned in a different timeframe than where they were first assigned
- Always return to `TIMEFRAME(DEFAULT)` before entry/exit logic

## Arrays

```
// Prefixed with $, indexed 0 to 999999
$myArray[0] = 10
$myArray[1] = 20
val = $myArray[0]

// Functions
ArrayMax($myArray)       // Highest value
ArrayMin($myArray)       // Lowest value
ArraySort($myArray, 0)   // 0=ascending, 1=descending
LastSet($myArray)        // Last defined index
IsSet($myArray[index])   // 1 if index is defined, else 0
UnSet($myArray)          // Reset all data
```

**Arrays are NOT historised** (unlike regular variables).

## Output & Debugging

```
GRAPH variable AS "Label"            // Plot on strategy subchart (ProBacktest only)
GRAPHONPRICE variable AS "Label"     // Plot on price chart (ProBacktest only)
PRINT variable                       // Debug output to separate window
```

**GRAPH, GRAPHONPRICE, and PRINT are NOT available in ProOrder (live trading).**

## Instrument Properties

| Variable | Description |
|----------|-------------|
| `Pipsize` | Size of one pip (= PointSize). E.g., 0.0001 for EURUSD, 1 for indices |
| `PointSize` | Same as Pipsize |
| `PipValue` | Monetary value of one pip in instrument currency (= PointValue) |
| `PointValue` | Same as PipValue |

## Special Instructions

| Instruction | Description |
|-------------|-------------|
| `CALL myFunction` | Call a user-defined indicator |
| `QUIT` | Stop the trading system |
| `Undefined` | Set a variable to undefined: `a = Undefined` |

## Indicators NOT Usable in ProOrder

These indicators cannot be used in automatic trading (ProOrder):
- `ZigZag`
- `ZigZagPoint`
- `DPO` (Detrended Price Oscillator)

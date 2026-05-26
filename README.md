# Cumulative Delta Alert — MQL4 Script

A MetaTrader 4 script that computes a **cumulative price delta** across a rolling lookback window by accumulating close-minus-open differences bar-by-bar via `iClose()` and `iOpen()`, tracking the cycle-to-cycle change in the cumulative total against a persistent `previousDelta` global, and firing buyer or seller dominance alerts when the absolute shift between consecutive cycles exceeds the configurable `AlertThreshold` — providing a lightweight proxy for order flow imbalance detection without requiring tick data.

---

## Overview

Cumulative delta analysis is a technique borrowed from professional order flow trading, where "delta" represents the net difference between aggressive buying and selling pressure over a period. True cumulative delta uses tick-by-tick bid/ask aggressor classification, which is unavailable to most retail MT4 deployments. This script implements a practical bar-based approximation: for each bar in the lookback window, `close − open` is computed as the bar's net directional contribution — a positive value indicates buyers controlled that bar's range; a negative value indicates sellers. Summing these across `LookbackPeriod` bars produces a cumulative delta that captures the aggregate directional pressure over the window. When this sum changes by more than `AlertThreshold` between consecutive cycles, it signals a meaningful shift in buyer or seller dominance that may precede a directional move.

> **Note on file naming:** This file is distributed as `Liquidity_Zones_001.mq4` but implements a Cumulative Delta alert. The README documents the actual implemented logic.

---

## Features

- **Bar-based cumulative delta accumulation** — iterates `i = 0` to `LookbackPeriod − 1`, accumulating `iClose(i) − iOpen(i)` into `delta`; sufficient-bar guard (`iBars() < period`) returns `0.0` with a log message
- **Cycle-to-cycle shift detection** — `MathAbs(cumulativeDelta − previousDelta) >= AlertThreshold` evaluates the magnitude of change between consecutive minute-loop readings, not the absolute delta value
- **Directional dominance classification** — shift direction determines alert type: `cumulativeDelta > previousDelta` → **Buyer Dominance Detected** (delta expanding positively); `cumulativeDelta < previousDelta` → **Seller Dominance Detected** (delta contracting or expanding negatively)
- **Persistent `previousDelta` state** — global double updated unconditionally at cycle end, maintaining the prior cumulative total for next-cycle comparison
- **Alert message includes both current and previous delta** — `AlertCumulativeDelta()` formats with `"Current Delta: %.2f\nPrevious Delta: %.2f"` for full shift-context transparency
- **Three notification channels:** sound alert, email, and mobile push
- **Default `PERIOD_M1` timeframe** — designed for short-timeframe order flow proxy analysis; configurable to any timeframe
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateCumulativeDelta()` validates bar count, then iterates `LookbackPeriod` bars accumulating `close − open` per bar
2. `MathAbs(cumulativeDelta − previousDelta) >= AlertThreshold` evaluated:
   - Net positive shift → **Buyer Dominance Detected**
   - Net negative shift → **Seller Dominance Detected**
3. `previousDelta = cumulativeDelta` updated unconditionally at cycle end

---

## Input Parameters

| Parameter        | Type            | Default      | Description                                                           |
|------------------|-----------------|--------------|-----------------------------------------------------------------------|
| `TradeSymbol`    | string          | `EURUSD`     | Symbol for analysis                                                   |
| `Timeframe`      | ENUM_TIMEFRAMES | `PERIOD_M1`  | Timeframe for delta calculation (M1 recommended for order flow proxy) |
| `LookbackPeriod` | int             | `100`        | Number of bars to accumulate delta across                             |
| `AlertThreshold` | double          | `100.0`      | Minimum absolute delta shift between cycles to trigger an alert       |
| `EnableAlerts`   | bool            | `true`       | Fire an on-screen/sound alert                                         |
| `EnableEmail`    | bool            | `false`      | Send an email notification                                            |
| `EnablePush`     | bool            | `false`      | Send a mobile push notification                                       |

---

## Alert Message Format

```
Buyer Dominance Detected detected on EURUSD (Timeframe: PERIOD_M1)
Current Delta: 0.00842
Previous Delta: -0.00231
```

---

## Installation

1. Copy `Liquidity_Zones_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

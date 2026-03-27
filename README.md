# FX Signals — Engulfing Strategy & Indicator

Pine Script **v6** tools for TradingView: a **strategy** that backtests and places orders with fixed dollar risk, and a matching **indicator** for the same signals, visuals, and an optional status table.

Repository: [github.com/quynhchi1009/FX_Signals](https://github.com/quynhchi1009/FX_Signals)

## Files

| File | Type | Purpose |
|------|------|---------|
| `engulfing_candle_strategy.pine` | Strategy | Backtesting, Strategy Tester, entries/exits with stop + limit |
| `engulfing_candle_indicator.pine` | Indicator | Arrows, bar colors, boxes, optional SL/TP lines, filter dashboard |

## Requirements

- [TradingView](https://www.tradingview.com/) account
- Pine **version 6** (for `active=` on inputs so dependent settings gray out when a filter is off)
- **Regular candlestick** chart recommended (not Heikin Ashi) so OHLC matches typical execution and backtests

## Quick start

1. Open **Pine Editor** on a chart.
2. **New** → paste `engulfing_candle_strategy.pine` → **Save** → **Add to chart** (strategy).
3. Optionally add **indicator**: paste `engulfing_candle_indicator.pine` → add to the same chart.
4. Open **Settings** on each script and align inputs if you want signals to match (see [Keeping strategy and indicator in sync](#keeping-strategy-and-indicator-in-sync)).

## Engulfing definition

Evaluated on the **current** bar vs the **previous** bar.

**Wick range**

- Current `high` ≥ previous `high`
- Current `low` ≤ previous `low`

**Body / close vs previous body**

- Previous body top = `max(open[1], close[1])`, bottom = `min(open[1], close[1])`
- **Bullish engulfing:** `close >` previous body top  
- **Bearish engulfing:** `close <` previous body bottom  

**Wick size (current bar)**

- `body = abs(close - open) > 0`
- Upper wick `< body`, lower wick `< body`

## Filters (all optional except where noted)

| Group | Default | Behavior |
|--------|---------|----------|
| **ENGULFING — Forex pip mode** | On | If on, 1 pip = `syminfo.mintick × 10`; if off, 1 pip = `syminfo.mintick` (used for min body in pips). |
| **ENGULFING — Minimum body (pips)** | **Off** | When enabled, signal body must be ≥ `min_candle_pips` pips. |
| **EMA FILTER** | **Off** | When enabled: longs require fast EMA > slow; shorts require fast < slow. EMAs are plotted on the strategy. |
| **ATR FILTER** | On | Pass when `(high - low) ≤ ATR(atr_len) × atr_multiplier`. |
| **TREND BIAS** | On | Uses `ta.pivothigh` / `ta.pivotlow` with **Bias pivot length**; keeps last **two** swing highs and **two** swing lows. Bull bias: higher high **and** higher low. Bear bias: lower high **and** lower low. |
| **SESSION FILTER** | On | Blocks new entries when bar time (UTC) is **22:45–23:15** inclusive. |

## `bars_ok` (warm-up gate)

The minimum-bar / ATR-ready gate runs **only** when **all three** are enabled:

- EMA filter  
- ATR filter  
- Trend bias filter  

If any of those is off, `bars_ok` does not block signals (no combined warm-up requirement).

## Execution (strategy)

- **`calc_on_every_tick = false`**, **`process_orders_on_close = true`** — intended to reduce repainting vs tick-by-tick intrabar logic.
- **Pyramiding:** 0 (one position direction at a time).
- **Stop loss:** long = signal bar **low**; short = signal bar **high** (same “wick” stop distance as sizing).
- **Take profit:** `tp_r_multiple` × stop distance from entry (default **3** → 1:3 reward vs that stop).
- **Position size:** `risk_usd / (stop_distance × syminfo.pointvalue)` when valid; **fixed risk per trade**, not martingale.
- **`initial_capital`** in script is very large for reporting convenience; adjust in Strategy Tester properties if you prefer.

The indicator **does not** send orders. It draws signals and optional SL/TP segments; set **TP line (R multiple)** to match the strategy’s **Take-profit (R multiple)** if you want lines to align.

## Indicator extras

- **DISPLAY:** SL/TP dashed lines (length in bars), filter table on/off.
- **Boxes / labels:** capped (oldest removed) so object limits are not exceeded.
- **Table:** session, bars OK (N/A when the three core filters are not all on), trend, EMA, ATR, min body (OFF when disabled).

## Keeping strategy and indicator in sync

The strategy and indicator are **separate** scripts: each has its **own** settings. They do **not** share inputs automatically.

To match behavior:

- Use the **same** symbol, timeframe, and chart type.
- Copy the same values for: Forex pip mode, min body toggle + pips, EMA toggle + lengths, ATR toggle + length + multiplier, trend toggle + pivot length, session toggle.
- Set indicator **TP line (R multiple)** = strategy **Take-profit (R multiple)**.

## Disclaimers

- Past performance does not guarantee future results.
- `syminfo.pointvalue` and contract specs vary by symbol; verify position size and P&L on your market.
- This is educational code; not financial advice.

## License

If you add a license file to the repo, describe usage there. Otherwise, treat usage as governed by your own terms on GitHub.

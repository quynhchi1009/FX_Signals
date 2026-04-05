# TradingView Pine Scripts

Pine Script projects for [TradingView](https://www.tradingview.com/): session-based and price-action tools. Most active scripts use **Pine v6**; a few utilities remain on **v5** (see each folder).

Repository: [github.com/quynhchi1009/FX_Signals](https://github.com/quynhchi1009/FX_Signals)

## Layout

| Folder | Contents |
|--------|----------|
| [`engulfing/`](engulfing/) | Engulfing candle **strategy** and **indicator** (v6) |
| [`casper_smc/`](casper_smc/) | Casper SMC 15m session range + FVG **strategy** and **indicator** (v6) |
| [`first_candle_value/`](first_candle_value/) | First NY-session **M15** opening range: **strategy** + **indicator** (v6) |
| [`monday_range/`](monday_range/) | **Monday range** ICT/SMC **strategy** (v5), daily chart |
| [`trend_filters_dashboard/`](trend_filters_dashboard/) | SMC/ICT **filters dashboard** **indicator** (v5) |

---

## Engulfing (`engulfing/`)

| File | Type | Purpose |
|------|------|---------|
| [`engulfing/engulfing_candle_strategy.pine`](engulfing/engulfing_candle_strategy.pine) | Strategy | Backtesting, Strategy Tester, entries/exits with stop + limit |
| [`engulfing/engulfing_candle_indicator.pine`](engulfing/engulfing_candle_indicator.pine) | Indicator | Arrows, bar colors, boxes, optional SL/TP lines, filter dashboard |

### Requirements

- [TradingView](https://www.tradingview.com/pricing/?share_your_love=quynhchii1009) account
- Pine **version 6** (for `active=` on inputs so dependent settings gray out when a filter is off)
- **Regular candlestick** chart recommended (not Heikin Ashi) so OHLC matches typical execution and backtests

### Quick start

1. Open **Pine Editor** on a chart.
2. **New** → paste [`engulfing/engulfing_candle_strategy.pine`](engulfing/engulfing_candle_strategy.pine) → **Save** → **Add to chart** (strategy).
3. Optionally add **indicator**: paste [`engulfing/engulfing_candle_indicator.pine`](engulfing/engulfing_candle_indicator.pine) → add to the same chart.
4. Open **Settings** on each script and align inputs if you want signals to match (see [Keeping strategy and indicator in sync](#keeping-strategy-and-indicator-in-sync)).

### Engulfing definition

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

### Filters (all optional except where noted)

| Group | Default | Behavior |
|--------|---------|----------|
| **ENGULFING — Forex pip mode** | On | If on, 1 pip = `syminfo.mintick × 10`; if off, 1 pip = `syminfo.mintick` (used for min body in pips). |
| **ENGULFING — Minimum body (pips)** | **Off** | When enabled, signal body must be ≥ `min_candle_pips` pips. |
| **EMA FILTER** | **Off** | When enabled: longs require fast EMA > slow; shorts require fast < slow. EMAs are plotted on the strategy. |
| **ATR FILTER** | On | Pass when `(high - low) ≤ ATR(atr_len) × atr_multiplier`. |
| **TREND BIAS** | On | Uses `ta.pivothigh` / `ta.pivotlow` with **Bias pivot length**; keeps last **two** swing highs and **two** swing lows. Bull bias: higher high **and** higher low. Bear bias: lower high **and** lower low. |
| **SESSION FILTER** | On | Blocks new entries when bar time (UTC) is **22:45–23:15** inclusive. |

### `bars_ok` (warm-up gate)

The minimum-bar / ATR-ready gate runs **only** when **all three** are enabled:

- EMA filter  
- ATR filter  
- Trend bias filter  

If any of those is off, `bars_ok` does not block signals (no combined warm-up requirement).

### Execution (strategy)

- **`calc_on_every_tick = false`**, **`process_orders_on_close = true`** — intended to reduce repainting vs tick-by-tick intrabar logic.
- **Pyramiding:** 0 (one position direction at a time).
- **Stop loss:** long = signal bar **low**; short = signal bar **high** (same “wick” stop distance as sizing).
- **Take profit:** `tp_r_multiple` × stop distance from entry (default **3** → 1:3 reward vs that stop).
- **Position size:** `risk_usd / (stop_distance × syminfo.pointvalue)` when valid; **fixed risk per trade**, not martingale.
- **`initial_capital`** in script is very large for reporting convenience; adjust in Strategy Tester properties if you prefer.

The indicator **does not** send orders. It draws signals and optional SL/TP segments; set **TP line (R multiple)** to match the strategy’s **Take-profit (R multiple)** if you want lines to align.

### Indicator extras

- **DISPLAY:** SL/TP dashed lines (length in bars), filter table on/off.
- **Boxes / labels:** capped (oldest removed) so object limits are not exceeded.
- **Table:** session, bars OK (N/A when the three core filters are not all on), trend, EMA, ATR, min body (OFF when disabled).

### Keeping strategy and indicator in sync

The strategy and indicator are **separate** scripts: each has its **own** settings. They do **not** share inputs automatically.

To match behavior:

- Use the **same** symbol, timeframe, and chart type.
- Copy the same values for: Forex pip mode, min body toggle + pips, EMA toggle + lengths, ATR toggle + length + multiplier, trend toggle + pivot length, session toggle.
- Set indicator **TP line (R multiple)** = strategy **Take-profit (R multiple)**.

---

## Casper SMC (`casper_smc/`)

NY session **9:30–12:00** logic: 09:30–09:45 range aggregation (no `request.security()`), FVG-style setups, limit entries. Use on **M1 / M5 / M15** only (script warns if higher).

| File | Purpose |
|------|---------|
| [`casper_smc/casper_smc_strategy.pine`](casper_smc/casper_smc_strategy.pine) | Backtest: limit entry, SL/TP, one attempt per day, cancel after 12:00 NY |
| [`casper_smc/casper_smc_indicator.pine`](casper_smc/casper_smc_indicator.pine) | Overlay: range lines (to 12:00 NY), FVG box, optional entry/SL/TP segments |

### Phone / MT5 workflow (alerts)

The indicator fires a TradingView **`alert()`** when a **full** BUY or SELL setup is confirmed **on bar close** (same rules as the on-chart “BUY SETUP / SELL SETUP” label). It does **not** use the `plotshape` “C1 bull” crossing trick—that was only marking candle 1 of three and is a poor alert trigger.

1. Add **`casper_smc_indicator.pine`** to the chart (same symbol and timeframe you trade).
2. In **indicator settings → Alerts**, leave **“Alert on BUY/SELL setup”** on (default).
3. Open **Create alert**, set **Condition** to this indicator and choose **“Any alert() function call”** (wording may vary slightly by app version).
4. Under **Notifications**, enable **App** (and optionally Sound) so your phone gets a push.
5. The notification body includes **Limit / SL / TP** text so you can place the limit on **MetaTrader 5** manually; always double-check prices against the chart and your broker’s symbol specs (US100 vs NAS100, digits, etc.).

**Note:** TradingView alerts run on TradingView’s servers when the condition is met; you do not need to keep the chart open 24/7, but you need a working alert subscription and notification permissions on your device.

---

## First Candle Value (`first_candle_value/`)

Opening **high/low** of the **first 15-minute bar** of your configured **New York session** (default cash-style window), with optional breakout/reversal-style logic and rich **strategy** exits (stops, R multiples, partials, trailing).

| File | Type | Notes |
|------|------|--------|
| [`first_candle_value/FirstCandleValue_Strategy.pine`](first_candle_value/FirstCandleValue_Strategy.pine) | Strategy | **Backtest on M15.** Dotted opening H/L lines visible on **M1 / M5 / M15** through session end; above M15 a table + tint explains to switch timeframe. |
| [`first_candle_value/FirstCandleValue_Indicator.pine`](first_candle_value/FirstCandleValue_Indicator.pine) | Indicator | **Value area** (~70% volume via LTF histogram proxy), VAH/VAL, signals, alerts; **≤ M15** intraday charts for full logic + timeframe warning above M15. |

- Pine **v6**; `process_orders_on_close = true` on the strategy for bar-close style fills.
- Strategy `strategy()` declaration does **not** use `max_tables_count` (indicator-only parameter in Pine).

---

## Monday range (`monday_range/`)

| File | Purpose |
|------|---------|
| [`monday_range/monday_range_strategy.pine`](monday_range/monday_range_strategy.pine) | Weekly Monday range MSS-style **strategy** on **1D**; one trade per week option, optional Friday flatten. |

- Pine **v5**.

---

## Trend filters dashboard (`trend_filters_dashboard/`)

| File | Purpose |
|------|---------|
| [`trend_filters_dashboard/trend_filters_dashboard_indicator.pine`](trend_filters_dashboard/trend_filters_dashboard_indicator.pine) | Overlay **table**: EMA, MACD, RSI, ADX, VWAP, Supertrend, higher-timeframe bias (configurable HTF). |

- Pine **v5**; display-only (no strategy orders).

---

## Disclaimers

- Past performance does not guarantee future results.
- `syminfo.pointvalue` and contract specs vary by symbol; verify position size and P&L on your market.
- This is educational code; not financial advice.

## License

If you add a license file to the repo, describe usage there. Otherwise, treat usage as governed by your own terms on GitHub.

# Dip Hunter — Tiny Bot for "Beat Buy-and-Hold on Solana"

A tiny mean-reversion paper-trading bot for make.fun's `bars-replay/v1`
harness (hourly Solana bars). It buys the dip and sells the bounce.

## Strategy

For each of the 7 symbols (FARTCOIN, JITOSOL, POPCAT, RAY, SOL, WETH, WIF),
on every hourly bar:

- Track a **24-bar EMA** of the close (24 bars = one day).
- **Buy** when the close is at least **2% below** that EMA — the asset is
  oversold against its own one-day trend.
- **Sell** the whole position when it is worth **+3% over cost** (take profit)
  or **−3% under cost** (stop loss), measured at the close.
- Stake is **20% of equity** per position (5 slots), under the 25% weight cap.
- Long only, no leverage, no shorts. Sells are sent before buys; a sell's
  proceeds never fund a buy of the same reply.

## Why this shape

The 136-bar dev window trends down — buy-and-hold loses about 8.4% on it.
This rule banks the intraday bounces instead of riding the trend down. A
local replay of the same bars (10 bps fee + 5 bps slippage, fills at the next
open, flatten at the end) gives about **+$694 net PnL (+6.9% on $10,000)**,
33 trades, max drawdown about 3.2%. It stays positive on 6 of 9 rolling
72-bar windows (mean about +$303), so it is not tuned to one lucky stretch.
The dip, the profit target and the stop are symmetric round numbers, and the
whole neighbourhood (dip 1.5–2.5%, TP 2–3.5%, SL 2–5%) stays profitable on
the same replay.

## Layout

- `main.ts` — the bot. The grader runs `node /bot/main.ts`.
- `vendor/makefun-bars.ts` — byte-identical copy of the kit's
  `sdk/makefun-bars.ts` (the bars-replay/v1 line-protocol shim).
- `.makefun/owner.json` — the payout owner's wallet.

## Grade locally

From the unpacked competition kit:

```sh
node cli/makefun.mjs grade-local --kit . --bot /path/to/this/repo --driver process
```

(`--driver docker` is the server's sandbox; `process` runs the Node bot on
this machine, unisolated.)

## Determinism

Pure float64 arithmetic over the bars the harness sends — no clock, no
environment, no randomness — so the two grading executions produce identical
action logs.

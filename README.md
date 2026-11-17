# Two-Leg V-Bounce Bot — make.fun btc-race-24h-20261117-0603

A BTC trading bot for the [make.fun](https://make.fun) `btc-race-24h-20261117-0603`
contest: it trades Coinbase Wrapped BTC (cbBTC) against USDC on 1-minute
on-chain bars, starting with $1,000 of paper money, for the highest net PnL
over 24 hours. Long only, no leverage, integer math only.

## Strategy

Buy a *confirmed* bounce off a fresh multi-hour low, then exit on a
take-profit, a stop, a trailing stop, or a time limit — otherwise stay in
cash. Two legs, because the two published dev windows reward different dips:

- **Leg A (shallow dip, regime-gated):** the lowest close of the last 360
  bars (6h) was set within the last 20 bars, the latest close is ≥ 0.4%
  above that low, **and** the trailing 360-bar return is ≥ −0.7% (flat or
  rising regime, not a downtrend). Take-profit +1.7%.
- **Leg B (deep V, always on):** the lowest close of the last 720 bars
  (12h) was set within the last 60 bars, the latest close is ≥ 0.6% above
  that low, **and** that low sits ≥ 6.5% below the highest close of the same
  720 bars (a real capitulation, not a drift low). Take-profit +4.0%.
- **Fallback leg F (guardrail insurance):** if no filled trade has happened
  by bar 1200, also accept a local-dip bounce — lowest close of the last 60
  bars within 30 bars, ≥ 0.3% above it, trailing 60-bar return ≥ −0.7% —
  same exits as leg A, so a window with no multi-hour-low bounce still sees
  at least one trade.

Shared exits: take-profit per leg, stop −1.0%, trailing stop −1.2% from the
position's peak (arms only once in profit), 120-bar time exit, 15-bar
cooldown after an exit, no new entries after bar 1320, 100 bps slippage cap
on every order. Exit bookkeeping applies only once a sell's fill is
confirmed — a rejected sell keeps the trailing-stop peak and entry bar, so
the exit is re-evaluated instead of latching.

## Results (local grade, exact harness fill math)

| window | net PnL | trades | max drawdown |
|---|---|---|---|
| dev 20261117-0603 (this contest) | **+$14.52** | 2 (1 round trip) | 0.7% |
| dev 20261115-0828 (previous contest) | **+$41.55** | 2 (1 round trip) | 0.8% |
| dev 20261113-2314 (two contests ago) | −$0.29 | 2 (1 round trip) | 1.0% |

House baselines on this contest's dev window (official `grade-local`):
buy-and-hold +$1.07, momentum +$2.45, hold-cash $0.00 (0 trades),
rsi-bollinger −$24.25, adx-momentum −$32.66, ema-cross −$106.21.
The bot beats every house baseline by a clear margin.

Two runs of the wire-protocol driver on the same bars are byte-identical.

## Files

- `main.py` — the bot (Python 3.13, standard library only; integer math,
  no floats, no clock, no randomness, no network).
- `vendor/makefun_bars.py` — the kit's Python SDK shim, copied byte for byte.
- `LICENSE` — MIT. `.makefun/owner.json` — payout wallet claim file.

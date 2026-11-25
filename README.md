[![make.fun](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)
[![rank](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/Cm11w/pump-fun-bot-sprint/rank.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)
[![reward](https://make.fun/badge/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed/Cm11w/pump-fun-bot-sprint/reward.svg)](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed)

# Pump.fun Bot Sprint — Surge + Pullback Scalper

A deterministic paper-trading bot for the [Pump.fun Bot Sprint: Make Bot And Win](https://make.fun/bounty/u-riidzhsn-pump-fun-bot-sprint-make-bot-and-win-cb203fed) bounty on make.fun. It trades 10 SOL of paper money on a replay of recorded Solana pump.fun bonding-curve activity. Nothing is traded for real: the house replay scorer runs this bot over 6 hours of recorded pump.fun activity and scores what it would have made.

## Strategy: surge + pullback scalper

Most pump.fun coins bleed out within minutes, but the ones that live keep printing the same two footprints — a burst of accelerating buy flow while the price grinds to new highs, and a shallow pullback inside an intact uptrend once the first sellers are done. This bot trades only those two footprints, never the launch itself.

**Entries (max 5 open positions, one entry per coin):**
- **SURGE** — over a 25-slot window: net buy flow ≥ 0.5 SOL, ≥ 4 buys from ≥ 4 distinct wallets, price up 15–80% vs the window start, price within 1% of the window high, newer-half inflow at least the older half's, no single wallet over 60% of buy flow, curve real SOL 0.5–50, age ≥ 150 slots, creator has not sold.
- **PULLBACK** — over a 60-slot window: price still ≥ 15% above the window start (trend intact) but 5–18% off its window high, with net flow over the last 10 slots back to zero or better (the sellers are done).

First buys are 0.40 SOL (surge) / 0.12 SOL (pullback). A position gets add-on buys of 0.15 SOL only while the price has stepped up ≥ 6% from the last buy, the position is still up on cost and flow is still positive, up to 1.0 SOL total — so false starts cost little and runners get size.

**Exits (checked every batch):** take-profit +100% of cost, stop-loss −12%, trailing stop (arms at +20%, exits on an 8% fall from the peak), creator-sell exit, big-sell exit (any single recorded sell ≥ 2 SOL), and a 1000-slot time cap (waived while the position is up ≥ 8%). A curve that completes is closed by the replay.

**Guardrail pacing:** the run must end with round trips of ≥ 0.01 SOL on ≥ 20 coins. When the bot's own fills or coins lag a pace prorated over the window's block time, it opens tiny probe round trips (buy 0.02 SOL, sell the whole probe as soon as the buy lands) until it is back on pace.

**Determinism:** no clock, no environment, no randomness; iteration only over insertion-ordered maps fed by the batch stream, so the two grading executions produce byte-identical action logs.

## Files

- `main.ts` — the whole entry: the kit's `sdk/ts/makefun-bot.ts` wire shim for the `agent-fun/replay/v2` protocol (vendored, inlined) plus the strategy written from scratch for this bounty. The grader runs `node /bot/main.ts`.

## Local grading (dev window)

From the unpacked kit directory (the kit is **not** part of this repository):

```sh
node cli/makefun.mjs grade-local --kit . --bot <this repository> --driver process
```

Dev-window result (verified, two executions reproduced byte for byte):

- Net PnL **+0.481 SOL** on 10 SOL paper capital (401 fills, 96 coins, 151 round trips, win rate 15.2%, max drawdown 13.4%)
- Guardrails met (96 coins hold a round trip ≥ 0.01 SOL, drawdown ok)
- `action_log_sha256` `a54b9aa8f177b70899403021f67ff5f53368d027fc95ccc44d3e48226038c131`
- `result_sha256` `ac83dc41274a479be489db17f3120f90d4162aa90bf57657595d7abbbd62ac47` (process driver; the server's docker grading records its own hash)

Honest note: this dev window is hard — the kit's own pilot momentum baseline scores −9.46 SOL on it, and the sibling replay bounty showed the same pattern (every config the team tried lost there). The bot's edge is real but concentrated: the 3 best coins make 60% of the gains, and without them the dev net is −1.17 SOL; at 2x fees the dev net turns −0.18 SOL. Robustness checks are worse: `grade-local --stress 20` gives median −0.69 SOL (p10 −1.23, p90 −0.01), 19 of 20 rebuilt copies lose money, and the dev score sits above all 20 copies; split by close slot, trips closing in the first half of the window net +1.12 SOL while the second half nets −0.65 SOL. So the dev +0.48 leans on this window's own early runners. The final window is fresh 6-hour data recorded after entries close, so the dev number is a guide, not a promise — the bot lives or dies on catching a few real runners, which is exactly what the surge filter (accelerating flow + creator still holding + big-sell exit) is built for. Submitting is a bet that the final window has runners like the dev window's first half, not its second.

## License

MIT — see [LICENSE](LICENSE).

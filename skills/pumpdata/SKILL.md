---
name: pumpdata
description: "Before an alpha call or a buy on pump.fun: the base rate for the market-cap band (how often coins like this one paid), what the buyers have been (winner, bot, supply seller, new), the deployer's record with how similar launches went, and an honest replay of a pick list with fees and fills. Wallet records since day one, from every PumpSwap trade, ever. Read-only data; paid per call in USDC over x402 (the user's own wallet signs; nothing is charged without `payment`), from $0.005; yesterday's base rates free. Use when the user mentions pumpdata or pumpdata.fun, or asks for a base rate, wallet record, launch record or replay of pump.fun coins. Not affiliated with pump.fun."
---

# pumpdata: base rates, wallet records, launch records, replay

pumpdata.fun is a warehouse of every PumpSwap trade, ever, with wallet records since day one. It is
read from the chain's own ledger, and the dead coins are kept. It tells you whether coins like this one
have ever paid, what its buyers have been, what its deployer has done, and what a list of picks would
really have made. The tools read data; a paid tool settles a USDC payment signed by the user's own x402
wallet, and nothing is charged without `payment`. Nothing here signs a trade. pumpdata is not affiliated
with pump.fun.

The `pumpdata` MCP server bundled with this plugin (`https://pumpdata.fun/mcp`, Streamable HTTP) exposes
these calls as tools.

## Free, before any payment

- `https://pumpdata.fun/public/baserate.json`: yesterday's base rate for every market-cap band,
  refreshed hourly. It is the same answer the paid call gives for that day. If you can fetch URLs, read
  it first, and pay only for another window.
- `https://pumpdata.fun/llms.txt` describes what is in the warehouse. `https://pumpdata.fun/openapi.json`
  is the contract, with the price of each call in `x-payment-info`.
- The `replay_result` tool is free.

## Tools

| tool | asks | answers | price |
|---|---|---|---|
| `base_rate` | `band` (edges snap to 5k, 10k, 15k, 20k, 28k, 40k, 60k, 100k, 250k, 1000k), `from`, `to` (a window of up to 31 days), optional `venue` (any, curve or pool) | `crossed` (coins with a crossing in the window), `reached` {2x, 5x, 10x, 20x} as shares within 3 h, `median_peak`, and the window used | $0.005 |
| `wallet_records` | `addresses` (up to 50), optional `mint` | per wallet: `class` (winner, bot, supply_seller, new, none), `pnl_sol`, `put_in`, `took_out`, `coins`, `first_seen` | $0.01 |
| `launch_record` | `mint` | the deployer's record (launches, graduations, the gap to the previous one), the bundle at creation, the insiders' share of supply, curve progress, and how launches that looked the same at this age went over their next 24 h | $0.005 |
| `replay` | `picks` (up to 500, each `mint`, `at`, `size_sol`; graduates only), optional `latency_s` (seconds from decision to fill, default 3), `bankroll_sol` (SOL at risk at once), `exit` {`take`, `trail`, `hold_s`} | a queued job `{id, state}`. Poll `replay_result` with the id. A run that fails is rerun free on the same id | $2 a run |
| `replay_result` | `id` | the job's state, and the replay once done: pick by pick and in sum, with fees, fills and the dead coins. A pick still on the bonding curve, or outside the archive, fills nothing and is counted in `skipped_not_covered` | free |

Definitions: a *crossing* is a coin's first trade with a market cap inside the band. A coin that jumps
over the band in one trade has none. The *peak* is the highest market cap, curve or pool, in the 3 h
after the crossing. A crossing needs its 3 h after, so a window ending under 3 h before the data's as-of
is cut there. USD is at the SOL price of the trade's hour. The HTTP API carries `X-Data-As-Of` (the build
an answer comes from); over MCP, quote the as-of the answer returns.

## Payment

A paid tool called without `payment` returns the x402 offer (price, payee, network) and charges
nothing. Payment is x402 v2, USDC on Solana: the user's own x402 wallet tool signs the transfer, and
you call the tool again with the resulting `PAYMENT-SIGNATURE` value as `payment`.

- **Never pay without the user's explicit go-ahead for that call and its price.** Show the offer first.
- Never ask for, read, or handle a private key or seed phrase. This plugin needs no key and no account.
- If an answer says `payment_settle_uncertain`, do not pay again blindly. If a replay was paid but
  not queued, keep the receipt: the run is owed.

## Rules

1. **Say the window.** Base rates move with the market. Always quote the band and the window the
   answer used. `from` and `to` are echoed back, clipped to coverage when needed, with a note.
2. **Say the as-of.** Quote the as-of the answer returns (`X-Data-As-Of` over HTTP) when the number matters. The archive trails the chain by hours.
3. **Dead coins count.** The shares are over every coin that crossed, including the ones that died.
   Never drop them.
4. **Never sign a trade from here.** This skill reads. Trades belong to other tools, after the read.
5. **Spend small.** Yesterday's base rate is free and a window is half a cent. Check one before
   spending $2 on a replay.
6. **Labels are rules.** A `winner` took out more than it put in over many coins. A `bot` trades like
   a machine. A `supply_seller` sold supply it was given. Report the label and the SOL behind it, not a
   verdict about anyone.
7. **Only quote numbers an answer returned.** Never estimate or fill in a figure.

## Typical uses

- **An alpha report on a coin at a given market cap.** Read the base rate for its band and window,
  free for yesterday. Then state what share of coins in that band reached 10x within 3 h: that is the
  bar the call has to beat.
- **A buy into a coin with visible buyers.** Call `wallet_records` with the buyers' addresses and the
  mint. Report the classes and the SOL behind them, then let the user decide.
- **A fresh launch.** Call `launch_record` with the mint. Report the deployer's record, the bundle,
  the insiders' share, and how similar launches went.
- **"My picks did 40x."** Call `replay` with the picks as posted and poll `replay_result`. Report
  the sum with fees, fills and the dead ones, how many came back `skipped_not_covered`, and the base
  rate for the same bands and window. If the replay is not above the base rate, say so.

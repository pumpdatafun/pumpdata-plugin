# pumpdata plugin for Grok Build

[pumpdata.fun](https://pumpdata.fun) provides base rates, wallet records since day one, launch records and
an honest replay for pump.fun coins, built from every PumpSwap trade, ever. This plugin bundles:

- **MCP server** `pumpdata`: the hosted server at `https://pumpdata.fun/mcp` (Streamable HTTP). It
  offers the tools `base_rate`, `wallet_records`, `launch_record`, `replay` and `replay_result`.
- **Skill** `pumpdata` (`skills/pumpdata/SKILL.md`): when to call each tool, how to read the answers,
  and how payment works.

pumpdata is not affiliated with pump.fun.

## Pricing

Read-only data, paid per call in USDC on Solana over x402: a paid tool settles a USDC payment signed by
the user's own x402 wallet, and nothing is charged without `payment`. Prices are from
[openapi.json](https://pumpdata.fun/openapi.json) `x-payment-info`:

| tool | price |
|---|---|
| `base_rate` | $0.005 |
| `wallet_records` | $0.01 |
| `launch_record` | $0.005 |
| `replay` | $2 a run |
| `replay_result` | free |

Yesterday's base rate for every band is free at
[`/public/baserate.json`](https://pumpdata.fun/public/baserate.json). A paid tool called without
`payment` returns the offer and charges nothing.

## Network endpoints and credentials

- **Endpoints:** `https://pumpdata.fun/mcp` (the MCP server). The skill may also point the agent at
  `https://pumpdata.fun/public/baserate.json`, `https://pumpdata.fun/llms.txt` and
  `https://pumpdata.fun/openapi.json`. Nothing else.
- **Credentials:** none. No API key, no account, no OAuth. Paying for a call needs the user's own x402
  wallet tool, which is separate from this plugin. This plugin never reads, stores or asks for keys.
- **Local code:** none. There are no hooks, no commands, no scripts and no binaries. The plugin
  consists of a remote MCP config, one skill file and a manifest.

## More

- Docs: https://pumpdata.fun/llms.txt
- Contract: https://pumpdata.fun/openapi.json
- Examples (curl, node, python, agent frameworks): https://github.com/pumpdatafun/pumpdata-examples

## License

MIT. See [LICENSE](LICENSE).

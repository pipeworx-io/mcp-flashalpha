# mcp-flashalpha

FlashAlpha MCP — wraps the FlashAlpha options-exposure analytics API (lab.flashalpha.com).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1864+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

| Tool | Description |
|------|-------------|
| `flashalpha_gex` | Get options gamma exposure (GEX) by strike for an underlying — net GEX, gamma flip, and per-strike call/put gamma. Useful for reading dealer positioning and likely support/resistance. Example: flashalpha_gex({ symbol: "SPY", _apiKey: "your-key" }) |
| `flashalpha_levels` | Get key options-derived levels for an underlying — gamma flip level, call wall, and put wall. These act as magnet / support / resistance levels driven by dealer gamma. Example: flashalpha_levels({ symbol: "SPY", _apiKey: "your-key" }) |
| `flashalpha_iv` | Get an implied-volatility summary for a stock — IV rank and IV percentile (plus current/underlying IV where available). Use to gauge whether options are relatively cheap or expensive. Example: flashalpha_iv({ symbol: "AAPL", _apiKey: "your-key" }) |
| `flashalpha_greeks` | Black-Scholes (BSM) option greeks calculator — pass spot, strike, days-to-expiry, volatility and option type to get delta, gamma, theta, vega (and higher-order greeks where returned). Example: flashalpha_greeks({ spot: 100, strike: 105, dte: 30, vol: 0.25, type: "call", _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "flashalpha": {
      "url": "https://gateway.pipeworx.io/flashalpha/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/flashalpha/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1864+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/flashalpha_gex`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-flashalpha` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/flashalpha/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "flashalpha": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-flashalpha"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Flashalpha data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT

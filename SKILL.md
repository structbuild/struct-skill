---
name: struct-sdk
description: Integrate @structbuild/sdk for prediction market data. Use when building apps with Struct API, accessing markets/events/traders data, setting up real-time WebSocket feeds, or working with Polymarket data.
---

# Struct SDK Integration

TypeScript SDK for prediction market data via [api.struct.to](https://api.struct.to).

## Installation

```bash
npm install @structbuild/sdk
# or: pnpm add @structbuild/sdk
# or: bun add @structbuild/sdk
```

## Quick Start

```typescript
import { StructClient } from "@structbuild/sdk";

const client = new StructClient({
  apiKey: process.env.STRUCT_API_KEY!,
});

const markets = await client.markets.getMarkets({ limit: 10 });
console.log(markets.data);
```

## Client Configuration

```typescript
const client = new StructClient({
  apiKey: "your-api-key",
  venue: "polymarket",           // default venue
  timeout: 10000,                // request timeout (ms)
  retry: {
    maxRetries: 3,
    initialDelay: 500,
  },
  onRequest: (info) => console.log(`${info.method} ${info.url}`),
  onResponse: (info) => console.log(`${info.status} in ${info.duration}ms`),
});
```

## API Namespaces

The client exposes 9 namespaces:

| Namespace | Purpose |
|-----------|---------|
| `markets` | Market data, trades, candlesticks, metrics |
| `events` | Event listings and metrics |
| `trader` | Portfolio, positions, PnL, trade history |
| `holders` | Position holders and holder history |
| `series` | Market series data |
| `search` | Search markets/events |
| `tags` | Tag listings |
| `bonds` | Bond data |
| `scoring` | Trader scoring |

## Common Patterns

### Fetch Markets

```typescript
const markets = await client.markets.getMarkets({ limit: 10 });
const market = await client.markets.getMarket({ conditionId: "0x..." });
const marketBySlug = await client.markets.getMarketBySlug({ slug: "will-x-happen" });
```

### Fetch Events

```typescript
const events = await client.events.getEvents({ limit: 10 });
const event = await client.events.getEvent({ id: "123" });
const eventBySlug = await client.events.getEventBySlug({ slug: "us-election" });
```

### Trader Portfolio

```typescript
const portfolio = await client.trader.getPortfolio({ address: "0x..." });
const positions = await client.trader.getPortfolioPositions({ address: "0x..." });
const pnl = await client.trader.getTraderPnl({ address: "0x..." });
```

### Market Analytics

```typescript
const trades = await client.markets.getTrades({ conditionId: "0x..." });
const candles = await client.markets.getCandlestick({
  conditionId: "0x...",
  interval: "1h",
});
const metrics = await client.markets.getMarketMetrics({ conditionId: "0x..." });
```

### Holders Data

```typescript
const holders = await client.holders.getMarketHolders({ conditionId: "0x..." });
const history = await client.holders.getMarketHoldersHistory({ conditionId: "0x..." });
```

## Pagination

Use the `paginate` helper for iterating all results:

```typescript
import { StructClient, paginate } from "@structbuild/sdk";

for await (const market of paginate(
  (params) => client.markets.getMarkets(params),
  { limit: 100 },
)) {
  console.log(market.slug);
}
```

## Error Handling

```typescript
import { HttpError, TimeoutError, NetworkError } from "@structbuild/sdk";

try {
  await client.markets.getMarket({ conditionId: "0x..." });
} catch (error) {
  if (error instanceof HttpError) {
    console.log(`HTTP ${error.status}: ${error.body}`);
  } else if (error instanceof TimeoutError) {
    console.log("Request timed out");
  } else if (error instanceof NetworkError) {
    console.log("Network error");
  }
}
```

## WebSocket (Real-time)

For detailed WebSocket usage, see [websocket.md](websocket.md).

```typescript
import { StructWebSocket } from "@structbuild/sdk";

const ws = new StructWebSocket({
  apiKey: process.env.STRUCT_API_KEY!,
});

ws.on("connected", () => console.log("Connected"));
ws.on("market_trade", (trade) => console.log("Trade:", trade));

ws.connect();
ws.subscribeMarket("0x...condition_id");
```

## Additional Resources

- [WebSocket API](websocket.md) - Real-time trade streaming
- [API Reference](api-reference.md) - Complete method reference

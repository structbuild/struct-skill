---
name: struct-sdk
description: Integrate @structbuild/sdk for prediction market data. Use when building apps with Struct API, accessing markets/events/traders data, or working with Polymarket data.
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
	venue: "polymarket",
	baseUrl: "https://api.struct.to/v1",
	timeout: 30000,
	headers: { "x-custom": "value" },
	retry: {
		maxRetries: 3,
		initialDelayMs: 1000,
		maxDelayMs: 30000,
	},
	onRequest: (info) => console.log(`${info.method} ${info.url}`),
	onResponse: (info) => console.log(`${info.status} in ${info.durationMs}ms`),
});
```

## Response Format

All methods return `HttpResponse<T>`:

```typescript
interface HttpResponse<T> {
	data: T;
	message: string | null;
	success: boolean;
	info?: {
		version: string;
		credits_consumed: number;
	};
	pagination?: {
		has_more: boolean;
		pagination_key: string | number | null;
	};
}
```

## API Namespaces

The client exposes 10 namespaces. Every method accepts an optional `venue` parameter as its last argument.

| Namespace  | Purpose                                                           |
| ---------- | ----------------------------------------------------------------- |
| `markets`  | Market data, trades, candlesticks, metrics, volume charts         |
| `events`   | Event listings, charts, metrics, outcomes                         |
| `trader`   | PnL, profiles, trade history, volume charts                       |
| `holders`  | Market/position holders and holder history                        |
| `series`   | Market series listings and outcomes                               |
| `search`   | Search markets and events                                         |
| `tags`     | Tag listings and lookup                                           |
| `bonds`    | Bond market data                                                  |
| `assets`   | Asset price history                                               |
| `webhooks` | Webhook management (create, update, delete, test)                 |

## Markets

```typescript
const markets = await client.markets.getMarkets({ limit: 10, sort_by: "volume" });
const market = await client.markets.getMarket({ conditionId: "0x..." });
const marketBySlug = await client.markets.getMarketBySlug({ slug: "will-x-happen" });
const chart = await client.markets.getMarketChart({ conditionId: "0x..." });

const trades = await client.markets.getTrades({ conditionId: "0x...", limit: 50 });
const candles = await client.markets.getCandlestick({ conditionId: "0x...", interval: "1h" });
const posCandles = await client.markets.getPositionCandlestick({ positionId: "pos_123", interval: "1h" });

const metrics = await client.markets.getMarketMetrics({ conditionId: "0x..." });
const posMetrics = await client.markets.getPositionMetrics({ positionId: "pos_123" });

const marketVolume = await client.markets.getMarketVolumeChart({ conditionId: "0x..." });
const posVolume = await client.markets.getPositionVolumeChart({ positionId: "pos_123" });

const priceJumps = await client.markets.getPriceJumps({ limit: 20 });
```

## Events

```typescript
const events = await client.events.getEvents({ limit: 10 });
const event = await client.events.getEvent({ identifier: "123" });
const eventBySlug = await client.events.getEventBySlug({ slug: "us-election" });
const chart = await client.events.getEventChart({ eventId: "123" });
const metrics = await client.events.getEventMetrics({ eventId: "123" });
const outcomes = await client.events.getEventOutcomes({ eventId: "123" });
```

## Trader

```typescript
const trades = await client.trader.getTraderTrades({ address: "0x...", limit: 50 });

const profile = await client.trader.getTraderProfile({ address: "0x..." });
const profiles = await client.trader.getTraderProfilesBatch({ addresses: "0xabc,0xdef" });

const pnl = await client.trader.getTraderPnl({ address: "0x..." });
const outcomePnl = await client.trader.getTraderOutcomePnl({ address: "0x..." });
const marketPnl = await client.trader.getTraderMarketPnl({ address: "0x..." });
const eventPnl = await client.trader.getTraderEventPnl({ address: "0x..." });
const pnlCandles = await client.trader.getTraderPnlCandles({
	address: "0x...",
	resolution: "1d",
	limit: 30,
});

const volumeChart = await client.trader.getTraderVolumeChart({ address: "0x..." });
const globalPnl = await client.trader.getGlobalPnl({ limit: 100 });
```

## Holders

```typescript
const marketHolders = await client.holders.getMarketHolders({ conditionId: "0x..." });
const positionHolders = await client.holders.getPositionHolders({ positionId: "pos_123" });

const marketHistory = await client.holders.getMarketHoldersHistory({ conditionId: "0x..." });
const positionHistory = await client.holders.getPositionHoldersHistory({ positionId: "pos_123" });
```

## Series

```typescript
const allSeries = await client.series.getSeriesList();
const outcomes = await client.series.getSeriesOutcomes({ seriesId: "series-id" });
```

## Search

```typescript
const results = await client.search.search({ query: "election" });
```

## Tags

```typescript
const tags = await client.tags.getTags();
const tag = await client.tags.getTag({ identifier: "politics" });
```

## Bonds

```typescript
const bonds = await client.bonds.getBonds({ limit: 50 });
```

## Assets

```typescript
const history = await client.assets.getAssetHistory({ asset: "BTC", interval: "1d" });
```

## Webhooks

```typescript
const webhooks = await client.webhooks.list();
const webhook = await client.webhooks.create({
	url: "https://example.com/webhook",
	events: ["market_trade"],
});
const detail = await client.webhooks.getWebhook({ webhookId: "wh_123" });
await client.webhooks.update({ webhookId: "wh_123", url: "https://example.com/new" });
await client.webhooks.deleteWebhook({ webhookId: "wh_123" });
await client.webhooks.test({ webhookId: "wh_123" });
await client.webhooks.rotateSecret({ webhookId: "wh_123" });
const events = await client.webhooks.listEvents();
```

## Pagination

Use the `paginate` helper for iterating all results:

```typescript
import { StructClient, paginate } from "@structbuild/sdk";

for await (const market of paginate((params) => client.markets.getMarkets(params), { limit: 100 })) {
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
		console.log(`HTTP ${error.status}: ${error.statusText}`, error.body);
	} else if (error instanceof TimeoutError) {
		console.log("Request timed out");
	} else if (error instanceof NetworkError) {
		console.log("Network error:", error.message);
	}
}
```

## Additional Resources

- [API Reference](api-reference.md) - Complete method reference
- [Webhooks](webhooks.md) - Events, filters, payload format, and examples
- [Guides](guides.md) - Market screening, trader analytics, and more

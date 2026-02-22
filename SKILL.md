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
	timeout: 10000,
	headers: { "x-custom": "value" },
	retry: {
		maxRetries: 3,
		initialDelayMs: 500,
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
}
```

## API Namespaces

The client exposes 9 namespaces. Every method accepts an optional `venue` parameter as its last argument.

| Namespace | Purpose                                                           |
| --------- | ----------------------------------------------------------------- |
| `markets` | Market data, trades, candlesticks, metrics, volume charts         |
| `events`  | Event listings and metrics                                        |
| `trader`  | Portfolio, positions, PnL, profiles, trade history, volume charts |
| `holders` | Market/event/position holders and holder history                  |
| `series`  | Market series listings and details                                |
| `search`  | Search markets and events                                         |
| `scoring` | Trader scores, smart money / insider / bot leaderboards           |
| `tags`    | Tag listings and lookup                                           |
| `bonds`   | Bond market data                                                  |

## Markets

```typescript
const markets = await client.markets.getMarkets({ limit: 10, sort_by: "volume" });
const market = await client.markets.getMarket({ conditionId: "0x..." });
const marketBySlug = await client.markets.getMarketBySlug({ slug: "will-x-happen" });

const trades = await client.markets.getTrades({ conditionId: "0x...", limit: 50 });
const candles = await client.markets.getCandlestick({ conditionId: "0x...", interval: "1h" });
const posCandles = await client.markets.getPositionCandlestick({ positionId: "pos_123", interval: "1h" });

const metrics = await client.markets.getMarketMetrics({ conditionId: "0x..." });
const posMetrics = await client.markets.getPositionMetrics({ positionId: "pos_123" });

const marketVolume = await client.markets.getMarketVolumeChart({ conditionId: "0x..." });
const posVolume = await client.markets.getPositionVolumeChart({ positionId: "pos_123" });
```

## Events

```typescript
const events = await client.events.getEvents({ limit: 10 });
const event = await client.events.getEvent({ id: "123", include_tags: true, include_markets: true });
const eventBySlug = await client.events.getEventBySlug({ slug: "us-election", include_markets: true });
const eventMetrics = await client.events.getEventMetrics({ eventId: "123" });
```

## Trader

```typescript
const portfolio = await client.trader.getPortfolio({ address: "0x...", timeframe: "30d" });
const positions = await client.trader.getPortfolioPositions({ address: "0x...", limit: 50 });
const trades = await client.trader.getTraderTrades({ address: "0x...", limit: 50 });

const profile = await client.trader.getTraderProfile({ address: "0x..." });
const profiles = await client.trader.getTraderProfilesBatch({ addresses: "0xabc,0xdef" });

const pnl = await client.trader.getTraderPnl({ address: "0x...", timeframe: "30d" });
const positionPnl = await client.trader.getTraderPositionPnl({
	address: "0x...",
	timeframe: "30d",
	sort_by: "pnl_usd",
	sort_direction: "desc",
	limit: 20,
});
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
const eventHolders = await client.holders.getEventHolders({ eventSlug: "us-election" });
const positionHolders = await client.holders.getPositionHolders({ positionId: "pos_123" });

const marketHistory = await client.holders.getMarketHoldersHistory({ conditionId: "0x...", hours: 24 });
const eventHistory = await client.holders.getEventHoldersHistory({ eventSlug: "us-election", hours: 48 });
const positionHistory = await client.holders.getPositionHoldersHistory({ positionId: "pos_123" });
```

## Scoring

```typescript
const score = await client.scoring.getTraderScore({ address: "0x..." });
const smartMoney = await client.scoring.getSmartMoneyLeaderboard({ limit: 50 });
const insiders = await client.scoring.getInsiderLeaderboard({ limit: 50 });
const bots = await client.scoring.getBots({ limit: 50 });
```

## Series

```typescript
const allSeries = await client.series.getSeriesList();
const detail = await client.series.getSeriesDetail({ identifier: "series-slug" });
const seriesEvents = await client.series.getSeriesEvents({ identifier: "series-slug" });
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

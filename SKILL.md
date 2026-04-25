---
name: struct-sdk
description: Integrate @structbuild/sdk for prediction market data. Use when building apps with Struct API, accessing markets/events/traders data, real-time WebSocket streams, or working with Polymarket data.
---

# Struct SDK Integration

TypeScript SDK for prediction market data via [api.struct.to](https://api.struct.to). REST + WebSocket, full type safety.

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

## Auth Modes

Two API key flavors:

- **`sk_*`** — secret server key. Full access. Never ship to a browser.
- **`pk_jwt_*`** — public JWT key. Frontend-safe. Useless without a valid end-user JWT from your configured auth provider (Privy, Auth0, Google, Turnkey, etc.).

```typescript
const client = new StructClient({
	apiKey: "pk_jwt_...",
	jwt: userAccessToken,
});

const ws = new StructWebSocket({
	apiKey: "pk_jwt_...",
	getJwt: () => userAccessToken,
});
```

Use `getJwt` when the token can rotate while a socket stays alive — reconnects rebuild the URL with a fresh token.

## Response Format

Every REST method returns `Promise<HttpResponse<T>>`:

```typescript
interface HttpResponse<T> {
	data: T;
	message: string | null;
	success: boolean;
	info?: ApiResponseInfo;
	pagination?: PaginationInfo;
}

interface ApiResponseInfo {
	version: string;
	credits_consumed: number;
	time_taken_ms?: number;
	compute_time_ms?: number;
}

interface PaginationInfo {
	has_more: boolean;
	pagination_key: string | number | null;
}
```

## REST Namespaces

12 namespaces. Every method accepts an optional `venue` parameter as its last argument.

| Namespace   | Purpose                                                                  |
| ----------- | ------------------------------------------------------------------------ |
| `markets`   | Markets, trades, candlesticks, metrics, volume, top traders, oracle      |
| `events`    | Event listings, charts, metrics, outcomes, top traders                   |
| `trader`    | PnL, profiles, trades, volume, leaderboard                               |
| `holders`   | Market/position holders and holder history                               |
| `series`    | Market series listings and outcomes                                      |
| `search`    | Search markets and events                                                |
| `tags`      | Tag listings, search, and lookup                                         |
| `bonds`     | Bond market data                                                         |
| `assets`    | Asset price history                                                      |
| `orderBook` | Order book snapshots, spread history, liquidity depth                    |
| `analytics` | Time-bucketed deltas, percentage changes, and timeseries across scopes   |
| `webhooks`  | Webhook management (create, update, delete, test, rotate-secret)         |

## Markets

```typescript
const markets = await client.markets.getMarkets({ limit: 10, sort_by: "volume" });
const market = await client.markets.getMarket({ conditionId: "0x..." });
const marketBySlug = await client.markets.getMarketBySlug({ marketSlug: "will-x-happen" });
const chart = await client.markets.getMarketChart({ condition_id: "0x..." });

const trades = await client.markets.getTrades({ condition_ids: "0x...", limit: 50 });
const candles = await client.markets.getCandlestick({ condition_id: "0x...", resolution: "60" });
const posCandles = await client.markets.getPositionCandlestick({ position_id: "pos_123", resolution: "1D" });

const metrics = await client.markets.getMarketMetrics({ condition_id: "0x..." });
const posMetrics = await client.markets.getPositionMetrics({ position_id: "pos_123" });

const marketVolume = await client.markets.getMarketVolumeChart({ condition_id: "0x..." });
const posVolume = await client.markets.getPositionVolumeChart({ position_id: "pos_123" });

const priceJumps = await client.markets.getPriceJumps({ limit: 20 });
const oracleEvents = await client.markets.getOracleEvents({ limit: 50 });
const marketTopTraders = await client.markets.getMarketTopTraders({ condition_id: "0x..." });
const positionTopTraders = await client.markets.getPositionTopTraders({ position_id: "pos_123" });
```

## Events

```typescript
const events = await client.events.getEvents({ limit: 10 });
const event = await client.events.getEvent({ identifier: "123" });
const eventBySlug = await client.events.getEventBySlug({ slug: "us-election" });
const chart = await client.events.getEventChart({ event_slug: "us-election", resolution: "1D" });
const metrics = await client.events.getEventMetrics({ event_slug: "us-election", timeframe: "24h" });
const outcomes = await client.events.getEventOutcomes({ event_slug: "us-election" });
const topTraders = await client.events.getEventTopTraders({ event_slug: "us-election" });
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
const pnlCandles = await client.trader.getTraderPnlCandles({ address: "0x...", resolution: "1d", limit: 30 });
const calendar = await client.trader.getTraderPnlCalendar({ address: "0x..." });

const volumeChart = await client.trader.getTraderVolumeChart({ address: "0x..." });
const globalPnl = await client.trader.getGlobalPnl({ limit: 100 });
const leaderboard = await client.trader.getLeaderboard({ timeframe: "30d", sort_by: "pnl", category: "politics" });
```

## Analytics

Time-bucketed deltas (changes within a window), percentage changes between two windows, and timeseries (multiple buckets). Each scope (`global`, `event`, `market`, `tag`, `trader`) supports the same three shapes.

```typescript
const counts = await client.analytics.getCounts();

const globalDeltas = await client.analytics.getDeltas({ resolution: "1d", count_back: 30 });
const globalChanges = await client.analytics.getChanges({ resolution: "1d" });
const globalTimeseries = await client.analytics.getTimeseries({ resolution: "1h", count_back: 24 });

const marketTimeseries = await client.analytics.getMarketTimeseries({ condition_id: "0x...", resolution: "1h" });
const eventDeltas = await client.analytics.getEventDeltas({ event_slug: "us-election", resolution: "1d" });
const tagChanges = await client.analytics.getTagChanges({ identifier: "politics" });
const traderTimeseries = await client.analytics.getTraderTimeseries({ address: "0x...", resolution: "1d" });
```

## Holders

```typescript
const marketHolders = await client.holders.getMarketHolders({ condition_id: "0x..." });
const positionHolders = await client.holders.getPositionHolders({ positionId: "pos_123" });
const marketHistory = await client.holders.getMarketHoldersHistory({ condition_id: "0x..." });
const positionHistory = await client.holders.getPositionHoldersHistory({ positionId: "pos_123" });
```

## Series, Search, Tags, Bonds, Assets

```typescript
const allSeries = await client.series.getSeriesList();
const seriesOutcomes = await client.series.getSeriesOutcomes({ series_slug: "weekly-btc" });

const results = await client.search.search({ query: "election", sort_by: "volume" });

const tags = await client.tags.getTags();
const tag = await client.tags.getTag({ identifier: "politics" });
const tagSearch = await client.tags.searchTags({ query: "elect" });

const bonds = await client.bonds.getBonds({ limit: 50 });

const history = await client.assets.getAssetHistory({ symbol: "BTC", variant: "1d" });
```

## OrderBook

```typescript
const book = await client.orderBook.getOrderBook({ asset_id: "0x..." });
const marketBook = await client.orderBook.getMarketOrderBook({ condition_id: "0x..." });
const history = await client.orderBook.getOrderBookHistory({ condition_id: "0x..." });
const spread = await client.orderBook.getSpreadHistory({ condition_id: "0x..." });
```

## Webhooks

```typescript
const webhooks = await client.webhooks.list();
const webhook = await client.webhooks.create({
	url: "https://example.com/webhook",
	event: "trader_whale_trade",
	filters: { min_usd_value: 10000 },
});
const detail = await client.webhooks.getWebhook({ webhookId: "wh_123" });
await client.webhooks.update({ webhookId: "wh_123", url: "https://example.com/new" });
await client.webhooks.deleteWebhook({ webhookId: "wh_123" });
await client.webhooks.test({ webhookId: "wh_123" });
await client.webhooks.rotateSecret({ webhookId: "wh_123" });
const events = await client.webhooks.listEvents();
```

See [webhooks.md](webhooks.md) for the full event list, filters, payload format, and signature verification.

## Real-Time WebSockets

Two separate clients:

- **`StructWebSocket`** — room-based streams (`/ws`). Subscribe to a room with filters; receive a flow of events for that room. Use for trades, order books, metrics, account balances, market/event streams.
- **`StructAlertsWebSocket`** — event-based alerts (`/ws/alerts`). Per-event subscription with the same filters as webhooks. Use as a polling-free replacement for webhook delivery during development or for in-app alerts.

```typescript
import { StructWebSocket, StructAlertsWebSocket } from "@structbuild/sdk";

const ws = new StructWebSocket({ apiKey: process.env.STRUCT_API_KEY! });
await ws.connect();

await ws.subscribe("polymarket_trades", { condition_ids: ["0xabc..."] });

ws.on("trade_stream_update", (event) => {
	console.log(event.condition_id, event.price, event.size, event.side);
});
```

```typescript
const alerts = new StructAlertsWebSocket({ apiKey: process.env.STRUCT_API_KEY! });
await alerts.connect();

await alerts.subscribe("trader_whale_trade", {
	wallet_addresses: ["0xd91..."],
	min_usd_value: 10000,
});

alerts.on("trader_whale_trade", (payload) => {
	console.log(payload.data.trader, payload.data.amount_usd);
});
```

See [websockets.md](websockets.md) for all rooms, filters, payload types, lifecycle events, reconnection, and patterns.

## Pagination

```typescript
import { StructClient, paginate } from "@structbuild/sdk";

for await (const market of paginate(
	(params) => client.markets.getMarkets(params),
	{ limit: 100 },
)) {
	console.log(market.slug);
}
```

## Trade Discriminated Unions

`getTrades` and `getTraderTrades` return a tagged union of on-chain event types. Narrow on `trade_type`:

```typescript
import type { Trade, MarketTrade, OracleEvent, TradeEventType } from "@structbuild/sdk";

const { data: trades } = await client.markets.getTrades();

for (const trade of trades) {
	switch (trade.trade_type) {
		case "OrderFilled":
		case "OrdersMatched":
			console.log(trade.price, trade.usd_amount);
			break;
		case "Redemption":
			console.log(trade.winning_outcome_index);
			break;
		case "Merge":
		case "Split":
			console.log(trade.usd_amount, trade.position_details);
			break;
	}
}
```

Convenience sub-unions: `MarketTrade` (on-chain trades), `OracleEvent` (UMA lifecycle: `Initialization`, `Proposal`, `Dispute`, `Settled`, `Resolution`, `ConditionResolution`, `Reset`, `Flag`, `Unflag`, `Pause`, `Unpause`, `ManualResolution`, `NegRiskOutcomeReported`), `TradeEventType` (string-literal union for filtering).

## Error Handling

```typescript
import { HttpError, TimeoutError, NetworkError, WebSocketError, WebSocketClosedError } from "@structbuild/sdk";

try {
	await client.markets.getMarket({ conditionId: "0x..." });
} catch (error) {
	if (error instanceof HttpError) {
		if (error.status === 404) return null;
		console.log(`HTTP ${error.status}`, error.body);
	} else if (error instanceof TimeoutError) {
		console.log("Request timed out");
	} else if (error instanceof NetworkError) {
		console.log("Network error:", error.message);
	}
}
```

## Tips

- **Parameter naming**: Path parameters are camelCase (`conditionId`, `positionId`, `webhookId`). Query parameters are snake_case (`condition_id`, `condition_ids`, `event_slug`, `sort_by`, `pagination_key`, `count_back`). Watch the inconsistency — `getMarket` takes `{ conditionId }` (path), but `getMarketChart` takes `{ condition_id }` (query).
- **`marketSlug` not `slug`**: `getMarketBySlug({ marketSlug })`. But `getEventBySlug({ slug })`. Different params for similar-looking methods.
- **Candlestick `resolution`, not `interval`**: `getCandlestick` and `getPositionCandlestick` use `resolution: "1" | "5" | "15" | "30" | "60" | "240" | "D" | "1D"`. `getEventChart` uses `resolution: "1H" | "6H" | "1D" | "1W" | "1M" | "ALL"`. Trader PnL candles use `resolution: "1h" | "4h" | "1d" | "1w"`. They are not interchangeable.
- **Both candlestick endpoints accept `count_back`** (max 2500) and `from`/`to` (Unix seconds) for time range queries.
- **Batch event lookup**: `client.events.getEvents({ event_slugs: "slug1,slug2" })` (max 50). Pass `include_metrics: false` for lighter responses during enrichment.
- **Event type alias**: Import `Event` as `PolymarketEvent` to avoid collision with the DOM `Event` type.
- **Pagination key types vary**: Most endpoints use string keys; bonds and leaderboard use numeric keys/offsets. `search` returns separate keys per result type (`events_pagination_key`, `markets_pagination_key`, `traders_pagination_key`).
- **Portfolio composition**: There is no single portfolio endpoint. Compose from parallel `getTraderPnl`, `getTraderPnlCandles`, `getTraderMarketPnl`, `getTraderOutcomePnl`.
- **404 → null**: Trader / market / event lookups return `HttpError` with `status === 404` for missing records. Catch and return `null` rather than propagating.
- **Webhook signature**: Verify deliveries using the `x-struct-signature` header with HMAC-SHA256 against the raw request body. See [webhooks.md](webhooks.md).
- **`pk_jwt_*` keys are frontend-safe**: They are useless without a valid JWT. Server-side `sk_*` keys must never ship to the browser.
- **Type extraction trick**: Pull SDK request types without importing each one — `type GetMarketsRequest = Parameters<StructClient["markets"]["getMarkets"]>[0]`.

## Additional Resources

- [API Reference](api-reference.md) — Full method reference with types
- [WebSockets](websockets.md) — Rooms, alerts, payload types, reconnection
- [Webhooks](webhooks.md) — Events, filters, payload format, signature verification
- [Guides](guides.md) — Real-world recipes and patterns

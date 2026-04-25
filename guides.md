# Building with the Struct SDK

Practical patterns for building applications with prediction market data.

## Market Screening

```typescript
const markets = await client.markets.getMarkets({
	status: "open",
	sort_by: "volume",
	min_volume: 100000,
	include_metrics: true,
	limit: 20,
});

const politics = await client.markets.getMarkets({
	categories: "politics",
	status: "open",
	sort_by: "volume",
	include_tags: true,
});

const jumps = await client.markets.getPriceJumps({ limit: 20 });
```

## Search With Per-Result Pagination

`search.search` returns multiple result kinds in one response, each with its own pagination cursor:

```typescript
const res = await client.search.search({ query: "bitcoin", sort_by: "volume" });

res.data.events;
res.data.markets;
res.data.traders;

res.data.events_pagination?.has_more;
res.data.events_pagination?.pagination_key;
res.data.markets_pagination?.pagination_key;
res.data.traders_pagination?.pagination_key;
```

Track each cursor separately if you build a paged search UI.

## Trader Analytics

### Leaderboard

```typescript
const top = await client.trader.getLeaderboard({
	timeframe: "30d",
	sort_by: "pnl",
	category: "politics",
	limit: 50,
});

for (const row of top.data) {
	console.log(row.address, row.pnl_usd, row.markets_traded, row.win_rate_pct);
}
```

`getGlobalPnl` is the older equivalent and still works; `getLeaderboard` is the recommended path going forward (supports category filtering and a defined PnL/volume sort axis).

### Top Traders by Market or Event

```typescript
const marketTop = await client.markets.getMarketTopTraders({ condition_id: "0x...", timeframe: "30d", limit: 25 });
const positionTop = await client.markets.getPositionTopTraders({ position_id: "pos_123", timeframe: "30d" });
const eventTop = await client.events.getEventTopTraders({ event_slug: "us-election", timeframe: "lifetime" });
```

### Full Trader Breakdown

```typescript
const address = "0xabc...";

const profile = await client.trader.getTraderProfile({ address });
const pnl = await client.trader.getTraderPnl({ address, timeframe: "30d" });
const markets = await client.trader.getTraderMarketPnl({
	address,
	sort_by: "realized_pnl_usd",
	sort_direction: "desc",
	limit: 10,
});
const trades = await client.trader.getTraderTrades({ address, limit: 20 });
```

## Trader Portfolio Composition

There is no single portfolio endpoint. Compose a full trader view from parallel calls:

```typescript
const address = "0xabc...";
const timeframe = "30d";

const [pnl, candles, marketPnl, outcomePnl] = await Promise.all([
	client.trader.getTraderPnl({ address, timeframe }),
	client.trader.getTraderPnlCandles({ address, resolution: "1d", limit: 30 }),
	client.trader.getTraderMarketPnl({ address, timeframe, limit: 100 }),
	client.trader.getTraderOutcomePnl({
		address,
		timeframe,
		limit: 200,
		sort_by: "buy_usd",
		sort_direction: "desc",
	}),
]);
```

## Market Deep Dive

```typescript
const condition_id = "0xabc...";

const [market, metrics, chart, holders, volume] = await Promise.all([
	client.markets.getMarket({ conditionId: condition_id }),
	client.markets.getMarketMetrics({ condition_id }),
	client.markets.getMarketChart({ condition_id }),
	client.holders.getMarketHolders({ condition_id }),
	client.markets.getMarketVolumeChart({ condition_id }),
]);
```

### Candlestick Data

```typescript
const candles = await client.markets.getCandlestick({
	condition_id: "0xabc...",
	resolution: "60",
	count_back: 168,
});

for (const bar of candles.data) {
	console.log(`${bar.t}: O ${bar.o} H ${bar.h} L ${bar.l} C ${bar.c} V ${bar.v}`);
}
```

| Endpoint | Resolution Values |
| -------- | ----------------- |
| `getCandlestick` / `getPositionCandlestick` | `"1"`, `"5"`, `"15"`, `"30"`, `"60"`, `"240"`, `"D"`, `"1D"` |
| `getEventChart` | `"1H"`, `"6H"`, `"1D"`, `"1W"`, `"1M"`, `"ALL"` |
| `getTraderPnlCandles` | `"1h"`, `"4h"`, `"1d"`, `"1w"` |
| `analytics.get*Timeseries` / `getDeltas` | `"1m"`, `"5m"`, `"15m"`, `"30m"`, `"1h"`, `"4h"`, `"1d"`, `"1w"` |

Both candlestick endpoints accept `count_back` (max 2500) and `from`/`to` (Unix seconds).

## Analytics: Pre-Aggregated Metric Deltas

The `analytics` namespace serves time-bucketed metric data without summing trades client-side. Three shapes per scope:

- **Deltas** — array of buckets, each with `volume_delta`, `txns_delta`, etc.
- **Changes** — single percentage-change object comparing two windows
- **Timeseries** — array of cumulative point-in-time values

```typescript
const last30Days = await client.analytics.getMarketTimeseries({
	condition_id: "0x...",
	resolution: "1d",
	count_back: 30,
});

const dailyChanges = await client.analytics.getMarketChanges({
	condition_id: "0x...",
	resolution: "1d",
});

const traderActivity = await client.analytics.getTraderTimeseries({
	address: "0x...",
	resolution: "1d",
	count_back: 90,
});
```

For "all-time" charts, page until `pagination.has_more === false` — analytics endpoints can return many buckets:

```typescript
const all: TimeBucketRow[] = [];
let cursor: string | number | null = null;

while (true) {
	const res = await client.analytics.getMarketTimeseries({
		condition_id: "0x...",
		resolution: "1d",
		pagination_key: cursor ?? undefined,
	});
	all.push(...res.data);
	if (!res.pagination?.has_more) break;
	cursor = res.pagination.pagination_key;
	if (all.length > 5000) break; // bound runaway loops
}
```

## Batch Event Enrichment

When enriching positions with event metadata, batch-fetch by slug (max 50 per call):

```typescript
const slugs = ["us-election-2024", "bitcoin-100k", "fed-rate-cut"];

const batches: string[][] = [];
for (let i = 0; i < slugs.length; i += 50) batches.push(slugs.slice(i, i + 50));

const results = await Promise.all(
	batches.map((batch) =>
		client.events.getEvents({ event_slugs: batch.join(","), include_metrics: false }),
	),
);

const eventBySlug = new Map<string, Event>();
for (const res of results) {
	if (!res.success) continue;
	for (const event of res.data) eventBySlug.set(event.event_slug, event);
}
```

Pass `include_metrics: false` to keep the response small during enrichment.

## Paginating Large Datasets

```typescript
import { paginate } from "@structbuild/sdk";

for await (const market of paginate(
	(params) => client.markets.getMarkets(params),
	{ status: "open" },
)) {
	// ...
}

for await (const trade of paginate(
	(params) => client.trader.getTraderTrades({ address: "0x...", ...params }),
)) {
	// ...
}
```

## Real-Time: Trade Tape

Stream live trades for one or more markets:

```typescript
import { StructWebSocket } from "@structbuild/sdk";

const ws = new StructWebSocket({ apiKey: process.env.STRUCT_API_KEY! });
await ws.connect();

await ws.subscribe("polymarket_trades", {
	condition_ids: ["0xabc...", "0xdef..."],
	trade_types: ["OrderFilled", "OrdersMatched"],
});

ws.on("trade_stream_update", (event) => {
	console.log(event.condition_id, event.side, event.price, event.size);
});
```

## Real-Time: Live Order Book

```typescript
await ws.subscribe("polymarket_order_book", { condition_ids: ["0xabc..."] });

ws.on("order_book_update", (book) => {
	bestBid = book.bids[0]?.price;
	bestAsk = book.asks[0]?.price;
});
```

## Real-Time: Market Stream (REST shape, push delivery)

`polymarket_markets_stream` and `polymarket_events_stream` push REST-shaped rows on a fixed cadence. Seed your UI from REST, then merge updates by `latest_block` to handle reordering:

```typescript
const seed = await client.markets.getMarkets({ status: "open", sort_by: "volume", limit: 50 });
const byId = new Map(seed.data.map((m) => [m.condition_id, m]));

await ws.subscribe("polymarket_markets_stream", {
	interval_ms: 1000,
	min_volume: 50000,
	categories: "politics",
});

ws.on("markets_stream_update", (update) => {
	for (const market of update.markets) {
		const prev = byId.get(market.condition_id);
		if (prev && market.latest_block && prev.latest_block && market.latest_block <= prev.latest_block) continue;
		byId.set(market.condition_id, market);
	}
});
```

## Alerts: Webhook Events Without a Receiver

Use `StructAlertsWebSocket` during development or for in-app notifications when running an HTTPS receiver isn't worth it:

```typescript
import { StructAlertsWebSocket } from "@structbuild/sdk";

const alerts = new StructAlertsWebSocket({ apiKey: process.env.STRUCT_API_KEY! });
await alerts.connect();

await alerts.subscribe("trader_whale_trade", { min_usd_value: 25000 });

alerts.on("trader_whale_trade", (payload) => {
	notify(`Whale: ${payload.data.trader} ${payload.data.side} $${payload.data.amount_usd}`);
});
```

The same filters and payload shapes work as the corresponding webhook event.

## Frontend-Safe Auth With JWT Public Keys

Issue a `pk_jwt_*` key in the dashboard, configure your auth provider's JWKS, then ship the key in your bundle:

```typescript
const client = new StructClient({
	apiKey: "pk_jwt_a1b2c3...",
	jwt: await getAccessToken(),
});

const ws = new StructWebSocket({
	apiKey: "pk_jwt_a1b2c3...",
	getJwt: () => sessionStore.accessToken,
});
```

The `pk_jwt_*` key is useless without a valid JWT from a configured provider, so it is safe to expose. Use `getJwt` for sockets so reconnects build the URL with a fresh token. Per-session rate limits apply per unique `sub` claim.

## Error Handling

### Retry-Aware Setup

```typescript
const client = new StructClient({
	apiKey: process.env.STRUCT_API_KEY!,
	retry: { maxRetries: 3, initialDelayMs: 1000, maxDelayMs: 30000 },
});
```

The SDK retries 429 and 5xx automatically with exponential backoff.

### 404 → null

Trader, market, and event lookups commonly 404 for unknown identifiers. Catch and return `null`:

```typescript
import { HttpError } from "@structbuild/sdk";

async function getProfile(address: string): Promise<UserProfile | null> {
	try {
		const res = await client.trader.getTraderProfile({ address });
		return res.data ?? null;
	} catch (err) {
		if (err instanceof HttpError && err.status === 404) return null;
		throw err;
	}
}
```

### Pagination Loops With Mid-Flight Errors

Don't drop already-collected data when the next page fails. Treat partial results as success:

```typescript
const buckets: TimeBucketRow[] = [];
let cursor: string | number | null = null;

for (let i = 0; i < 20; i++) {
	try {
		const res = await client.analytics.getMarketTimeseries({
			condition_id,
			resolution: "1h",
			pagination_key: cursor ?? undefined,
		});
		buckets.push(...res.data);
		if (!res.pagination?.has_more) break;
		cursor = res.pagination.pagination_key;
	} catch (err) {
		console.warn("pagination broke; returning partial", err);
		break;
	}
}
```

## Type Helpers

Pull request types directly off the client without importing each one:

```typescript
import type { StructClient } from "@structbuild/sdk";

type GetMarketsRequest = Parameters<StructClient["markets"]["getMarkets"]>[0];
type GetTraderOutcomePnlRequest = Parameters<StructClient["trader"]["getTraderOutcomePnl"]>[0];
type SearchRequest = Parameters<StructClient["search"]["search"]>[0];
```

Useful for typed form inputs, query objects, or building generic data layers.

## Trade Type Narrowing

`getTrades` and `getTraderTrades` return a discriminated union. Narrow on `trade_type` before reading variant-specific fields:

```typescript
import type { Trade } from "@structbuild/sdk";

function summarize(trade: Trade): string {
	switch (trade.trade_type) {
		case "OrderFilled":
		case "OrdersMatched":
			return `${trade.side} ${trade.shares_amount} @ ${trade.price}`;
		case "Merge":
		case "Split":
			return `${trade.trade_type} $${trade.usd_amount}`;
		case "Redemption":
			return `redeem outcome ${trade.winning_outcome_index}`;
		case "PositionsConverted":
			return `convert ${trade.usd_amount}`;
		default:
			return trade.trade_type;
	}
}
```

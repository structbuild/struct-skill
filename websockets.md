# WebSockets Reference

Real-time streaming for prediction market data. Two separate clients with different mental models:

- **`StructWebSocket`** — room-based feeds (`wss://api.struct.to/ws`). One subscription per room, multiplexed over a single connection. Best for trades, order books, account balances, market/event streams.
- **`StructAlertsWebSocket`** — event-based alerts (`wss://api.struct.to/ws/alerts`). Same event names and filters as webhooks. Best for in-app notifications without running a webhook receiver.

Both auto-reconnect with exponential backoff and replay subscriptions on reconnect. Both ping every 30s and close the socket if no pong within 60s.

## StructWebSocket

```typescript
import { StructWebSocket } from "@structbuild/sdk";

const ws = new StructWebSocket({
	apiKey: process.env.STRUCT_API_KEY!,
	// Optional:
	jwt: userToken,
	getJwt: () => userToken,
	baseUrl: "wss://api.struct.to",
	subscribeTimeout: 10_000,
	reconnect: { maxRetries: Infinity, initialDelayMs: 500, maxDelayMs: 30_000 },
});

await ws.connect();
```

### Subscribing to rooms

`subscribe(room, filters?)` is fully typed per room. Some rooms accept optional filters; others require them.

```typescript
const res = await ws.subscribe("polymarket_trades", {
	condition_ids: ["0xabc..."],
});

await ws.subscribe("polymarket_order_book", { asset_ids: ["0xabc..."] });
await ws.subscribe("polymarket_asset_prices");
await ws.subscribe("polymarket_clob_rewards", { subscribe_all: true });

ws.unsubscribe("polymarket_trades");
```

`subscribe()` returns a Promise that resolves with the room's typed `*_subscribe_response` data once the server confirms, or rejects with `WebSocketError` (timeout, server-side error, or superseded by a newer subscribe to the same room).

### Listening for updates

```typescript
ws.on("trade_stream_update", (event) => {
	event.condition_id;
	event.price;
	event.size;
	event.side;
});

ws.on("order_book_update", (event) => {
	event.asset_id;
	event.bids;
	event.asks;
});

const off = ws.on("market_metrics_update", (event) => { /* ... */ });
off(); // unsubscribe a single listener

ws.once("connected", () => { /* fires once */ });
ws.removeAllListeners("trade_stream_update");
```

### Available rooms

| Room | Required Filters | Optional Filters | Event Type(s) |
| ---- | ---------------- | ---------------- | ------------- |
| `polymarket_trades` | — | `condition_ids`, `market_slugs`, `event_slugs`, `position_ids`, `traders`, `trade_types`, `status`, `subscribe_all` | `trade_stream_update` |
| `polymarket_asset_prices` | — | `asset_symbols` | `asset_price_tick`, `asset_price_window_update` |
| `polymarket_asset_window_updates` | `asset_symbols` | `timeframes` | `asset_window_update` |
| `polymarket_market_metrics` | `condition_ids` | — | `market_metrics_update` |
| `polymarket_event_metrics` | `event_slugs` | — | `event_metrics_update` |
| `polymarket_position_metrics` | `position_ids` | — | `position_metrics_update` |
| `polymarket_trader_pnl` | `traders` | — | `trader_global_pnl_update`, `trader_market_pnl_update`, `trader_event_pnl_update` |
| `polymarket_trader_positions` | `traders` | — | `trader_position_update` |
| `polymarket_accounts` | `wallets` | `include_usdce`, `include_matic`, `include_pusd` | `accounts_update`, `usdce_update`, `matic_update`, `pusd_update` |
| `polymarket_order_book` | — | `condition_ids`, `position_ids` | `order_book_update` |
| `polymarket_clob_rewards` | — | `condition_ids`, `subscribe_all` | `clob_rewards_update` |
| `polymarket_events_stream` | `interval_ms` | filter mode: `search`, `categories`, `tags`, `min_volume`, `timeframe`, … / IDs mode: `event_slugs`, `event_ids` (max 500) | `events_stream_update` |
| `polymarket_markets_stream` | `interval_ms` | filter mode: `search`, `categories`, `tags`, `min_volume`, `min_txns`, `min_liquidity`, `timeframe`, … / IDs mode: `condition_ids`, `market_slugs`, `event_slugs` (max 500 total) | `markets_stream_update` |
| `polymarket_oracle_events` | — | `condition_ids`, `event_slugs`, `event_types` | `oracle_event_stream` (typed `OracleEventTyped`) |

### Stream rooms (`events_stream`, `markets_stream`)

These rooms push REST-shaped rows on a fixed cadence rather than per-event. Each subscription declares one of four `interval_ms` values (`500`, `1000`, `3000`, `10000`) and either filter mode (server-side filter expression) or IDs mode (`event_ids` / `condition_ids` / `market_slugs`, max 500). Markets stream filter mode rejects `status` — the server cache only holds open markets. There is no initial snapshot; seed your UI from the matching REST list endpoint, then merge stream updates by `latest_block` / `latest_confirmed_at` to handle reordering.

### Lifecycle events

```typescript
ws.on("connected", () => {});
ws.on("disconnected", ({ code, reason }) => {});
ws.on("reconnecting", ({ attempt }) => {});
ws.on("reconnect_failed", (err) => {});
ws.on("auth_failed", (err) => {});
ws.on("warning", (warning) => {});
ws.on("error", (err) => {});
```

### Reconnection and replay

The transport reconnects automatically. On `connected`, the SDK re-issues `join_room` + `subscribe` for every active subscription, so listeners stay bound and your room state survives socket drops. `disconnect()` clears subscriptions and stops reconnecting.

### Cleanup

```typescript
ws.unsubscribe("polymarket_trades");
ws.disconnect();
```

## StructAlertsWebSocket

Same event names as webhooks. Same filter shapes. Same payload schemas. The difference is delivery mechanism — push over a socket instead of HTTP POST to your endpoint.

```typescript
import { StructAlertsWebSocket } from "@structbuild/sdk";

const alerts = new StructAlertsWebSocket({ apiKey: process.env.STRUCT_API_KEY! });
await alerts.connect();

await alerts.subscribe("trader_whale_trade", {
	wallet_addresses: ["0xd91..."],
	min_usd_value: 10000,
});

await alerts.subscribe("probability_spike", {
	spike_direction: "up",
	min_probability_change_pct: 5,
	timeframes: ["1h", "24h"],
});

alerts.on("trader_whale_trade", (payload) => {
	payload.event;          // "trader_whale_trade"
	payload.timestamp;      // delivery time (ms)
	payload.data.trader;    // event-specific payload
	payload.data.amount_usd;
});

alerts.unsubscribe("probability_spike");
alerts.disconnect();
```

### Supported alert events

`StructAlertsWebSocket` mirrors the full webhook event list — see [webhooks.md](webhooks.md) for events, filters, and payload shapes. Subscribe to one event at a time; pass per-event filters as the second argument.

```typescript
type WsAlertEventName =
	| "trader_first_trade" | "trader_new_market" | "trader_whale_trade"
	| "trader_new_trade" | "trader_trade_event"
	| "trader_global_pnl" | "trader_market_pnl" | "trader_event_pnl"
	| "condition_metrics" | "event_metrics" | "position_metrics"
	| "market_volume_milestone" | "event_volume_milestone" | "position_volume_milestone"
	| "market_volume_spike" | "event_volume_spike" | "position_volume_spike"
	| "probability_spike" | "price_spike"
	| "close_to_bond" | "market_created" | "oracle_events"
	| "asset_price_tick" | "asset_price_window_update";
```

## Patterns

### Seed-from-REST-then-stream

Stream rooms emit only deltas. Render the initial UI from REST, then apply socket updates:

```typescript
const seed = await client.events.getEvents({ limit: 50, sort_by: "volume" });
renderEvents(seed.data);

await ws.subscribe("polymarket_events_stream", {
	interval_ms: 1000,
	min_volume: 50_000,
});

ws.on("events_stream_update", (update) => {
	mergeEventsByBlock(update);
});
```

### Subscription deduplication

Track active subscriptions to avoid duplicate `subscribe()` calls when multiple components mount with the same key:

```typescript
const active = new Set<string>();

async function ensureMarketStream(conditionId: string) {
	if (active.has(conditionId)) return;
	active.add(conditionId);
	await ws.subscribe("polymarket_market_metrics", { condition_ids: [conditionId] });
}
```

When the last subscriber unmounts, call `ws.unsubscribe(room)` and remove from the set.

### JWT-authenticated browser clients

Pass `getJwt` rather than `jwt` so reconnects pick up rotated tokens. The SDK rebuilds the WS URL each connect:

```typescript
const ws = new StructWebSocket({
	apiKey: "pk_jwt_...",
	getJwt: () => sessionStore.getAccessToken(),
});
```

### Error and lifecycle handling

```typescript
ws.on("auth_failed", (err) => {
	// Refresh JWT, then reconnect
});

ws.on("disconnected", ({ code }) => {
	if (code === 4001) {
		// Server-initiated close (e.g., subscription invalid). Don't blindly reconnect.
	}
});

ws.on("error", (err) => log.error(err));
```

## Imports

```typescript
import {
	StructWebSocket,
	StructAlertsWebSocket,
	WebSocketError,
	WebSocketClosedError,
} from "@structbuild/sdk";
import type {
	WsRoomId,
	StructWebSocketConfig,
	ConnectionState,
	TradeStreamEvent,
	MarketMetricsEvent,
	EventMetricsEvent,
	PositionMetricsEvent,
	OrderBookUpdateEvent,
	OracleEventStreamEvent,
	AccountsUpdateEvent,
	WsAlertEventName,
	WsAlertEventPayload,
} from "@structbuild/sdk";
```

# API Reference

Complete method reference for all SDK namespaces. Every method accepts an optional `venue` parameter as its last argument and returns `Promise<HttpResponse<T>>`.

## Response Format

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

## Client Configuration

```typescript
new StructClient({
	apiKey: string,                                  // sk_* or pk_jwt_*
	jwt?: string,                                    // required when apiKey is pk_jwt_*
	getJwt?: () => string | undefined,               // for rotating tokens
	venue?: "polymarket",                            // default
	baseUrl?: string,                                // default https://api.struct.to/v1
	timeout?: number,                                // ms
	headers?: Record<string, string>,
	retry?: { maxRetries?: number; initialDelayMs?: number; maxDelayMs?: number },
	onRequest?: (info: { method: string; url: string }) => void,
	onResponse?: (info: { status: number; durationMs: number; url: string }) => void,
});
```

## Markets Namespace

```typescript
client.markets.getMarkets(params?, venue?)
// params: { limit?, pagination_key?, sort_by?, ascending?, status?, categories?, tags?, min_volume?, include_metrics?, include_tags? }
// returns: HttpResponse<MarketResponse[]>

client.markets.getMarket({ conditionId, ...query }, venue?)
// returns: HttpResponse<MarketResponse>

client.markets.getMarketBySlug({ marketSlug, ...query }, venue?)
// returns: HttpResponse<MarketResponse>

client.markets.getMarketChart({ condition_id, ...query }, venue?)
// returns: HttpResponse<PositionChartOutcome[]>

client.markets.getMarketMetrics({ condition_id, timeframe?, ...query }, venue?)
// returns: HttpResponse<ConditionMetricsResponse>

client.markets.getTrades({ condition_ids?, market_slugs?, event_slugs?, position_ids?, traders?, trade_types?, status?, limit?, pagination_key? }, venue?)
// returns: HttpResponse<Trade[]>

client.markets.getCandlestick({ condition_id, resolution, count_back?, from?, to?, pagination_key? }, venue?)
// resolution: "1" | "5" | "15" | "30" | "60" | "240" | "D" | "1D"
// returns: HttpResponse<PredictionCandlestickBar[]>

client.markets.getPositionCandlestick({ position_id, resolution, count_back?, from?, to? }, venue?)
// returns: HttpResponse<PredictionCandlestickBar[]>

client.markets.getPositionMetrics({ position_id, timeframe?, ...query }, venue?)
// returns: HttpResponse<PositionMetricsResponse>

client.markets.getMarketVolumeChart({ condition_id, ...query }, venue?)
// returns: HttpResponse<MarketVolumeDataPoint[]>

client.markets.getPositionVolumeChart({ position_id, ...query }, venue?)
// returns: HttpResponse<PositionVolumeDataPoint[]>

client.markets.getPriceJumps(params?, venue?)
// returns: HttpResponse<PriceJump[]>

client.markets.getOracleEvents(params?, venue?)
// params: { condition_ids?, event_slugs?, event_types?, limit?, pagination_key? }
// returns: HttpResponse<TradeEvent[]>

client.markets.getMarketTopTraders({ condition_id, timeframe?, sort_by?, limit? }, venue?)
client.markets.getPositionTopTraders({ position_id, timeframe?, sort_by?, limit? }, venue?)
```

## Events Namespace

```typescript
client.events.getEvents(params?, venue?)
// params: { limit?, pagination_key?, sort_by?, ascending?, status?, categories?, tags?, event_slugs?, include_metrics?, include_tags? }
// returns: HttpResponse<Event[]>

client.events.getEvent({ identifier, ...query }, venue?)
// returns: HttpResponse<Event>

client.events.getEventBySlug({ slug, ...query }, venue?)
// returns: HttpResponse<Event>

client.events.getEventChart({ event_slug, resolution?, ...query }, venue?)
// resolution: "1H" | "6H" | "1D" | "1W" | "1M" | "ALL"
// returns: HttpResponse<EventMarketChartOutcome[]>

client.events.getEventMetrics({ event_slug, timeframe?, ...query }, venue?)
// returns: HttpResponse<EventMetricsResponse>

client.events.getEventOutcomes({ event_slug, ...query }, venue?)
// returns: HttpResponse<Record<string, string>>

client.events.getEventTopTraders({ event_slug, timeframe?, sort_by?, limit? }, venue?)
```

## Trader Namespace

```typescript
client.trader.getTraderTrades({ address, ...query }, venue?)
// query: { condition_ids?, event_slugs?, trade_types?, status?, limit?, pagination_key? }
// returns: HttpResponse<Trade[]>

client.trader.getTraderProfile({ address }, venue?)
// returns: HttpResponse<UserProfile>

client.trader.getTraderProfilesBatch({ addresses }, venue?)
// addresses: comma-separated wallet addresses (max 50)
// returns: HttpResponse<UserProfile[]>

client.trader.getTraderVolumeChart({ address, ...query }, venue?)
// returns: HttpResponse<TraderVolumeDataPoint[]>

client.trader.getTraderPnl({ address, timeframe?, ...query }, venue?)
// returns: HttpResponse<TraderPnlSummary>

client.trader.getTraderOutcomePnl({ address, timeframe?, sort_by?, sort_direction?, limit?, pagination_key? }, venue?)
// returns: HttpResponse<TraderOutcomePnlEntry[]>

client.trader.getTraderMarketPnl({ address, timeframe?, sort_by?, sort_direction?, limit?, pagination_key? }, venue?)
// returns: HttpResponse<TraderMarketPnlEntry[]>

client.trader.getTraderEventPnl({ address, timeframe?, sort_by?, sort_direction?, limit?, pagination_key? }, venue?)
// returns: HttpResponse<TraderEventPnlEntry[]>

client.trader.getTraderPnlCandles({ address, resolution?, start_ts?, end_ts?, limit? }, venue?)
// resolution: "1h" | "4h" | "1d" | "1w"
// returns: HttpResponse<PnlCandleEntry[]>

client.trader.getTraderPnlCalendar({ address, ...query }, venue?)
// returns: HttpResponse<PnlCandleEntry[]>

client.trader.getGlobalPnl(params?, venue?)
// params: { timeframe?, sort_by?, limit?, pagination_key? }
// returns: HttpResponse<GlobalPnlTrader[]>

client.trader.getLeaderboard(params?, venue?)
// params: { timeframe?: "1d"|"7d"|"30d"|"lifetime", sort_by?: "pnl"|"volume", category?: "overall"|"politics"|..., limit?, offset? }
// returns: HttpResponse<LeaderboardEntry[]>
```

## Holders Namespace

```typescript
client.holders.getMarketHolders({ condition_id, ...query }, venue?)
// returns: HttpResponse<MarketHoldersResponse>

client.holders.getPositionHolders({ positionId, ...query }, venue?)
// returns: HttpResponse<PositionHoldersResponse>

client.holders.getMarketHoldersHistory({ condition_id, ...query }, venue?)
// returns: HttpResponse<HolderHistoryCandle[]>

client.holders.getPositionHoldersHistory({ positionId, ...query }, venue?)
// returns: HttpResponse<HolderHistoryCandle[]>
```

## Series Namespace

```typescript
client.series.getSeriesList(params?, venue?)
// returns: HttpResponse<Series[]>

client.series.getSeriesOutcomes({ series_slug, ...query }, venue?)
// returns: HttpResponse<Record<string, string>>
```

## Search Namespace

```typescript
client.search.search({ query, sort_by?, limit?, pagination_key? }, venue?)
// returns: HttpResponse<SearchResponse>

// Note: SearchResponse exposes separate pagination keys per result kind:
// events_pagination?: { has_more, pagination_key }
// markets_pagination?: { has_more, pagination_key }
// traders_pagination?: { has_more, pagination_key }
```

## Tags Namespace

```typescript
client.tags.getTags(params?, venue?)
// params: { sort_by?: TagSortBy, timeframe?: TagSortTimeframe, limit?, pagination_key? }
// returns: HttpResponse<Tag[]>

client.tags.getTag({ identifier }, venue?)
// returns: HttpResponse<Tag>

client.tags.searchTags({ query, limit? }, venue?)
// returns: HttpResponse<Tag[]>
```

## Bonds Namespace

```typescript
client.bonds.getBonds(params?, venue?)
// params: { sort_by?: BondsSortBy, limit?, pagination_key? } — pagination_key is numeric
// returns: HttpResponse<BondMarket[]>
```

## OrderBook Namespace

```typescript
client.orderBook.getOrderBook({ asset_id, ...query }, venue?)
client.orderBook.getOrderBookHistory({ position_id?, condition_id?, market_slug?, limit?, pagination_key? }, venue?)
client.orderBook.getMarketOrderBook({ condition_id?, market_slug? }, venue?)
client.orderBook.getSpreadHistory({ position_id?, condition_id?, market_slug?, limit?, pagination_key? }, venue?)
```

## Assets Namespace

```typescript
client.assets.getAssetHistory({ symbol, variant, ...query }, venue?)
// symbol: "BTC" | "ETH" | "SOL" | "XRP"; variant: "1m" | "5m" | "1h" | "1d" | ...
// returns: HttpResponse<AssetPriceHistoryRow[]>
```

## Analytics Namespace

Time-bucketed metric deltas, percentage changes between two windows, and timeseries arrays. Each scope (`global`, `event`, `market`, `tag`, `trader`) supports the same `Deltas / Changes / Timeseries` shape.

```typescript
client.analytics.getCounts(venue?)
// returns: HttpResponse<GlobalCountsResponse>

client.analytics.getDeltas(params?, venue?)
client.analytics.getChanges(params, venue?)
client.analytics.getTimeseries(params?, venue?)
// params: { resolution?: AnalyticsResolution, count_back?, from?, to?, pagination_key? }
// returns: HttpResponse<TimeBucketRow[]>  (Changes returns MetricPctChange)

client.analytics.getEventDeltas({ event_slug, ...query }, venue?)
client.analytics.getEventChanges({ event_slug, ...query }, venue?)
client.analytics.getEventTimeseries({ event_slug, ...query }, venue?)

client.analytics.getMarketDeltas({ condition_id, ...query }, venue?)
client.analytics.getMarketChanges({ condition_id, ...query }, venue?)
client.analytics.getMarketTimeseries({ condition_id, ...query }, venue?)

client.analytics.getTagDeltas({ identifier, ...query }, venue?)
client.analytics.getTagChanges({ identifier, ...query }, venue?)
client.analytics.getTagTimeseries({ identifier, ...query }, venue?)

client.analytics.getTraderDeltas({ address, ...query }, venue?)
client.analytics.getTraderChanges({ address, ...query }, venue?)
client.analytics.getTraderTimeseries({ address, ...query }, venue?)
```

`AnalyticsResolution`: `"1m" | "5m" | "15m" | "30m" | "1h" | "4h" | "1d" | "1w"`. Trader scope uses a slimmer resolution set (`1h`, `4h`, `1d`, `1w`).

## Webhooks Namespace

Platform-level (not venue-scoped):

```typescript
client.webhooks.list(params?)
// returns: HttpResponse<WebhookListResponseBody>

client.webhooks.create({ url, event, secret?, description?, filters? })
// event: PolymarketWebhookEvent (single event per webhook — see webhooks.md)
// returns: HttpResponse<WebhookResponse>

client.webhooks.getWebhook({ webhookId })
client.webhooks.update({ webhookId, url?, event?, secret?, description?, filters? })
client.webhooks.deleteWebhook({ webhookId })
client.webhooks.test({ webhookId })            // sends a synthetic delivery
client.webhooks.rotateSecret({ webhookId })    // returns new secret
client.webhooks.listEvents()                    // metadata for every event type
```

## Venue Override

Every method accepts an optional `venue` parameter as the last argument:

```typescript
const markets = await client.markets.getMarkets();                       // default venue
const polymarket = await client.markets.getMarkets({}, "polymarket");    // explicit
```

## WebSocket Clients

```typescript
import { StructWebSocket, StructAlertsWebSocket } from "@structbuild/sdk";

new StructWebSocket({
	apiKey: string,
	jwt?: string,
	getJwt?: () => string | undefined,
	baseUrl?: string,                  // default wss://api.struct.to
	subscribeTimeout?: number,         // default 10000
	reconnect?: { maxRetries?, initialDelayMs?, maxDelayMs? },
});

// Per-class API:
//   connect(): Promise<void>
//   disconnect(): void
//   subscribe(room, filters?): Promise<*_subscribe_response>     (or event for alerts)
//   unsubscribe(room): void
//   on(event, listener): () => void
//   once(event, listener): () => void
//   off(event, listener): void
//   removeAllListeners(event?): void
//   state: "disconnected" | "connecting" | "connected" | "reconnecting"
```

See [websockets.md](websockets.md) for rooms, filters, payload types, and patterns.

## Pagination Helper

```typescript
import { paginate } from "@structbuild/sdk";

for await (const item of paginate(
	(params) => client.markets.getMarkets(params),
	{ limit: 100, status: "open" },
)) {
	// item is one record — paginate flattens pages and follows pagination_key
}
```

## Error Classes

```typescript
import {
	StructError,
	HttpError,           // .status, .statusText, .body, .url, .method
	NetworkError,
	TimeoutError,        // .timeout
	WebSocketError,
	WebSocketClosedError,
} from "@structbuild/sdk";
```

## Type Imports

The SDK exports every generated type from `@structbuild/sdk`:

```typescript
import type {
	// Markets
	MarketResponse, MarketMetadata, ConditionMetricsResponse,
	PredictionCandlestickBar, MarketVolumeDataPoint, PriceJump,
	// Events
	Event as PolymarketEvent, EventMetricsResponse, EventMarketChartOutcome,
	// Trades (discriminated unions)
	Trade, MarketTrade, OracleEvent, TradeEventType,
	OrderFilledTrade, RedemptionTrade, MergeTrade, SplitTrade,
	CancelledTrade, PositionsConvertedTrade, RegisterTokenTrade, ApprovalTrade,
	// Trader
	UserProfile, TraderPnlSummary, GlobalPnlTrader, LeaderboardEntry,
	TraderMarketPnlEntry, TraderEventPnlEntry, TraderOutcomePnlEntry, PnlCandleEntry,
	// Analytics
	TimeBucketRow, MetricPctChange, GlobalCountsResponse,
	AnalyticsResolution,
	// Holders / Series / Tags / Bonds / Assets
	MarketHoldersResponse, PositionHoldersResponse, HolderHistoryCandle,
	Series, Tag, BondMarket, AssetPriceHistoryRow,
	// WebSocket
	WsRoomId, ConnectionState, StructWebSocketConfig,
	TradeStreamEvent, MarketMetricsEvent, EventMetricsEvent, PositionMetricsEvent,
	OrderBookUpdateEvent, OracleEventStreamEvent, AccountsUpdateEvent,
	WsAlertEventName, WsAlertEventPayload,
	// Webhooks
	PolymarketWebhookEvent, CreateWebhookParams, WebhookResponse,
	WebhookDelivery, WebhookFiltersBody,
	// Common
	Venue, ApiResponseInfo, PaginationInfo, HttpResponse,
} from "@structbuild/sdk";
```

To pull a method's request type without importing each one explicitly:

```typescript
type GetMarketsRequest = Parameters<StructClient["markets"]["getMarkets"]>[0];
type SearchRequest = Parameters<StructClient["search"]["search"]>[0];
```

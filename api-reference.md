# API Reference

Complete method reference for all SDK namespaces. Every method accepts an optional `venue` parameter as its last argument and returns `Promise<HttpResponse<T>>`.

## Response Format

```typescript
interface HttpResponse<T> {
	data: T;
	message: string | null;
	success: boolean;
	info?: ApiResponseInfo;
}

interface ApiResponseInfo {
	version: string;
	credits_consumed: number;
}
```

## Markets Namespace

```typescript
client.markets.getMarkets(params?, venue?)
// params: { limit?, offset?, sort_by?, sort_order?, status?, tag?, search? }
// returns: HttpResponse<MarketMetadata[]>

client.markets.getMarket({ conditionId }, venue?)
// returns: HttpResponse<MarketMetadata>

client.markets.getMarketBySlug({ slug }, venue?)
// returns: HttpResponse<MarketMetadata>

client.markets.getMarketMetrics({ conditionId, ...query }, venue?)
// returns: HttpResponse<ConditionMetricsResponse>

client.markets.getTrades(params?, venue?)
// params: { conditionId?, limit?, offset?, trader?, side? }
// returns: HttpResponse<Trade[]>

client.markets.getCandlestick({ conditionId, interval, ...query }, venue?)
// interval: "1m" | "5m" | "15m" | "1h" | "4h" | "1d"
// returns: HttpResponse<CandlestickResponse>

client.markets.getPositionCandlestick({ positionId, interval, ...query }, venue?)
// returns: HttpResponse<CandlestickResponse>

client.markets.getPositionMetrics({ positionId, ...query }, venue?)
// returns: HttpResponse<PositionMetricsResponse>

client.markets.getMarketVolumeChart({ conditionId, ...query }, venue?)
// returns: HttpResponse<MarketVolumeChartResponse>

client.markets.getPositionVolumeChart({ positionId, ...query }, venue?)
// returns: HttpResponse<PositionVolumeChartResponse>
```

## Events Namespace

```typescript
client.events.getEvents(params?, venue?)
// params: { limit?, offset?, sort_by?, sort_order?, status?, tag?, search? }
// returns: HttpResponse<Event[]>

client.events.getEvent({ id, include_tags?, include_markets? }, venue?)
// returns: HttpResponse<Event>

client.events.getEventBySlug({ slug, include_tags?, include_markets? }, venue?)
// returns: HttpResponse<Event>

client.events.getEventMetrics({ eventId, ...query }, venue?)
// returns: HttpResponse<EventMetricsResponse>
```

## Trader Namespace

```typescript
client.trader.getPortfolio({ address, timeframe? }, venue?)
// timeframe: "7d" | "30d" | "lifetime"
// returns: HttpResponse<Portfolio>

client.trader.getPortfolioPositions({ address, limit?, offset?, ...query }, venue?)
// returns: HttpResponse<PositionsResponse>

client.trader.getTraderTrades({ address, limit?, offset?, ...query }, venue?)
// returns: HttpResponse<TradesResponse>

client.trader.getTraderProfile({ address }, venue?)
// returns: HttpResponse<UserProfile>

client.trader.getTraderProfilesBatch({ addresses }, venue?)
// addresses: comma-separated string of wallet addresses
// returns: HttpResponse<Record<string, UserProfile>>

client.trader.getTraderVolumeChart({ address, ...query }, venue?)
// returns: HttpResponse<TraderVolumeChartResponse>

client.trader.getTraderPnl({ address, timeframe? }, venue?)
// timeframe: "7d" | "30d" | "lifetime"
// returns: HttpResponse<TraderPnlSummary>

client.trader.getTraderPositionPnl({ address, timeframe?, sort_by?, sort_direction?, limit?, pagination_key?, condition_id?, event_slug? }, venue?)
// returns: HttpResponse<PnlListResponse<TraderPositionPnlEntry>>

client.trader.getTraderMarketPnl({ address, timeframe?, sort_by?, sort_direction?, limit?, pagination_key?, condition_id?, event_slug? }, venue?)
// returns: HttpResponse<PnlListResponse<TraderMarketPnlEntry>>

client.trader.getTraderEventPnl({ address, timeframe?, sort_by?, sort_direction?, limit?, pagination_key?, condition_id?, event_slug? }, venue?)
// returns: HttpResponse<PnlListResponse<TraderEventPnlEntry>>

client.trader.getTraderPnlCandles({ address, resolution?, start_ts?, end_ts?, limit? }, venue?)
// resolution: "1h" | "4h" | "1d" | "1w"
// returns: HttpResponse<PnlCandlesResponse>

client.trader.getGlobalPnl(params?, venue?)
// returns: HttpResponse<GlobalPnlResponse>
```

## Holders Namespace

```typescript
client.holders.getMarketHolders({ conditionId, ...query }, venue?)
// returns: HttpResponse<MarketHoldersResponse>

client.holders.getEventHolders({ eventSlug, ...query }, venue?)
// returns: HttpResponse<EventHoldersResponse>

client.holders.getPositionHolders({ positionId, ...query }, venue?)
// returns: HttpResponse<PositionHoldersResponse>

client.holders.getMarketHoldersHistory({ conditionId, hours? }, venue?)
// returns: HttpResponse<HolderHistoryCandle[]>

client.holders.getEventHoldersHistory({ eventSlug, hours? }, venue?)
// returns: HttpResponse<HolderHistoryCandle[]>

client.holders.getPositionHoldersHistory({ positionId, hours? }, venue?)
// returns: HttpResponse<HolderHistoryCandle[]>
```

## Scoring Namespace

```typescript
client.scoring.getTraderScore({ address }, venue?)
// returns: HttpResponse<TraderScore>

client.scoring.getSmartMoneyLeaderboard({ limit? }, venue?)
// returns: HttpResponse<SmartMoneyEntry[]>

client.scoring.getInsiderLeaderboard({ limit? }, venue?)
// returns: HttpResponse<InsiderEntry[]>

client.scoring.getBots({ limit? }, venue?)
// returns: HttpResponse<BotEntry[]>
```

## Series Namespace

```typescript
client.series.getSeriesList(params?, venue?)
// returns: HttpResponse<Series[]>

client.series.getSeriesDetail({ identifier, ...query }, venue?)
// identifier: series ID or slug
// returns: HttpResponse<SeriesDetail>

client.series.getSeriesEvents({ identifier, ...query }, venue?)
// returns: HttpResponse<Event[]>
```

## Search Namespace

```typescript
client.search.search({ query, ...params }, venue?)
// returns: HttpResponse<SearchResponse>
```

## Tags Namespace

```typescript
client.tags.getTags(params?, venue?)
// returns: HttpResponse<Tag[]>

client.tags.getTag({ identifier }, venue?)
// returns: HttpResponse<Tag>
```

## Bonds Namespace

```typescript
client.bonds.getBonds(params?, venue?)
// returns: HttpResponse<BondMarket[]>
```

## Venue Override

Every method accepts an optional `venue` parameter as the last argument:

```typescript
const markets = await client.markets.getMarkets();
const kalshiMarkets = await client.markets.getMarkets({}, "kalshi");
```

## Error Classes

```typescript
import {
	StructError, // Base error
	HttpError, // HTTP 4xx/5xx — has .status, .statusText, .body, .responseHeaders
	NetworkError, // Connection failed
	TimeoutError, // Request timeout — has .timeout
	WebSocketError, // WebSocket error
	WebSocketClosedError, // WebSocket closed — has .code, .reason
} from "@structbuild/sdk";
```

# API Reference

Complete method reference for all SDK namespaces.

## Markets Namespace

```typescript
client.markets.getMarkets(params?, venue?)
// params: { limit?, offset?, sort_by?, sort_order?, status?, tag?, search? }
// returns: HttpResponse<MarketMetadata[]>

client.markets.getMarket({ conditionId }, venue?)
// returns: HttpResponse<MarketMetadata>

client.markets.getMarketBySlug({ slug }, venue?)
// returns: HttpResponse<MarketMetadata>

client.markets.getMarketMetrics({ conditionId }, venue?)
// returns: HttpResponse<ConditionMetricsResponse>

client.markets.getTrades(params?, venue?)
// params: { conditionId?, limit?, offset?, trader?, side? }
// returns: HttpResponse<Trade[]>

client.markets.getCandlestick({ conditionId, interval }, venue?)
// interval: "1m" | "5m" | "15m" | "1h" | "4h" | "1d"
// returns: HttpResponse<CandlestickResponse>

client.markets.getPositionCandlestick({ positionId, interval }, venue?)
// returns: HttpResponse<CandlestickResponse>

client.markets.getPositionMetrics({ positionId }, venue?)
// returns: HttpResponse<PositionMetricsResponse>

client.markets.getPositionVolumeChart({ positionId }, venue?)
// returns: HttpResponse<PositionVolumeChartResponse>

client.markets.getMarketVolumeChart({ conditionId }, venue?)
// returns: HttpResponse<MarketVolumeChartResponse>
```

## Events Namespace

```typescript
client.events.getEvents(params?, venue?)
// params: { limit?, offset?, sort_by?, sort_order?, status?, tag?, search? }
// returns: HttpResponse<Event[]>

client.events.getEvent({ id }, venue?)
// returns: HttpResponse<Event>

client.events.getEventBySlug({ slug }, venue?)
// returns: HttpResponse<Event>

client.events.getEventMetrics({ eventId }, venue?)
// returns: HttpResponse<EventMetricsResponse>
```

## Trader Namespace

```typescript
client.trader.getPortfolio({ address }, venue?)
// returns: HttpResponse<Portfolio>

client.trader.getPortfolioPositions({ address, limit?, offset? }, venue?)
// returns: HttpResponse<PositionsResponse>

client.trader.getTraderTrades({ address, limit?, offset? }, venue?)
// returns: HttpResponse<TradesResponse>

client.trader.getTraderProfile({ address }, venue?)
// returns: HttpResponse<UserProfile>

client.trader.getTraderProfilesBatch({ addresses }, venue?)
// addresses: comma-separated string
// returns: HttpResponse<Record<string, UserProfile>>

client.trader.getTraderVolumeChart({ address }, venue?)
// returns: HttpResponse<TraderVolumeChartResponse>

client.trader.getTraderPnl({ address }, venue?)
// returns: HttpResponse<TraderPnlSummary>

client.trader.getTraderPositionPnl({ address, limit?, offset? }, venue?)
// returns: HttpResponse<PnlListResponse<TraderPositionPnlEntry>>

client.trader.getTraderMarketPnl({ address, limit?, offset? }, venue?)
// returns: HttpResponse<PnlListResponse<TraderMarketPnlEntry>>

client.trader.getTraderEventPnl({ address, limit?, offset? }, venue?)
// returns: HttpResponse<PnlListResponse<TraderEventPnlEntry>>

client.trader.getTraderPnlCandles({ address }, venue?)
// returns: HttpResponse<PnlCandlesResponse>

client.trader.getGlobalPnl(params?, venue?)
// returns: HttpResponse<GlobalPnlResponse>
```

## Holders Namespace

```typescript
client.holders.getMarketHolders({ conditionId, limit?, offset? }, venue?)
// returns: HttpResponse<MarketHoldersResponse>

client.holders.getEventHolders({ eventSlug, limit?, offset? }, venue?)
// returns: HttpResponse<EventHoldersResponse>

client.holders.getPositionHolders({ positionId, limit?, offset? }, venue?)
// returns: HttpResponse<PositionHoldersResponse>

client.holders.getMarketHoldersHistory({ conditionId }, venue?)
// returns: HttpResponse<HolderHistoryCandle[]>

client.holders.getEventHoldersHistory({ eventSlug }, venue?)
// returns: HttpResponse<HolderHistoryCandle[]>

client.holders.getPositionHoldersHistory({ positionId }, venue?)
// returns: HttpResponse<HolderHistoryCandle[]>
```

## Series Namespace

```typescript
client.series.getSeriesList(venue?)
// returns: HttpResponse<Series[]>

client.series.getSeriesDetail({ identifier }, venue?)
// returns: HttpResponse<SeriesDetail>

client.series.getSeriesEvents({ identifier }, venue?)
// returns: HttpResponse<Event[]>
```

## Search Namespace

```typescript
client.search.search({ query }, venue?)
// returns: HttpResponse<SearchResults>
```

## Tags Namespace

```typescript
client.tags.getTags(venue?)
// returns: HttpResponse<Tag[]>
```

## Bonds Namespace

```typescript
client.bonds.getBonds(params?, venue?)
// returns: HttpResponse<Bond[]>
```

## Scoring Namespace

```typescript
client.scoring.getTraderScores({ address }, venue?)
// returns: HttpResponse<TraderScores>
```

## Response Format

All methods return `HttpResponse<T>`:

```typescript
interface HttpResponse<T> {
  data: T;
  status: number;
  headers: Headers;
}
```

## Venue Override

Every method accepts an optional `venue` parameter as the last argument:

```typescript
// Use client default venue
const markets = await client.markets.getMarkets();

// Override for specific call
const kalshiMarkets = await client.markets.getMarkets({}, "kalshi");
```

## Error Classes

```typescript
import {
  StructError,      // Base error
  HttpError,        // HTTP 4xx/5xx (has .status, .body)
  NetworkError,     // Connection failed
  TimeoutError,     // Request timeout
  WebSocketError,   // WebSocket error
  WebSocketClosedError, // WebSocket closed (has .code, .reason)
} from "@structbuild/sdk";
```

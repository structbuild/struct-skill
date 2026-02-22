# WebSocket API

Real-time trade streaming via `StructWebSocket`.

## Setup

```typescript
import { StructWebSocket } from "@structbuild/sdk";

const ws = new StructWebSocket({
  apiKey: process.env.STRUCT_API_KEY!,
  reconnect: {
    maxRetries: 5,
    initialDelay: 1000,
  },
});
```

## Connection Lifecycle

```typescript
ws.on("connected", () => console.log("Connected"));
ws.on("disconnected", ({ code, reason }) => console.log(`Disconnected: ${code}`));
ws.on("reconnecting", ({ attempt }) => console.log(`Reconnecting attempt ${attempt}`));
ws.on("error", (error) => console.error("Error:", error));

ws.connect();

// Check connection state
console.log(ws.state); // "disconnected" | "connecting" | "connected" | "reconnecting"

// Clean disconnect
ws.disconnect();
```

## Market Subscriptions

Subscribe to trades for specific markets:

```typescript
// By condition ID
ws.subscribeMarket("0x...condition_id");
ws.unsubscribeMarket("0x...condition_id");

// By position ID
ws.subscribeMarketByPosition("position_123");
ws.unsubscribeMarketByPosition("position_123");

// Listen for trades
ws.on("market_trade", (trade) => {
  console.log({
    trader: trade.trader,
    side: trade.side, // 0 = sell, 1 = buy
    usdAmount: trade.usd_amount,
    price: trade.price,
    outcome: trade.outcome,
  });
});
```

## Special Trade Rooms

Monitor high-value or significant trades:

```typescript
// Whale trades (large positions)
ws.subscribeWhaleTrades();
ws.on("whale_trade", (trade) => {
  console.log(`Whale: $${trade.usd_amount} on ${trade.question}`);
});

// Smart money trades (profitable traders)
ws.subscribeSmartMoneyTrades();
ws.on("smart_money_trade", (trade) => {
  console.log(`Smart money score: ${trade.smart_money_score}`);
});

// Insider trades (informed traders)
ws.subscribeInsiderTrades();
ws.on("insider_trade", (trade) => {
  console.log(`Insider score: ${trade.insider_score}`);
});

// Unsubscribe
ws.unsubscribeWhaleTrades();
ws.unsubscribeSmartMoneyTrades();
ws.unsubscribeInsiderTrades();
```

## Wallet Tracking

Monitor trades from specific wallets:

```typescript
ws.trackWallets(["0xabc...", "0xdef..."]);

ws.on("wallet_tracking_alert", (alert) => {
  console.log({
    trader: alert.trader,
    isBuy: alert.is_buy,
    amount: alert.usd_amount,
    market: alert.metadata?.question,
  });
});

ws.untrackWallets(["0xabc..."]);
```

## Condition Tracking

Monitor trades on specific market conditions:

```typescript
ws.trackConditions(["0x...condition1", "0x...condition2"]);

ws.on("conditions_tracking_alert", (trade) => {
  console.log(`Trade on ${trade.condition_id}: $${trade.usd_amount}`);
});

ws.untrackConditions(["0x...condition1"]);
```

## Event Types

| Event | Payload | Description |
|-------|---------|-------------|
| `connected` | `void` | Connection established |
| `disconnected` | `{ code, reason }` | Connection closed |
| `reconnecting` | `{ attempt }` | Reconnection attempt |
| `error` | `Error` | Connection error |
| `market_trade` | `PredictionTrade` | Trade in subscribed market |
| `whale_trade` | `EnrichedPredictionTrade` | Large trade |
| `smart_money_trade` | `EnrichedPredictionTrade` | Profitable trader trade |
| `insider_trade` | `EnrichedPredictionTrade` | Informed trader trade |
| `wallet_tracking_alert` | `PredictionWalletTrackingAlert` | Tracked wallet activity |
| `conditions_tracking_alert` | `PredictionTrade` | Tracked condition activity |

## Trade Payload Fields

```typescript
interface PredictionTrade {
  id: string;
  hash: string;
  trader: string;        // wallet address
  side: number;          // 0 = sell, 1 = buy
  condition_id: string;
  position_id: string;
  outcome: string;
  question: string;
  slug: string;
  usd_amount: string;
  shares_amount: string;
  price: number;
  probability: number;
  confirmed_at: number;  // unix timestamp
}

interface EnrichedPredictionTrade extends PredictionTrade {
  smart_money_score: number;
  insider_score: number;
  categories: string[];
  image_url: string;
}
```

## Complete Example

```typescript
import { StructWebSocket } from "@structbuild/sdk";

const ws = new StructWebSocket({
  apiKey: process.env.STRUCT_API_KEY!,
});

ws.on("connected", () => {
  console.log("Connected to Struct WebSocket");
  
  ws.subscribeWhaleTrades();
  ws.subscribeSmartMoneyTrades();
  ws.trackWallets(["0x...known_trader"]);
});

ws.on("whale_trade", (trade) => {
  const action = trade.side === 1 ? "bought" : "sold";
  console.log(`🐋 Whale ${action} $${trade.usd_amount} of "${trade.outcome}" @ ${trade.price}`);
});

ws.on("smart_money_trade", (trade) => {
  console.log(`🧠 Smart money (score: ${trade.smart_money_score}) on ${trade.question}`);
});

ws.on("wallet_tracking_alert", (alert) => {
  console.log(`👀 Tracked wallet ${alert.trader} traded on ${alert.metadata?.question}`);
});

ws.on("disconnected", ({ code, reason }) => {
  console.log(`Disconnected: ${code} - ${reason}`);
});

ws.connect();
```

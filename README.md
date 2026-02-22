# Struct SDK Skill

An agent skill for integrating [@structbuild/sdk](https://www.npmjs.com/package/@structbuild/sdk) - the TypeScript SDK for prediction market data via [api.struct.to](https://api.struct.to).

## Installation

### Via npx (skills.sh)

```bash
npx skills install struct-sdk-skill
```

### Manual Installation

Clone to your skills directory:

```bash
# Personal (all projects)
git clone https://github.com/structbuild/struct-sdk-skill ~/.cursor/skills/struct-sdk

# Project-specific
git clone https://github.com/structbuild/struct-sdk-skill .cursor/skills/struct-sdk
```

## What This Skill Provides

This skill helps AI agents integrate the Struct SDK into your projects:

- **REST API integration** - Markets, events, traders, holders, and more
- **WebSocket real-time feeds** - Trade streaming, whale alerts, wallet tracking
- **Pagination helpers** - Iterate through large datasets
- **Error handling** - Typed errors for robust applications
- **TypeScript types** - Full type safety from OpenAPI spec

## Skill Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill file with quick start and common patterns |
| `websocket.md` | Real-time WebSocket API reference |
| `api-reference.md` | Complete method reference for all namespaces |

## When the Agent Uses This Skill

The agent will automatically apply this skill when:

- Building apps that use prediction market data
- Integrating with Struct API or Polymarket
- Setting up real-time trade feeds
- Working with market/event/trader data

## Quick Example

```typescript
import { StructClient, StructWebSocket } from "@structbuild/sdk";

// REST API
const client = new StructClient({ apiKey: process.env.STRUCT_API_KEY! });
const markets = await client.markets.getMarkets({ limit: 10 });

// WebSocket
const ws = new StructWebSocket({ apiKey: process.env.STRUCT_API_KEY! });
ws.on("whale_trade", (trade) => console.log(`Whale: $${trade.usd_amount}`));
ws.connect();
ws.subscribeWhaleTrades();
```

## License

MIT

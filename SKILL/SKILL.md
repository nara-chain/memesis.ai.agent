---
name: memesis-mcp
description: Solana meme token launchpad and trading platform MCP Server. Provides 31 tools for token creation, buy/sell trading, market analysis, portfolio management, smart money tracking, K-line technical indicators, and more. Use this skill when users need to operate Solana meme tokens — including launching new tokens, trading, checking prices, analyzing holdings, tracking whales/smart money, etc.
version: 2.1.0
protocol: MCP 2024-11-05
transport: HTTP+JSON (Streamable HTTP)
auth: API Key / JWT
---

# Memesis MCP Server Skill

Once connected to the Memesis.ai MCP Gateway, you gain full operational capabilities over the Solana meme token ecosystem.

## Capabilities Overview

| Category | Tools | Description |
|----------|-------|-------------|
| **Token Launch** | `launch_token`, `estimate_launch_fee` | Create and launch new meme tokens |
| **Trading** | `buy_token`, `sell_token`, `send_signed_transaction` | Build tx → sign → submit on-chain |
| **Quotes** | `get_buy_quote`, `get_sell_quote` | Pre-trade price and slippage estimates |
| **Token Info** | `get_token_info`, `get_token_graduation`, `list_tokens` | Token details, graduation progress |
| **Market Data** | `get_price`, `get_klines`, `get_technical_indicators` | Price, K-lines, RSI/MACD/Bollinger Bands |
| **Trade History** | `get_trade_history`, `get_wallet_trades`, `get_recent_trades` | Token/wallet/platform-wide trade history |
| **Holder Analysis** | `get_holders`, `get_holder_distribution`, `get_whales` | Holder list, concentration, whales |
| **Smart Money** | `get_token_smart_money`, `get_smart_money_wallets` | Smart money holdings and wallet tracking |
| **Portfolio** | `get_portfolio`, `get_portfolio_holding`, `get_portfolio_performance` | Total holdings, per-token positions, PnL |
| **Wallet Labels** | `get_wallet_labels` | Identify wallets: smart money/whale/sniper/KOL/bot |
| **Liquidity** | `get_pool_info`, `get_pool_reserves` | Liquidity pool info and reserves |
| **Market Scan** | `market_scan` | Trending/new/graduating/smart money flow |
| **Platform Stats** | `get_platform_stats` | Platform-wide volume, active tokens, users |
| **AI Annotation** | `set_trade_ai_reason` | Annotate trades with AI decision reasoning |
| **Auth** | `wallet_auth` | Wallet signature auth to obtain JWT |
| **Config** | `get_rpc_config` | Get Solana RPC endpoint |
| **WebSocket** | `subscribe_ws_channels`, `get_ws_status` | Real-time event subscriptions |

## Connection

The MCP Gateway uses **HTTP+JSON** transport (not stdio), compliant with the MCP 2024-11-05 standard.

### Endpoints

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/mcp/info` | GET | No | Server info and capabilities |
| `/mcp/tools` | GET | No | List all available tools |
| `/mcp/tools/call` | POST | **Yes** | Execute a tool |
| `/mcp/resources` | GET | No | List available resources |
| `/mcp/resources/read` | POST | **Yes** | Read a resource |
| `/mcp/prompts` | GET | No | List prompt templates |
| `/mcp/prompts/get` | POST | No | Get prompt content |
| `/mcp/ws` | WebSocket | **Yes** | Real-time event stream |
| `/health` | GET | No | Health check |

### Authentication

All write operations and sensitive queries require authentication. Two methods:

```
# API Key (recommended for Agents)
Authorization: ApiKey <your-api-key>

# JWT Token (obtained after wallet signature)
Authorization: Bearer <jwt-token>
```

### Tool Call Example

```json
POST /mcp/tools/call
Authorization: ApiKey sk-xxxx

{
  "name": "get_price",
  "arguments": {
    "token_address": "So11111111111111111111111111111111111111112"
  }
}
```

Response:
```json
{
  "content": [
    {
      "type": "text",
      "text": "{\"price_sol\": 0.00042, \"price_usd\": 0.063, \"change_24h\": 12.5, \"volume_24h\": 150000}"
    }
  ],
  "is_error": false
}
```

## Resources

| URI | Description |
|-----|-------------|
| `memesis://market/overview` | Overall market status |
| `memesis://token/{address}/summary` | Full token summary |
| `memesis://portfolio/{wallet}` | Wallet portfolio overview |
| `memesis://smart-money/signals` | Recent smart money signals |

## Prompts

| Name | Description | Arguments |
|------|-------------|-----------|
| `trading_assistant` | Trading assistant | `wallet_address` (required), `risk_level` (optional) |
| `market_analyst` | Market analyst | `token_address` (optional) |
| `launch_advisor` | Token launch advisor | `token_concept` (optional) |

## Trading Flow

Operations involving SOL transfers (buy/sell/launch) use a two-step flow:

1. **Build transaction**: Call `buy_token` etc. → returns Base64 unsigned transaction
2. **Sign & submit**: Agent signs with wallet private key → calls `send_signed_transaction` to submit on-chain

```
buy_token(token, 0.5 SOL) → { unsigned_tx: "base64..." }
                                    ↓ client-side signing
send_signed_transaction(signed_tx) → { signature: "5abc...", status: "confirmed" }
```

## Typical Use Cases

### 1. Analyze and Trade
```
1. market_scan(filter="trending")         — Find trending tokens
2. get_token_info(token)                  — View details
3. get_technical_indicators(token)         — Technical analysis
4. get_holder_distribution(token)          — Holder concentration
5. get_buy_quote(token, 0.5)              — Get quote
6. buy_token(token, 0.5)                  — Buy
7. send_signed_transaction(signed_tx)      — Submit
```

### 2. Smart Money Tracking
```
1. get_smart_money_wallets()              — Get smart money list
2. get_wallet_trades(whale_address)        — View their trades
3. get_wallet_labels(address)              — Confirm wallet labels
4. get_token_smart_money(token)            — View smart money holdings
```

### 3. Launch a New Token
```
1. estimate_launch_fee(initial_buy=1.0)    — Estimate fees
2. launch_token(name, symbol, desc)        — Create token
3. send_signed_transaction(signed_tx)      — Submit on-chain
4. get_token_info(new_token)               — Confirm live
```

## Notes

- All addresses are in Solana Base58 format
- Amounts are in SOL (not lamports)
- Slippage is in basis points (100 = 1%)
- K-line intervals: `1m`, `5m`, `15m`, `1h`, `4h`, `1d`
- Rate limit: 60 RPM by default
- WebSocket channel format: `global.trades`, `token.<mint>.trades`, etc.

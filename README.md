# Memesis AI Agent

A lightweight, self-contained AI-powered trading agent for the **Memesis.ai** Solana meme-token launch platform.  
Ships as a single binary (~500 KB after PyInstaller packaging) — no Electron, no Docker required.

---

## Table of Contents

- [Key Features](#key-features)
- [Architecture](#architecture)
- [Installation](#installation)
  - [Download Pre-built Binary](#download-pre-built-binary)
  - [Verify Integrity](#verify-integrity)
  - [Platform Notes](#platform-notes)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
  - [MCP Gateway](#mcp-gateway)
  - [LLM Provider](#llm-provider)
  - [Telegram Bot](#telegram-bot)
  - [Trading Parameters](#trading-parameters)
  - [Risk Management](#risk-management)
  - [Server](#server)
  - [Environment Variable Overrides](#environment-variable-overrides)
- [Web UI](#web-ui)
- [MCP Tools Reference](#mcp-tools-reference)
  - [Token Launch](#token-launch)
  - [Token Information](#token-information)
  - [Trading](#trading)
  - [Trade History](#trade-history)
  - [Market Data](#market-data)
  - [Portfolio Management](#portfolio-management)
  - [On-Chain Analytics](#on-chain-analytics)
  - [Authentication & Config](#authentication--config)
  - [Task Management](#task-management)
- [Automated Task Scheduler](#automated-task-scheduler)
  - [Supported Task Types](#supported-task-types)
  - [Task Lifecycle](#task-lifecycle)
- [Trading Strategies](#trading-strategies)
- [Wallet Management](#wallet-management)
- [Telegram Bot Integration](#telegram-bot-integration)
- [Auto-Update](#auto-update)
- [FAQ](#faq)
- [License](#license)

---

## Key Features

| Category | Details |
|---|---|
| **LLM-Driven** | Natural language → LLM intent parsing → MCP tool orchestration → result summary |
| **Multi-Provider LLM** | OpenAI (GPT-4o), Anthropic (Claude 3.5 Sonnet), DeepSeek, Ollama / vLLM local models |
| **31 MCP Tools** | Token launch, buy/sell, market data, portfolio, on-chain analytics, task management |
| **16 Automated Tasks** | DCA, stop-loss, take-profit, trailing stop, grid trading, smart-money follow, and more |
| **Secure Local Wallet** | Ed25519 keypair, AES-256-GCM encrypted private key storage, client-side transaction signing |
| **Telegram Bot** | Optional remote control via Telegram commands and natural language messages |
| **Web UI** | Flask + Vue 3 (CDN) — configuration, chat, portfolio, task dashboard, log viewer |
| **Auto-Update** | Checks GitHub Releases for new versions, SHA256 verification, self-replace + restart |
| **Cross-Platform** | Linux x86_64 / ARM64, macOS ARM64 (Intel via Rosetta 2), Windows x86_64 |

---

## Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│               Memesis AI Agent Client  (single process)            │
│                                                                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
│  │ Flask Web  │  │ LLM Engine │  │ MCP Client │  │ Telegram   │  │
│  │ Server     │  │            │  │            │  │ Bot        │  │
│  │ Port 5678  │  │ OpenAI /   │  │ 31 Tools   │  │ (optional) │  │
│  │            │  │ Claude /   │  │ 4 Resources│  │            │  │
│  │            │  │ Local      │  │            │  │            │  │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  └──────┬─────┘  │
│        │               │               │                │         │
│  ┌─────▼───────────────▼───────────────▼────────────────▼──────┐  │
│  │                      Agent Core                              │  │
│  │  User input → LLM → MCP tool orchestration → result summary │  │
│  │  Task scheduler (APScheduler + WebSocket-driven)             │  │
│  │  Wallet manager (keypair / encrypt / sign / derive)          │  │
│  └──────────────────────┬───────────────────────────────────────┘  │
│                         │                                          │
└─────────────────────────┼──────────────────────────────────────────┘
                          │  HTTP / JSON  (MCP Protocol)
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│              MCP Gateway (Rust / Axum) — Port 8081                  │
│              31 Tools · 4 Resources · 3 Prompts                     │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│              Backend API Server — Port 8080                         │
│              Auth · Trading · Market Data · On-chain                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Installation

### Download Pre-built Binary

Go to the [**Releases**](https://github.com/nara-chain/memesis.ai.agent/releases) page, download the binary matching your platform:

| Platform | File |
|---|---|
| Linux x86_64 | `memesis-agent-linux-x86_64` |
| Linux ARM64 | `memesis-agent-linux-aarch64` |
| macOS ARM64 (Apple Silicon) | `memesis-agent-darwin-arm64` |
| Windows x86_64 | `memesis-agent-windows-x86_64.exe` |

> **macOS Intel users**: download the `darwin-arm64` build — it runs via Rosetta 2.

### Verify Integrity

Each binary ships with a `.sha256` checksum file. Verify before running:

```bash
# Linux / macOS
sha256sum -c memesis-agent-linux-x86_64.sha256

# macOS alternative
shasum -a 256 -c memesis-agent-darwin-arm64.sha256

# Windows (PowerShell)
$expected = (Get-Content .\memesis-agent-windows-x86_64.exe.sha256).Split(" ")[0]
$actual   = (Get-FileHash .\memesis-agent-windows-x86_64.exe -Algorithm SHA256).Hash.ToLower()
if ($expected -eq $actual) { "OK" } else { "MISMATCH" }
```

### Platform Notes

| OS | Notes |
|---|---|
| **Linux** | `chmod +x memesis-agent-linux-*` before running. glibc ≥ 2.31 required. |
| **macOS** | On first launch, macOS Gatekeeper may block it. Go to **System Preferences → Security & Privacy** and click **Open Anyway**, or run: `xattr -d com.apple.quarantine memesis-agent-darwin-arm64` |
| **Windows** | No installation required. Double-click or run from PowerShell. Windows Defender SmartScreen may prompt — click **More info → Run anyway**. |

---

## Quick Start

1. **Download & make executable** (Linux/macOS):
   ```bash
   chmod +x memesis-agent-linux-x86_64
   ```

2. **Run**:
   ```bash
   ./memesis-agent-linux-x86_64
   ```

3. **Open the Web UI** at [http://localhost:5678](http://localhost:5678).

4. **Configure** via the Web UI config panel:
   - Set the **MCP Gateway URL** (e.g., `https://mcp.memesis.ai`)
   - Enter your **LLM API key** (OpenAI / Anthropic / DeepSeek)
   - Create or import a **wallet**

5. **Start chatting**: type natural language commands in the chat panel.
   - "Buy 0.5 SOL of PEPE"
   - "Show me trending tokens"
   - "Set up a DCA: buy 0.1 SOL of AI every hour"

---

## Configuration

On first run, the agent creates a `data/config.yaml` file. Edit it directly or use the Web UI config panel.

```yaml
# data/config.yaml

mcp:
  gateway_url: "http://localhost:8081"
  api_key: ""
  auth_type: "api_key"        # api_key | bearer
  timeout: 30

llm:
  provider: "openai"           # openai | anthropic | deepseek | ollama
  api_key: ""
  model: ""                    # leave empty for provider default
  base_url: ""                 # custom endpoint for ollama / vLLM
  temperature: 0.7
  max_tokens: 4096

telegram:
  enabled: false
  bot_token: ""
  allowed_users: []            # Telegram user ID whitelist

trading:
  slippage_bps: 300            # 300 = 3%
  priority_fee_lamports: 100000
  confirm_threshold_sol: 0.1   # ask confirmation above this amount
  max_daily_trades: 50
  auto_sign: false             # auto-sign transactions without confirmation

risk:
  max_position_sol: 5.0        # max SOL in a single position
  max_daily_loss_sol: 2.0
  stop_loss_pct: 30
  take_profit_pct: 100

server:
  host: "0.0.0.0"
  port: 5678
  debug: false
```

### MCP Gateway

| Key | Type | Default | Description |
|---|---|---|---|
| `gateway_url` | string | `http://localhost:8081` | URL of the MCP Gateway server |
| `api_key` | string | `""` | API key or JWT token for authentication |
| `auth_type` | string | `api_key` | Authentication type: `api_key` or `bearer` |
| `timeout` | int | `30` | HTTP request timeout in seconds |

### LLM Provider

| Key | Type | Default | Description |
|---|---|---|---|
| `provider` | string | `openai` | `openai`, `anthropic`, `deepseek`, or `ollama` |
| `api_key` | string | `""` | API key for the chosen provider |
| `model` | string | `""` | Model name (empty = provider default) |
| `base_url` | string | `""` | Custom endpoint URL (required for `ollama` / vLLM) |
| `temperature` | float | `0.7` | LLM response randomness (0.0–1.0) |
| `max_tokens` | int | `4096` | Maximum tokens per LLM response |

**Provider Details:**
- **OpenAI**: GPT-4o (default), GPT-4o-mini. Supports Function Calling and Responses API.
- **Anthropic**: Claude 3.5 Sonnet (default). Uses Anthropic Tool Use format.
- **DeepSeek**: DeepSeek Chat. OpenAI-compatible endpoint.
- **Ollama / vLLM**: Set `base_url` to your local server (e.g., `http://localhost:11434`). Any model with function calling support.

### Telegram Bot

| Key | Type | Default | Description |
|---|---|---|---|
| `enabled` | bool | `false` | Enable Telegram bot integration |
| `bot_token` | string | `""` | Bot token from [@BotFather](https://t.me/BotFather) |
| `allowed_users` | list[int] | `[]` | Telegram user IDs allowed to interact (whitelist) |

### Trading Parameters

| Key | Type | Default | Description |
|---|---|---|---|
| `slippage_bps` | int | `300` | Slippage tolerance in basis points (300 = 3%) |
| `priority_fee_lamports` | int | `100000` | Solana priority fee in lamports |
| `confirm_threshold_sol` | float | `0.1` | Ask for user confirmation when trade exceeds this SOL amount |
| `max_daily_trades` | int | `50` | Maximum number of trades per day |
| `auto_sign` | bool | `false` | Auto-sign and submit transactions without manual confirmation |

### Risk Management

| Key | Type | Default | Description |
|---|---|---|---|
| `max_position_sol` | float | `5.0` | Maximum SOL value in a single token position |
| `max_daily_loss_sol` | float | `2.0` | Maximum daily loss in SOL before trading is paused |
| `stop_loss_pct` | int | `30` | Default stop-loss percentage |
| `take_profit_pct` | int | `100` | Default take-profit percentage |

### Server

| Key | Type | Default | Description |
|---|---|---|---|
| `host` | string | `0.0.0.0` | Web server listen address |
| `port` | int | `5678` | Web server port |
| `debug` | bool | `false` | Enable Flask debug mode |

### Environment Variable Overrides

All configuration values can be overridden via environment variables using the `MEMESIS_` prefix:

```bash
export MEMESIS_LLM__PROVIDER=anthropic
export MEMESIS_LLM__API_KEY=sk-ant-xxx
export MEMESIS_MCP__GATEWAY_URL=https://mcp.memesis.ai
export MEMESIS_SERVER__PORT=8080
```

Use double underscores (`__`) to separate nested keys.

---

## Web UI

The built-in Web UI (Vue 3 via CDN) is served at `http://localhost:5678` and includes:

| Page | Description |
|---|---|
| **Chat Panel** | Natural language conversation with the AI agent |
| **Config Panel** | MCP, LLM, wallet, Telegram, trading, and risk settings |
| **Wallet Manager** | Create, import, list, switch, and delete wallets |
| **Task Dashboard** | View, pause, resume, and delete automated tasks |
| **Portfolio View** | Token holdings, total value, P&L |
| **Log Viewer** | Real-time application logs |

---

## MCP Tools Reference

The agent communicates with the Memesis platform through **31 MCP tools** via the MCP Gateway.

### Token Launch

| Tool | Description |
|---|---|
| `launch_token` | Launch a new meme token (name, symbol, description, optional image/social links) |
| `estimate_launch_fee` | Estimate the fee required to launch a new token |

### Token Information

| Tool | Description |
|---|---|
| `get_token_info` | Get detailed info (name, symbol, market cap, liquidity) for a token |
| `get_token_graduation` | Check graduation progress (bonding curve → Raydium DEX) |
| `list_tokens` | List/search tokens with sorting and pagination |
| `get_token_stats` | Get trading volume, holder changes, and other stats |

### Trading

| Tool | Description |
|---|---|
| `buy_token` | Build an unsigned buy transaction (SOL → token) |
| `sell_token` | Build an unsigned sell transaction (token → SOL) |
| `send_signed_transaction` | Submit a signed transaction to the Solana network |
| `get_buy_quote` | Get a buy quote (estimated tokens received, price impact) |
| `get_sell_quote` | Get a sell quote (estimated SOL received, price impact) |

> **Transaction flow**: `buy_token` / `sell_token` → unsigned transaction → agent signs locally → `send_signed_transaction`

### Trade History

| Tool | Description |
|---|---|
| `get_trade_history` | Get trade history for a specific token |
| `get_wallet_trades` | Get trade history for a specific wallet |
| `get_recent_trades` | Get recent trades across the platform |

### Market Data

| Tool | Description |
|---|---|
| `get_price` | Get current price and 24h change |
| `get_klines` | Get OHLCV candlestick data (1m, 5m, 15m, 1h, 4h, 1d) |
| `get_technical_indicators` | Get technical indicators (RSI, MACD, Bollinger Bands) |
| `market_scan` | Discover trending / new / graduating / smart-money tokens |
| `get_platform_stats` | Get platform-wide statistics |
| `get_pool_info` | Get liquidity pool info for a token |
| `get_pool_reserves` | Get pool reserve amounts |

### Portfolio Management

| Tool | Description |
|---|---|
| `get_portfolio` | Get full portfolio (all token holdings + total value) |
| `get_portfolio_holding` | Get holding details for a specific token |
| `get_portfolio_performance` | Get investment performance (P&L, returns) |

### On-Chain Analytics

| Tool | Description |
|---|---|
| `get_holders` | Get token holder list |
| `get_holder_distribution` | Get holder distribution (top 10/50/100 concentration) |
| `get_whales` | Get whale (large holder) list |
| `get_token_smart_money` | Get smart money activity on a token |
| `get_smart_money_wallets` | Get tracked smart money wallets |
| `get_wallet_labels` | Get wallet labels (smart money, whale, sniper, KOL, bot, market maker) |

### Authentication & Config

| Tool | Description |
|---|---|
| `wallet_auth` | Authenticate via wallet signature, get JWT |
| `get_rpc_config` | Get Solana RPC endpoint configuration |

### Task Management

| Tool | Description |
|---|---|
| `create_task` | Create an automated monitoring/trading task |
| `list_tasks` | List all automated tasks and their statuses |
| `get_task_detail` | Get task details and recent execution logs |
| `pause_task` | Pause an automated task |
| `resume_task` | Resume a paused task |
| `delete_task` | Delete an automated task |

---

## Automated Task Scheduler

The agent includes a powerful task scheduler powered by **APScheduler** with **WebSocket-driven** event triggering.

### Supported Task Types

| Type | Description | Key Config |
|---|---|---|
| `dca` | Dollar-Cost Averaging — buy a fixed SOL amount at regular intervals | `token_address`, `amount_sol`, `interval_seconds` |
| `stop_loss` | Sell all/partial holdings when price drops below threshold | `token_address`, `sell_pct`, `stop_loss_pct` |
| `take_profit` | Sell all/partial holdings when price rises above threshold | `token_address`, `sell_pct`, `take_profit_pct` |
| `trailing_stop` | Dynamic stop-loss that moves up with price | `token_address`, `trailing_pct` (e.g., 10 = sell if 10% down from peak) |
| `price_monitor` | Alert when price hits a target | `token_address`, `target_price`, `direction` |
| `new_token_monitor` | Monitor new token launches, auto-evaluate meme score, optionally auto-buy | `meme_keywords`, `meme_score_threshold` (0–10, default 6), `amount_sol` |
| `smart_money_follow` | Copy trades from specified wallets | `wallet_addresses`, `follow_amount_sol` |
| `graduation_monitor` | Monitor token graduation progress (bonding curve → DEX) | `token_address`, `graduation_threshold` (%, default 80) |
| `whale_alert` | Alert on large-value transactions | `token_address`, `min_sol_amount` |
| `volume_spike_monitor` | Alert on unusual volume | `token_address`, `volume_multiplier` (default 3x) |
| `portfolio_rebalance` | Auto-rebalance portfolio to target allocations | `target_allocations` (dict), `rebalance_threshold_pct` (default 10) |
| `grid_trading` | Grid trading between price bounds | `token_address`, `grid_lower`, `grid_upper`, `grid_count`, `amount_sol` |
| `daily_report` | Send daily portfolio summary | `cron_expression` (e.g., `0 20 * * *` for 8 PM daily) |
| `risk_check` | Periodic risk assessment | `max_position_pct`, `max_drawdown_pct` |
| `position_age_alert` | Alert when holding duration exceeds threshold | `max_position_days` (default 7) |
| `strategy_loop` | Run a custom trading strategy on a schedule | `strategy_name`, `interval_seconds` |

### Task Lifecycle

```
Created → Running ⇄ Paused → Deleted
                  ↓
              Completed (max_runs reached)
```

Tasks persist in a local **SQLite** database and survive restarts. Use `list_tasks` and the Web UI Task Dashboard to monitor.

---

## Trading Strategies

The agent supports pluggable trading strategies (under the `strategies/` module):

| Strategy | Description |
|---|---|
| **Momentum** | Buy on strong upward momentum, sell on reversal |
| **Mean Reversion** | Buy when price deviates below average, sell when it returns |
| **DCA** | Dollar-Cost Averaging — time-based regular buys |
| **Grid** | Place buy/sell orders at preset price intervals |
| **Copy Trade** | Mirror smart money wallet activity |

All strategies implement the `BaseStrategy` interface and produce `TradeSignal` objects (`BUY` / `SELL` / `HOLD` with confidence score).

---

## Wallet Management

- **Key Generation**: Ed25519 keypair via the `solders` library
- **Encrypted Storage**: Private keys are encrypted with **AES-256-GCM** using a password-derived key (Scrypt KDF)
- **Client-Side Signing**: Transactions are signed locally — private keys never leave your machine
- **Multi-Wallet**: Create, import, list, switch between, and delete wallets
- **Address Derivation**: Standard Solana address derivation from Ed25519 public keys

Wallet files are stored in `data/wallets/` as encrypted JSON.

> **Security**: Always set a strong password. The agent never transmits your private key.

---

## Telegram Bot Integration

When enabled, the Telegram bot provides remote access to all agent features.

### Commands

| Command | Description |
|---|---|
| `/start` | Welcome message and setup guide |
| `/help` | List available commands |
| `/buy <token> <amount>` | Buy a token |
| `/sell <token> <amount>` | Sell a token |
| `/price <token>` | Get current price |
| `/portfolio` | View your portfolio |
| `/tasks` | List running automated tasks |

### Natural Language

Any text message (not starting with `/`) is forwarded to the Agent for natural language processing. You can interact with the full set of 31 MCP tools via casual conversation.

### Notifications

The bot pushes notifications for:
- Task triggers (stop-loss hit, DCA executed, whale alert, etc.)
- Trade confirmations
- Daily reports

### User Whitelist

Only Telegram users listed in `telegram.allowed_users` can interact with the bot.

---

## Auto-Update

The agent includes a built-in auto-update mechanism:

1. **Periodic Check**: Every 30 minutes (configurable via `UPDATE_CHECK_INTERVAL` env var), the agent checks GitHub Releases for a newer version.
2. **Download**: If a new version is found, the binary for the current platform is downloaded.
3. **Verify**: SHA256 checksum is verified against the `.sha256` file.
4. **Replace & Restart**: The current binary is replaced and the agent restarts automatically.

Override the update check interval:
```bash
export UPDATE_CHECK_INTERVAL=3600   # check every hour
```

---

## FAQ

**Q: Which LLM model do you recommend?**  
A: GPT-4o or Claude 3.5 Sonnet provide the best function-calling accuracy. For local/private deployment, use a model with strong function-calling support via Ollama.

**Q: Can I run the agent without an LLM key?**  
A: You need at least one LLM provider configured. For a free option, run a local model via Ollama.

**Q: Is my private key safe?**  
A: Yes. Private keys are encrypted with AES-256-GCM and stored locally. Transactions are signed client-side. The key is never sent over the network.

**Q: How do I update to a new version?**  
A: The agent auto-updates by default. You can also download the latest binary manually from the Releases page.

**Q: Can I use this on macOS Intel?**  
A: Yes. Download the `darwin-arm64` binary — it runs via Rosetta 2 on Intel Macs.

**Q: How do I run the agent as a background service?**  
A: On Linux, create a systemd service. On macOS, use a LaunchAgent. Example systemd unit:
```ini
[Unit]
Description=Memesis AI Agent
After=network.target

[Service]
Type=simple
ExecStart=/opt/memesis/memesis-agent-linux-x86_64
WorkingDirectory=/opt/memesis
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

## License

Copyright © 2025-2026 Memesis.ai — All rights reserved.

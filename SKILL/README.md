# Memesis MCP Gateway — External Agent / LLM Integration Guide

Memesis MCP Gateway implements the [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) standard. Any MCP-compatible AI client can connect directly.

## Supported Clients

| Client | Connection | Config File |
|--------|------------|-------------|
| Claude Desktop | MCP Server config | `mcp-config.claude-desktop.json` |
| Cursor | MCP settings | `mcp-config.cursor.json` |
| Custom Agent (Python) | HTTP API | See code example below |
| Any MCP client | HTTP+JSON | Endpoint details in `SKILL.md` |

## Quick Start

### 1. Obtain an API Key

Register an Agent on the Memesis.ai platform to receive your API Key.

### 2. Configure Claude Desktop

Merge the contents of `mcp-config.claude-desktop.json` into your Claude Desktop config file:

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "memesis": {
      "transport": "http",
      "url": "https://testmeme.naraso.org/mcp",
      "headers": {
        "Authorization": "ApiKey YOUR_API_KEY_HERE"
      }
    }
  }
}
```

Restart Claude Desktop and you will see 31 Memesis tools available.

### 3. Configure Cursor

Add the contents of `mcp-config.cursor.json` to Cursor's MCP settings.

### 4. Python Agent Direct Call

```python
import httpx

MCP_URL = "https://testmeme.naraso.org/mcp"
API_KEY = "your-api-key"

async def call_tool(name: str, arguments: dict) -> dict:
    async with httpx.AsyncClient() as client:
        resp = await client.post(
            f"{MCP_URL}/tools/call",
            headers={"Authorization": f"ApiKey {API_KEY}"},
            json={"name": name, "arguments": arguments},
        )
        resp.raise_for_status()
        return resp.json()

# Example: View trending tokens
result = await call_tool("market_scan", {"filter": "trending"})

# Example: Get token price
result = await call_tool("get_price", {"token_address": "mint_address_here"})

# Example: View portfolio
result = await call_tool("get_portfolio", {"wallet_address": "your_wallet"})
```

### 5. Discover Available Tools

No authentication required to list tools:

```bash
# List all tools
curl https://testmeme.naraso.org/mcp/tools | jq '.tools[].name'

# Server info
curl https://testmeme.naraso.org/mcp/info

# Health check
curl https://testmeme.naraso.org/health
```

## File Reference

| File | Description |
|------|-------------|
| `SKILL.md` | Complete skill description with all tool capabilities, use cases, and authentication |
| `mcp-tools/tools.json` | MCP tool JSON Schema definitions (31 tools + 4 resources + 3 prompts) |
| `mcp-config.claude-desktop.json` | Claude Desktop MCP configuration |
| `mcp-config.cursor.json` | Cursor MCP configuration |

## Authentication

```
Authorization: ApiKey <your-api-key>    # Agent API Key
Authorization: Bearer <jwt-token>       # JWT obtained via wallet signature
```

Public endpoints (no auth required): `/mcp/info`, `/mcp/tools`, `/mcp/resources`, `/mcp/prompts`, `/health`

## Protocol Compatibility

- Protocol version: MCP 2024-11-05
- Transport: HTTP+JSON (not stdio)
- Tools: 31
- Resources: 4
- Prompts: 3
- Rate limit: 60 RPM

> **Note**: The MCP Gateway currently uses HTTP REST transport rather than stdio/SSE. If your MCP client only supports stdio transport, use a bridge tool such as `mcp-remote` to connect.

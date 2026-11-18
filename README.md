# NoctHost MCP server

Create, list, reboot and destroy cloud servers from an AI assistant.

NoctHost gives you one account, one prepaid balance and one API for servers on Hetzner, Vultr, DigitalOcean, Linode and OVHcloud, plus bare metal on Cherry Servers, including NVIDIA GPU servers. Servers are billed by the hour. This is the configuration for its hosted MCP server.

- Endpoint: `https://mcp.nocthost.com/mcp` (Streamable HTTP)
- Auth: `Authorization: Bearer nh_...` (API key from the dashboard: Profile, API keys)
- Docs: https://nocthost.com/docs/mcp
- Official MCP Registry name: `com.nocthost/mcp`

The server is hosted, so there is nothing to install. This repository holds the client configuration and the registry manifest.

## Connect

**Claude Code**

```bash
claude mcp add --transport http nocthost https://mcp.nocthost.com/mcp \
  --header "Authorization: Bearer nh_your_key"
```

**Cursor** (`~/.cursor/mcp.json`)

```json
{
  "mcpServers": {
    "nocthost": {
      "url": "https://mcp.nocthost.com/mcp",
      "headers": { "Authorization": "Bearer nh_your_key" }
    }
  }
}
```

**VS Code** (`.vscode/mcp.json`, asks for the key once)

```json
{
  "inputs": [
    { "type": "promptString", "id": "nocthost-key", "description": "NoctHost API key", "password": true }
  ],
  "servers": {
    "nocthost": {
      "type": "http",
      "url": "https://mcp.nocthost.com/mcp",
      "headers": { "Authorization": "Bearer ${input:nocthost-key}" }
    }
  }
}
```

**Claude Desktop** (`claude_desktop_config.json`, through the `mcp-remote` bridge, needs Node.js)

```json
{
  "mcpServers": {
    "nocthost": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.nocthost.com/mcp", "--header", "Authorization:${NOCTHOST_AUTH}"],
      "env": { "NOCTHOST_AUTH": "Bearer nh_your_key" }
    }
  }
}
```

**Any other MCP client:** the URL above, Streamable HTTP transport, and the `Authorization` header.

## API key scopes

When you create a key you pick what it may do:

- `read`: balance, sizes, prices and servers
- `deploy`: create servers, reboot, stop, start
- `destroy`: delete servers

A key without `destroy` is a good default for an assistant: it can build, and you delete from the dashboard.

## Tools

| Tool | What it does |
|---|---|
| `get_account` | Balance, hourly spend, how long the balance lasts, servers and the account limit |
| `list_options` | Sizes (shared vCPU, dedicated CPU, GPU, bare metal) with specs, hourly and monthly price, clouds and cities. Filters by city, country, cloud, kind, GPU, minimum vCPU, RAM or disk, maximum hourly price |
| `quote` | Price of a plan in a city for a number of servers and hours, and whether the balance covers the first hour |
| `create_server` | Creates 1 to 10 servers, waits for them to become active, returns ids, IPv4 and SSH commands |
| `list_servers` | Servers on the account with status and IPs |
| `get_server` | One server in detail |
| `server_power` | Reboot, stop or start |
| `destroy_server` | Deletes a server |

There is also a resource, `nocthost://catalog` (clouds, cities, plans, OS images), and a prompt, `deploy-a-server`.

## Money and safety

- One `create_server` call makes at most 3 servers and at most $1 per hour in total unless a higher `max_hourly_usd` is passed. Above that the tool returns the price and does not create anything.
- `destroy_server` needs `confirm: true` and the exact server label. Without both it only describes what it would delete.
- Give the assistant your SSH public key. Without one, a random root password is generated and returned once.

## Example prompts

- "Spin up a small Ubuntu server in Frankfurt for an hour, here is my key: ssh-ed25519 AAAA..."
- "What is the cheapest GPU I can get in Europe, and how long does my balance last with it?"
- "List my servers and destroy the ones labelled test."

## Links

- Docs: https://nocthost.com/docs/mcp
- REST API: https://nocthost.com/docs/api
- Guide for AI agents: https://nocthost.com/llms.txt
- Website: https://nocthost.com

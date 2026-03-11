# Playwright MCP Setup

Instructions for installing Playwright MCP to enable automated design verification.

## Option 1: Via NPM (Recommended)

```bash
npm install -g @playwright/mcp-server
claude code mcp add playwright
```

## Option 2: Manual Configuration

Create or update your MCP configuration file:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp-server"],
      "env": {}
    }
  }
}
```

Then restart Claude Code.

## Verification

After installation, confirm Playwright MCP is working:

```bash
claude code mcp list
```

You should see `playwright` in the list of available MCP servers.

# bc365-skills

Claude Code skills for [Microsoft Dynamics 365 Business Central](https://dynamics.microsoft.com/business-central/overview/).

Works with the [bc365 CLI](https://www.npmjs.com/package/@habitusnet/bc365) and its two MCP servers:
- **bc-data** — query and update BC data (customers, items, orders, G/L entries, …)
- **bc-admin** — manage environments, companies, apps, and permissions

## Install

```bash
claude plugin install habitusnet/bc365-skills
```

## Skills

| Skill | When to use |
|-------|-------------|
| `bc-query` | Ask Claude to list, filter, or update BC data via the bc-data MCP server |
| `bc-admin` | Ask Claude to inspect environments, companies, apps, and sessions via bc-admin |
| `bc-diagnose` | Ask Claude to explain BC error messages and suggest fixes |

## Setup

1. Install the [bc365 CLI](https://www.npmjs.com/package/@habitusnet/bc365):
   ```bash
   npm install -g @habitusnet/bc365
   ```

2. Authenticate and generate `.mcp.json` in your project root:
   ```bash
   bc365 onboard
   ```
   This creates `.mcp.json` with `bc-data` and `bc-admin` MCP server configuration.

3. Install these skills:
   ```bash
   claude plugin install habitusnet/bc365-skills
   ```

The skills use the MCP servers configured in `.mcp.json`. Both must be reachable for the skills to work. See the [mcp-d365-BC repository](https://github.com/habitusnet/mcp-d365-BC) for full documentation.

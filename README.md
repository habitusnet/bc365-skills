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

Run `bc365 onboard` first to generate `.mcp.json` with the MCP server configuration.

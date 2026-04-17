# AMG skills

Agent plugin for [Azure Managed Grafana](https://learn.microsoft.com/en-us/azure/managed-grafana/) — health checks, cost analysis, and diagnostics for Azure resources via the built-in AMG-MCP server. Ships in two formats from one repo: **Claude Code** and **GitHub Copilot** (VS Code + CLI).

## Prerequisites

- An **Azure Managed Grafana** instance with the [MCP server enabled](https://learn.microsoft.com/en-us/azure/managed-grafana/grafana-mcp-server)
- One of the supported clients below (Claude Code, VS Code with GitHub Copilot, or GitHub Copilot CLI)

Your Grafana MCP endpoint: `https://<your-grafana-endpoint>/api/azure-mcp`

## Available Skills

| Skill | Command | Description |
|---|---|---|
| PostgreSQL Flex | `/amg-check-pg-flex 7d` | Fleet-wide health check for Azure PostgreSQL Flexible Servers |
| Cosmos DB MongoDB | `/amg-check-cosmosdb-mongo-ru 7d` | Health check for Cosmos DB for MongoDB (RU) accounts |
| Storage Account | `/amg-check-storage-account 7d` | Health check for Azure Storage Accounts |
| Key Vault | `/amg-check-key-vault 7d` | Health check for Azure Key Vaults |
| Azure Spend | `/amg-check-azure-spend` | Monthly cost analysis for Azure subscriptions |

---

## Claude Code

Bearer-token auth. Requires a [Grafana service account token](https://learn.microsoft.com/en-us/azure/managed-grafana/how-to-service-accounts) or Entra ID token.

### Install

Set the env vars first:

```bash
export AMG_MCP_URL="https://<your-grafana-endpoint>/api/azure-mcp"
export AMG_MCP_TOKEN="<your-service-account-token>"
```

Then add the marketplace and install the plugin:

```
/plugin marketplace add Azure/amg-skills
/plugin install amg-toolkit@amg-skills
```

### Update

```
/plugin marketplace update amg-skills
/plugin install amg-toolkit@amg-skills
```

Updates are manual — Claude Code detects them by comparing the `version` field in `marketplace.json` against the installed version. Verify the installed version with:

```
/plugin list
```

### Uninstall

```
/plugin uninstall amg-toolkit@amg-skills
```

Add `--keep-data` to preserve saved configuration and reports.

---

## GitHub Copilot CLI

OAuth auto-login. No token required — `copilot` opens a browser for you to sign in to the AMG-MCP server once on first use.

### Install

Set the endpoint (URL only — no token):

```bash
export AMG_MCP_URL="https://<your-grafana-endpoint>/api/azure-mcp"
```

Then:

```
copilot plugin marketplace add Azure/amg-skills
copilot plugin install amg-toolkit@amg-skills
```

> **Known issue**: [github/copilot-cli#2709](https://github.com/github/copilot-cli/issues/2709) can prevent the plugin's `.mcp.json` from being auto-merged into `~/.copilot/mcp-config.json`. If `/mcp show amg` returns nothing after install, register the server manually using one of these two fallbacks.
>
> **Fallback A — interactive `/mcp add`** (inside a `copilot` session):
>
> ```
> /mcp add
> ```
>
> Fill in the prompts:
>
> | Prompt | Value |
> |---|---|
> | Server Name | `amg` |
> | Server Type | `HTTP` |
> | URL | `https://<your-grafana-endpoint>/api/azure-mcp` |
> | HTTP Headers | *(leave empty — OAuth handles auth)* |
> | Tools | `*` *(or press Enter for all)* |
>
> Press `Ctrl+S` to save.
>
> **Fallback B — edit `~/.copilot/mcp-config.json` directly**:
>
> ```json
> {
>   "mcpServers": {
>     "amg": {
>       "type": "http",
>       "url": "https://<your-grafana-endpoint>/api/azure-mcp",
>       "headers": {},
>       "tools": ["*"]
>     }
>   }
> }
> ```
>
> Verify with `/mcp show amg` afterward.

### Update

```
copilot plugin marketplace update amg-skills
copilot plugin install amg-toolkit@amg-skills
```

Updates are manual — Copilot detects them by comparing the `version` field in `marketplace.json` against the installed version. Verify the installed version with:

```
copilot plugin list
```

### Uninstall

```
copilot plugin uninstall amg-toolkit@amg-skills
```

---

## GitHub Copilot in VS Code

OAuth auto-login. No token required — VS Code opens a browser for you to sign in to the AMG-MCP server once on first use; the session is cached thereafter.

### Install

Set the endpoint (URL only — no token):

```bash
export AMG_MCP_URL="https://<your-grafana-endpoint>/api/azure-mcp"
```

Then open VS Code, launch the **Chat Customizations** editor, go to **Marketplace**, search for `amg-toolkit`, and click **Install**. VS Code will prompt you to sign in to AMG-MCP on first skill invocation.

Alternatively, run the Copilot CLI commands (see above) from the VS Code integrated terminal.

### Update

In the Chat Customizations editor, open the installed plugins list and click **Update** next to `amg-toolkit`. Or use the Copilot CLI `update` commands from a terminal.

### Uninstall

In the Chat Customizations editor, open the installed plugins list and click **Uninstall**. Or use the Copilot CLI `uninstall` command from a terminal.

---

## How It Works

Each skill queries Azure resources through the AMG-MCP (Model Context Protocol) server built into Azure Managed Grafana. Skills follow a phased workflow:

1. **Validate datasource** — confirm Azure Monitor datasource
2. **Discover resources** — inventory via Azure Resource Graph
3. **Tier 1 scan** — key metrics across the entire fleet
4. **Tier 2 deep dive** — detailed investigation of abnormal resources
5. **Report** — findings with known issue cross-reference

On first run, each skill auto-discovers the Azure Monitor datasource UID and prompts for the subscription ID(s) to scan. Configuration is saved to `memory/<skill-name>/config.md`.

## License

See [LICENSE](LICENSE).

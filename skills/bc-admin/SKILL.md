---
name: bc-admin
description: Manage Microsoft Dynamics 365 Business Central environments, companies, installed apps, feature flags, and user sessions using the bc-admin MCP server. Use when the user asks about environment status, app updates, permissions, or wants to perform admin operations.
allowed-tools: ["mcp__bc-admin__get_environment_informations", "mcp__bc-admin__get_environment_companies", "mcp__bc-admin__get_installed_apps", "mcp__bc-admin__get_available_app_updates", "mcp__bc-admin__update_app", "mcp__bc-admin__get_available_features", "mcp__bc-admin__activate_feature", "mcp__bc-admin__deactivate_feature", "mcp__bc-admin__get_active_sessions", "mcp__bc-admin__kill_active_sessions", "mcp__bc-admin__get_usage_storage_for_environment", "mcp__bc-admin__get_usage_storage_for_all_environments", "mcp__bc-admin__clear_cached_token", "mcp__bc-admin__get_token_cache_status", "mcp__bc-admin__get_extension_deployment_status", "mcp__bc-admin__get_app_operations"]
---

# BC Admin Skill

Manage Business Central environments and tenant operations via the `bc-admin` MCP server.

## Prerequisites

The `bc-admin` MCP server must be configured in `.mcp.json` (run `bc365 onboard` if not). The `BC_TENANT_ID` environment variable must be set to the tenant's Entra ID GUID.

## Tool Reference

### Environment & Company

| Tool | Purpose |
|------|---------|
| `mcp__bc-admin__get_environment_informations()` | List all environments for the tenant |
| `mcp__bc-admin__get_environment_companies(environment)` | List companies in an environment |
| `mcp__bc-admin__get_usage_storage_for_environment(environment)` | Storage usage for one environment |
| `mcp__bc-admin__get_usage_storage_for_all_environments()` | Storage usage across all environments |

### Apps & Extensions

| Tool | Purpose |
|------|---------|
| `mcp__bc-admin__get_installed_apps(environment)` | List installed AL apps/extensions |
| `mcp__bc-admin__get_available_app_updates(environment)` | List apps with pending updates |
| `mcp__bc-admin__update_app(environment, appId, appVersion)` | Update an app to a specific version |
| `mcp__bc-admin__get_extension_deployment_status(environment, operationId)` | Check app deployment status |
| `mcp__bc-admin__get_app_operations(environment)` | List all app operations and their status |

### Feature Flags

| Tool | Purpose |
|------|---------|
| `mcp__bc-admin__get_available_features(environment)` | List all BC feature flags and their state |
| `mcp__bc-admin__activate_feature(environment, feature)` | Enable a feature flag |
| `mcp__bc-admin__deactivate_feature(environment, feature)` | Disable a feature flag |

### Sessions

| Tool | Purpose |
|------|---------|
| `mcp__bc-admin__get_active_sessions(environment)` | List currently active user sessions |
| `mcp__bc-admin__kill_active_sessions(environment)` | Force-disconnect all sessions (e.g. before upgrade) |

### Auth

| Tool | Purpose |
|------|---------|
| `mcp__bc-admin__get_token_cache_status()` | Check if cached token is valid |
| `mcp__bc-admin__clear_cached_token()` | Clear stale auth token (then re-run `bc365 onboard`) |

## Common Workflows

### Inspect environments
```
mcp__bc-admin__get_environment_informations()
# Returns: name, type (Production/Sandbox), applicationVersion, status
```

### List companies in Production
```
mcp__bc-admin__get_environment_companies(environment="Production")
# Returns: id (GUID), name, displayName — use id as BC_COMPANY in .mcp.json
```

### Check for app updates
```
mcp__bc-admin__get_available_app_updates(environment="Production")
# Returns: appId, name, currentVersion, latestVersion
```

### Update an app
```
# 1. Get current installed apps and their IDs
mcp__bc-admin__get_installed_apps(environment="Production")

# 2. Check for available updates
mcp__bc-admin__get_available_app_updates(environment="Production")

# 3. Update (always confirm with user first — this affects a live environment)
mcp__bc-admin__update_app(environment="Production", appId="<guid>", appVersion="<latestVersion>")

# 4. Monitor deployment
mcp__bc-admin__get_extension_deployment_status(environment="Production", operationId="<operationId from step 3>")

# 5. Or check all pending operations
mcp__bc-admin__get_app_operations(environment="Production")
```

### Prepare for maintenance (force logout users)
```
# 1. Check who is logged in
mcp__bc-admin__get_active_sessions(environment="Production")

# 2. Confirm with user before disconnecting — this will interrupt active work
mcp__bc-admin__kill_active_sessions(environment="Production")
```

### Check storage usage
```
mcp__bc-admin__get_usage_storage_for_all_environments()
# Returns: environment name, database size, file size in GB
```

### Manage feature flags
```
# List available features and their current state
mcp__bc-admin__get_available_features(environment="Production")

# Enable a feature (confirm with user — some features cannot be disabled after activation)
mcp__bc-admin__activate_feature(environment="Production", feature="<featureKey>")
```

## Auth Troubleshooting

If bc-admin calls return 401 errors:
```
mcp__bc-admin__get_token_cache_status()
# If expired or missing:
mcp__bc-admin__clear_cached_token()
# Then tell user: run `bc365 onboard` to re-authenticate
```

## Profile & Environment Context

The active tenant is set by the `BC_TENANT_ID` env var in `.mcp.json`. To switch tenant/environment:
```bash
bc365 switch <profile-name>   # updates .mcp.json to a different profile
bc365 profiles                # list saved profiles
```

---
name: bc-diagnose
description: Interpret Microsoft Dynamics 365 Business Central error messages and API failures, and suggest specific fixes. Use when the user receives an HTTP error, OData error, permission error, or unexpected BC behaviour. Cross-references bc-admin skill for permission lookups.
---

# BC Diagnose Skill

Interpret Business Central errors and guide the user to the correct fix.

## HTTP Status Codes

### 401 Unauthorized
**Meaning:** The access token is expired, missing, or invalid.

**Fix steps:**
1. Check token health: `mcp__bc-admin__get_token_cache_status()`
2. Clear the cached token: `mcp__bc-admin__clear_cached_token()`
3. Tell the user: `bc365 logout && bc365 onboard` to re-authenticate

If `bc365 onboard` also fails with 401, the Entra ID app registration may have expired or the user's device code session timed out. Ask the user to run `bc365 onboard` in a terminal with a TTY.

### 403 Forbidden
**Meaning:** Authenticated successfully, but the user lacks a required permission set.

**Fix steps:**
1. List companies via bc-admin to confirm the correct company: `mcp__bc-admin__get_environment_companies(environment="Production")`
2. Common missing permission sets:
   - `D365 BUS FULL ACCESS` — general read/write access
   - `D365 BASIC` — minimum read access
   - `D365 ACCOUNTANT` — finance/G/L access
   - `D365 SALES` — sales access
   - `D365 PURCHASING` — purchasing access
   - `SUPER` — admin only
3. A BC administrator must assign the missing permission in the BC UI (Settings → Users → select user → Permission Sets) — this cannot be done via MCP

### 404 Not Found
**Meaning:** The entity or endpoint does not exist. Most common causes:

| Cause | Symptom | Fix |
|-------|---------|-----|
| Wrong company GUID | 404 on any data request | Run `bc365 profiles`, compare `BC_COMPANY` in `.mcp.json` with actual company ID from `mcp__bc-admin__get_environment_companies` |
| Wrong API version | URL has `/v1.0/` instead of `/v2.0/` | Check `BC_URL_SERVER` in `.mcp.json` — should end in `/api/v2.0` |
| Entity name wrong | 404 on specific entity | Use the exact camelCase entity name (e.g. `salesOrders` not `sales_orders`) — use `bc-query` skill for the full entity list |
| Record deleted | 404 on specific ID | The record no longer exists — list the entity to confirm |

### 429 Too Many Requests
**Meaning:** BC's API rate limit was hit.

**Fix steps:**
1. Wait 30–60 seconds before retrying
2. Reduce query size: add `top=50` to `list_items` calls
3. Avoid broad queries with no filter on large entities (e.g. `generalLedgerEntries` without a date filter)
4. If hitting limits repeatedly, schedule bulk operations during off-peak hours

### 500 Internal Server Error
**Meaning:** BC server-side error. Usually transient.

**Fix steps:**
1. Retry the same operation after 30 seconds
2. If persistent, check BC service health at [aka.ms/bchealth](https://aka.ms/bchealth)
3. Check environment status: `mcp__bc-admin__get_environment_informations()` — look for `status: Updating`

### 503 Service Unavailable
**Meaning:** BC environment is unavailable (often during scheduled maintenance or upgrade).

**Fix steps:**
1. `mcp__bc-admin__get_environment_informations()` — check `status` field
2. If `status: Updating` — wait for upgrade to complete
3. Monitor upgrade progress: `mcp__bc-admin__get_app_operations(environment="Production")`
4. If unexpected downtime, report to Microsoft via BC Admin Center

## OData Errors

### Filter parse error
**Symptom:** `BadRequest` or `An error occurred while reading from the store provider's data reader`

**Cause:** OData `$filter` syntax error.

**Common mistakes:**
- String values must be in single quotes: `displayName eq 'Contoso'` (not double quotes)
- Date values need full ISO format: `postingDate gt 2024-01-01T00:00:00Z`
- Field names are camelCase: `displayName` not `Display Name`
- Use `mcp__bc-data__get_schema(entity="...")` to verify correct field names

### Unauthorized in OData response body
**Cause:** bc-data server uses `azure_cli` auth — `az login` token has expired.

**Fix:** Tell the user to run `az login` to refresh Azure CLI credentials.

### ResourceNotFound in OData response
**Cause:** Company GUID mismatch or entity not available in this BC version.

**Fix:**
1. Verify company: `mcp__bc-admin__get_environment_companies(environment="Production")`
2. Compare with `BC_COMPANY` in `.mcp.json`
3. If mismatch: `bc365 onboard` to regenerate config with the correct company

## Permission Set Reference

| Permission Set | Grants |
|----------------|--------|
| `D365 BUS FULL ACCESS` | Full read/write on business data |
| `D365 BASIC` | Read-only on most business data |
| `D365 ACCOUNTANT` | Full access to finance/G/L |
| `D365 SALES` | Full access to sales |
| `D365 PURCHASING` | Full access to purchasing |
| `SUPER` | Full system access including user management (admin only) |

## Diagnostic Workflow

When a user reports an error, follow this sequence:

1. **Identify the HTTP status code** — use the tables above
2. **Check token health first** (most errors are auth-related):
   ```
   mcp__bc-admin__get_token_cache_status()
   ```
3. **Verify the environment is up**:
   ```
   mcp__bc-admin__get_environment_informations()
   ```
4. **Reproduce with a minimal query** — try this as a baseline:
   ```
   mcp__bc-data__list_items(entity="companies", top=1)
   ```
5. **Verify BC_COMPANY matches** — compare `mcp__bc-admin__get_environment_companies` output with the `BC_COMPANY` value in `.mcp.json`
6. **Check field names** — if the error is filter-related, use `mcp__bc-data__get_schema` to confirm field names

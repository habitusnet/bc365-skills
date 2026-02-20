---
name: bc-query
description: Query and update Microsoft Dynamics 365 Business Central data using the bc-data MCP server. Use when the user asks to list customers, find items, check sales orders, query general ledger entries, or any other BC data operation.
allowed-tools: ["mcp__bc-data__list_items", "mcp__bc-data__get_items_by_field", "mcp__bc-data__get_schema", "mcp__bc-data__create_item", "mcp__bc-data__update_item", "mcp__bc-data__delete_item"]
---

# BC Query Skill

Query and update Business Central data via the `bc-data` MCP server.

## Prerequisites

The `bc-data` MCP server must be configured in `.mcp.json` (run `bc365 onboard` if not). The server uses Azure CLI credentials — run `az login` if you see auth errors.

## Available Tools

| Tool | Purpose |
|------|---------|
| `mcp__bc-data__list_items` | List entities with optional OData filtering |
| `mcp__bc-data__get_items_by_field` | Find records matching a specific field value |
| `mcp__bc-data__get_schema` | Discover available fields for an entity |
| `mcp__bc-data__create_item` | Create a new record |
| `mcp__bc-data__update_item` | Update an existing record by ID |
| `mcp__bc-data__delete_item` | Delete a record by ID |

## Common Entity Names

Use these exact strings as the `entity` parameter:

| Entity | Description |
|--------|-------------|
| `customers` | Customer master records |
| `vendors` | Vendor master records |
| `items` | Item (product/service) master |
| `salesOrders` | Sales order headers |
| `salesInvoices` | Posted sales invoices |
| `purchaseOrders` | Purchase order headers |
| `purchaseInvoices` | Posted purchase invoices |
| `generalLedgerEntries` | G/L ledger entries |
| `accounts` | Chart of accounts |
| `companies` | Companies in this environment |
| `currencies` | Currency codes and exchange rates |
| `paymentTerms` | Payment term codes |

## Querying Patterns

### List all records (with limit)
```
mcp__bc-data__list_items(entity="customers", top=50)
```

### Filter by field value
```
mcp__bc-data__list_items(entity="customers", filter="displayName eq 'Contoso'")
mcp__bc-data__list_items(entity="items", filter="startswith(displayName, 'BIKE')")
mcp__bc-data__list_items(entity="salesOrders", filter="sellToCustomerNumber eq '10000'")
```

### Select specific fields (reduces payload)
```
mcp__bc-data__list_items(entity="customers", select="number,displayName,email,phoneNumber", top=100)
```

### Find by known field value (simpler than filter)
```
mcp__bc-data__get_items_by_field(entity="customers", field="number", value="10000")
mcp__bc-data__get_items_by_field(entity="items", field="number", value="ITEM-001")
```

### Discover available fields before querying
```
mcp__bc-data__get_schema(entity="salesOrders")
```
Always call `get_schema` first when the user asks about a field you haven't seen before.

### Sort results
```
mcp__bc-data__list_items(entity="generalLedgerEntries", filter="postingDate gt 2024-01-01", orderby="postingDate desc", top=200)
```

### Paginate large result sets
If the response contains `@odata.nextLink`, call `list_items` again with a `skip` offset:
```
mcp__bc-data__list_items(entity="generalLedgerEntries", top=200, skip=200)
```

## OData Filter Syntax

BC uses OData v4. Supported operators:
- Equality: `field eq 'value'` or `field eq 12345`
- Comparison: `field gt value`, `field lt value`, `field ge value`, `field le value`
- String: `startswith(field, 'prefix')`, `contains(field, 'substring')`
- Date: `postingDate gt 2024-01-01T00:00:00Z`
- Combine: `field1 eq 'x' and field2 gt 100`

**Common mistake:** Field names are camelCase in the API (`displayName`, `sellToCustomerNumber`), NOT the BC UI names ("Name", "Sell-to Customer No."). Use `get_schema` to find the correct API field name.

## Write Operations (use with care)

Before creating or updating, always call `get_schema` to see required fields and field types.

```
# Create a new customer
mcp__bc-data__create_item(entity="customers", data={"displayName": "New Corp", "currencyCode": "CHF"})

# Update a customer's email
mcp__bc-data__update_item(entity="customers", id="<guid>", data={"email": "new@example.com"})
```

Destructive operations (`delete_item`) require the record's GUID `id`, not its BC number. Always confirm with the user before deleting.

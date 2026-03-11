# Supabase Explorer

A standalone HTML database browser for Supabase — like Supabase Studio, but runs from a single file with no server or dependencies.

## Quick Start

1. Open `supabase-explorer.html` in any browser
2. Enter the Supabase URL (`https://xxxxx.supabase.co`)
3. Enter the API key (anon key, service_role key, or JWT)
4. Click **Connect**

You can also auto-connect via URL params:
```
supabase-explorer.html?url=https://xxxxx.supabase.co&key=eyJ...
```

## Features

### Connection
- **Anon key** — subject to RLS (Row Level Security)
- **Service role key** — bypasses all RLS (full access)
- **JWT token** — authenticated user access (optional field)
- **Connection history** — last 20 connections saved, select from dropdown
- **Auto-reconnect on refresh** — URL, key, JWT, schema all persist in localStorage

### Data Tab
- **Table list** — left sidebar shows all tables with row counts
- **Row counts cached** — counts persist across page refreshes, no re-scanning
- **Paginated grid** — 100 rows per page (configurable: 25, 50, 100, 250, 500, 1000)
- **Sorting** — click any column header to sort ascending/descending
- **Filtering** — PostgREST filter syntax in the filter bar:
  - `email=ilike.%@gmail%` — case-insensitive pattern match
  - `status=eq.active` — exact match
  - `age=gt.18` — greater than
  - `created_at=gt.2025-01-01` — date range
  - `or=(role.eq.admin,role.eq.mod)` — OR conditions
  - `deleted_at=is.null` — null check
- **Cell detail** — click any cell to expand full content (JSON, long text)
- **Column select** — dropdown to show specific columns only
- **Export CSV** — export current page to CSV file
- **Export JSON** — export current page to JSON file

### Schema Switching
Switch between PostgreSQL schemas via the dropdown:

| Schema | What's inside |
|--------|--------------|
| `public` | Application tables (default) |
| `storage` | Buckets, objects, uploads (10 tables) |
| `auth` | Users, sessions, tokens, MFA (19 tables) |
| `vault` | Encrypted secrets (1 table) |
| `_realtime` | Realtime config, JWT secrets (3 tables) |
| `pgsodium` | Encryption key metadata |
| `graphql_public` | GraphQL functions |

### RPCs Tab
- Lists all RPC (stored procedure) functions from the schema
- Select any RPC, enter parameters as JSON
- Execute and see the response with HTTP status code

### Auth Tab
- **Auth Settings** — view provider config, signup policy, auto-confirm status
- **Schema Probe** — test all 10 schemas in one click (accessible vs blocked)
- **Storage Buckets** — list all storage buckets
- **Sign Up / Login** — create account or login to get an authenticated JWT
  - JWT automatically set as the active token for subsequent requests

## How It Works

The app uses the Supabase REST API (PostgREST) directly from the browser:

```
GET  /rest/v1/              → OpenAPI schema (table names, columns, types)
GET  /rest/v1/{table}       → Read table data (with ?limit, ?offset, ?order, ?filter)
POST /rest/v1/rpc/{name}    → Call RPC function
GET  /storage/v1/bucket     → List storage buckets
GET  /auth/v1/settings      → Auth configuration
POST /auth/v1/signup        → Create account
POST /auth/v1/token         → Login
```

Key headers:
- `apikey` — API authentication
- `Authorization: Bearer {token}` — JWT for authenticated access
- `Accept-Profile: {schema}` — switch PostgreSQL schema
- `Prefer: count=exact` — get true row count in `content-range` header

## Security Notes

- All data stays in your browser — nothing is sent to any third party
- Connection credentials are saved in localStorage (clear browser data to remove)
- The app makes requests directly to the Supabase instance — CORS must allow browser access
- For security testing: use this to verify RLS policies, check exposed data, test write access

## Requirements

- Any modern browser (Chrome, Firefox, Edge, Safari)
- No server, no npm, no build step — just open the HTML file
- Target Supabase instance must allow CORS from the browser origin

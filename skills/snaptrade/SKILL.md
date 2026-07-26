---
name: snaptrade
description: Use when working with SnapTrade to fetch brokerage account data, positions, balances, orders, or transactions via the SnapTrade API. All accounts are at Fidelity.
---

# SnapTrade

Personal brokerage data via SnapTrade API. Auth uses OAuth bearer token stored in `~/.config/snaptrade/settings.json`.

> **Note:** Settings must have `"authMode": "oauth"` (not `"apiKey"`). With `apiKey` mode, the CLI tries to register a new user and fails (personal key only allows one user). Use `curl` directly for API calls — the CLI is only needed for token refresh.

## Auth

```bash
# Extract bearer token (auto-refreshes via snaptrade status)
snaptrade status > /dev/null 2>&1  # triggers token refresh if needed
TOKEN=$(python3 -c "import json; d=json.load(open('$HOME/.config/snaptrade/settings.json')); print(d['profiles']['default']['oauthAccessToken'])")

# Base URL
BASE="https://api.snaptrade.com/api/v1"
```

Helper function for a session:

```bash
snap() { curl -sf -H "Authorization: Bearer $TOKEN" "$BASE/$1" "${@:2}"; }
```

## Known Accounts

Account IDs and personal identifiers live in `context.json` (gitignored, machine-local).

```bash
CONTEXT="$(dirname "$0")/context.json"  # or full path to SKILL.md sibling
# Load all account IDs
jq -r '.accounts[] | "\(.name)\t\(.id)"' "$CONTEXT"
# Get a specific account ID
ACCT=$(jq -r '.accounts[] | select(.type=="ROTH") | .id' "$CONTEXT")
```

## Endpoints

### Accounts
```bash
snap accounts | jq '[.[] | {name, balance: .balance.total.amount, type: .raw_type}]'
```

### Positions
```bash
ACCT="67ea89da-a892-4d3a-8547-2c109b4c6066"  # Individual
snap "accounts/$ACCT/positions" | jq '.[]'
```

### Balances
```bash
snap "accounts/$ACCT/balances" | jq '.'
```

### Orders / Recent Activity
```bash
snap "accounts/$ACCT/orders" | jq '.'
```

### Transactions (with date range)
```bash
snap "accounts/$ACCT/activities?startDate=2026-01-01&endDate=2026-05-18" | jq '.'
```

## Quick Patterns

```bash
CONTEXT="$HOME/.claude/plugins/claude-code-plugins/skills/snaptrade/context.json"

# Total portfolio value across all accounts
snap accounts | jq '[.[] | .balance.total.amount] | add'

# All account balances summary
snap accounts | jq -r '.[] | "\(.name)\t\(.balance.total.amount)"' | column -t

# Positions for one account as table (load ID from context)
ACCT=$(jq -r '.accounts[] | select(.type=="I") | .id' "$CONTEXT")
snap "accounts/$ACCT/positions" \
  | jq -r '.[] | [.symbol.symbol, .units, .price, (.units * .price)] | @tsv' \
  | column -t

# All positions across all accounts
jq -r '.accounts[].id' "$CONTEXT" | while read ACCT; do
  snap "accounts/$ACCT/positions"
done | jq -s 'add'
```

## API Reference

- Docs: https://docs.snaptrade.com/reference
- Base: `https://api.snaptrade.com/api/v1`
- Auth: `Authorization: Bearer <token>`
- Token location: `~/.config/snaptrade/settings.json` → `.profiles.default.oauthAccessToken`
- Token refresh: run `snaptrade status` to trigger refresh if expired

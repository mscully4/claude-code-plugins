---
name: plaid
description: Use when working with the Plaid CLI to fetch financial data, link bank accounts, manage items, retrieve transactions/balances/investments/liabilities, or test with sandbox items.
---

# Plaid CLI

CLI for Plaid financial API. Binary at `/home/linuxbrew/.linuxbrew/bin/plaid` (via brew). Experimental — interface may change without notice.

## Setup

```bash
plaid register       # new account → opens Dashboard signup in browser
plaid login          # authenticate → stores tokens + fetches API keys automatically
plaid trial          # apply for Trial plan (required for Production/real institutions)
plaid keys fetch     # refresh API keys (run after Trial plan approval)
plaid logout         # remove stored credentials
plaid config         # check readiness status (secrets masked)
```

### Manual / Headless Credentials

Credential precedence: **flags > env vars > config file**

Each environment (sandbox, production) stores credentials separately — switching preserves all stored data.

```bash
plaid config set --client-id <id> --env sandbox   # secret prompted interactively
plaid config set --env production                  # switch environment
```

Environment variables:

| Variable | Purpose |
|----------|---------|
| `PLAID_CLIENT_ID` | Client ID |
| `PLAID_SECRET` | Secret key |
| `PLAID_ENV` | Environment (`sandbox` or `production`) |
| `PLAID_ACCESS_TOKEN` | Access token (overrides stored) |

### Teams

```bash
plaid teams list     # list available Dashboard teams
plaid teams choose   # interactively choose active team
plaid teams use 2    # set active team by index
```

## Linking Accounts

```bash
plaid link                                          # default: transactions product, opens browser
plaid link --products transactions,liabilities       # specify products
plaid link --products transactions \
  --required-if-supported-products liabilities      # conditional products
```

Products: `balance`, `transactions`, `investments`, `liabilities`

## Sandbox

```bash
plaid config set --env sandbox
plaid sandbox link                                  # no browser, picks institution automatically
plaid sandbox link --products balance
plaid sandbox link --products transactions,liabilities
plaid sandbox link --products investments
plaid sandbox link --products balance --institution-id ins_56
```

## Managing Items

Items = logins at financial institutions.

```bash
plaid item list                        # list all linked items
plaid item get --item my-bank          # show item + accounts
plaid item rename item-123 my-bank     # set alias for easier reference
plaid item remove --item my-bank       # destructive
```

## Fetching Data

When one Item linked → used by default. Multiple Items → use `--item` or `--all`.

### Balances
```bash
plaid balance
plaid balance --item my-bank
plaid balance --all --json
plaid balance --min-last-updated-datetime 2026-05-01T00:00:00Z
```

### Transactions
```bash
plaid transactions list                                              # last 30 days, default item
plaid transactions list --item my-bank --start-date 2026-04-01 --end-date 2026-04-30
plaid transactions list --all --count 500 --json
plaid transactions sync                # incremental cursor-based sync
plaid transactions sync --all --limit 2000 --json
```

### Investments
```bash
plaid investments holdings
plaid investments holdings --item my-brokerage --json
plaid investments holdings --all
plaid investments transactions --start-date 2026-01-01 --end-date 2026-05-17
plaid investments transactions --all --json
```

### Liabilities
```bash
plaid liabilities
plaid liabilities --item my-bank --json
plaid liabilities --all
```

## JSON Output (for agents/scripts)

`--json` / `-j` on any command:
- **stdout** → primary JSON result
- **stderr** → diagnostics only
- Browser commands still open browser when needed

```bash
plaid balance --json
plaid balance --json | jq '.accounts[0].name'
plaid transactions sync --all --json
```

## Common Flags

| Flag | Purpose |
|------|---------|
| `--item <id-or-alias>` | Target specific item (required when multiple items linked) |
| `--all` | Query all linked items in current environment |
| `--json` / `-j` | Structured JSON on stdout |
| `--access-token` | Override stored token |

## Quick Patterns

```bash
# Full financial snapshot
plaid balance --all --json
plaid transactions sync --all --json
plaid investments holdings --all --json
plaid liabilities --all --json

# Last 90 days transactions for one account
plaid transactions list --item my-bank \
  --start-date 2026-02-15 --end-date 2026-05-17 \
  --count 500 --json

# Pipe to jq
plaid balance --json | jq '.accounts[] | {name, available: .balances.available}'
```

## Agent State File

Persist discovered items, environment, and sync cursors across sessions to avoid redundant CLI calls.

State file: `~/.claude/skills/plaid/state.json` (gitignored — see `state.example.json` for schema).

**Bootstrap state (run once per session):**

```bash
STATE="${HOME}/.claude/skills/plaid/state.json"
SKILL_DIR="${HOME}/.claude/skills/plaid"

# Create from example if missing
[ ! -f "$STATE" ] && cp "$SKILL_DIR/state.example.json" "$STATE"

# Populate config + items
CONFIG=$(plaid config --json)
ITEMS=$(plaid item list --json)
NOW=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

jq -n \
  --argjson cfg "$CONFIG" \
  --argjson items "$ITEMS" \
  --arg ts "$NOW" \
  '{
    last_updated: $ts,
    config: {
      env: $cfg.env,
      available_environments: $cfg.available_environments,
      client_id: $cfg.client_id,
      team_id: $cfg.team_id,
      dashboard_auth: $cfg.dashboard_auth,
      linked_items: $cfg.linked_items,
      trial_plan_item_adds: $cfg.trial_plan_item_adds
    },
    items: ($items.items | map({
      item_id,
      institution_id,
      alias: null,
      has_cursor,
      accounts: []
    })),
    sync_cursors: {}
  }' > "$STATE"
```

**Enrich with account details per item:**

```bash
# Run for each item_id from state
plaid item get --item <item_id> --json | jq '.accounts | map({account_id, name, official_name, type, subtype, mask})'
# Merge result into state.json items[].accounts
```

**Read state to skip re-discovery:**

```bash
jq '.items[] | {item_id, alias, institution_id, has_cursor}' "$STATE"
jq '.sync_cursors' "$STATE"
jq '.config.env' "$STATE"
```

**Update cursor after sync:**

```bash
# transactions sync emits two JSON lines on stdout; second has cursor
CURSOR=$(plaid transactions sync --item <id> --json 2>/dev/null | tail -1 | jq -r '.cursor')
jq --arg id "<item_id>" --arg cur "$CURSOR" \
  '.sync_cursors[$id] = $cur | .last_updated = now | todate' "$STATE" > /tmp/s.json && mv /tmp/s.json "$STATE"
```

**Schema fields:**
- `config` — from `plaid config --json` (env, team, trial limits)
- `items[].item_id` / `institution_id` / `has_cursor` — from `plaid item list --json`
- `items[].alias` — set manually after `plaid item rename`; not in CLI JSON output
- `items[].accounts[]` — from `plaid item get --json` (account_id, type, subtype, mask)
- `sync_cursors` — `{ item_id: cursor }` from `plaid transactions sync --json` (second stdout line)
- `last_updated` — ISO8601 timestamp, update on every write

## Feedback

```bash
plaid feedback "message"   # with message
plaid feedback             # interactive
```

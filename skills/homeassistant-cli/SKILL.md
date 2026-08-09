---
name: homeassistant-cli
description: Query and control Home Assistant smart home devices via hass-cli. Use when the user asks about the state of their house or wants to control it — lights, switches, thermostats, garage doors, locks, sensors, cameras, or presence. Triggers include "turn on/off the lights", "what's the temperature", "is the garage door open", "lock the door", "set the thermostat", "close the garage", "check the house", "who's home", "smart home", "home assistant", "hassio".
allowed-tools: Bash(hass-cli:*)
---

# hass-cli — Home Assistant CLI

Query and control smart home devices with `hass-cli` ([home-assistant-cli](https://github.com/home-assistant-ecosystem/home-assistant-cli), PyPI package `homeassistant-cli`).

Verified against **hass-cli 1.0.0** / **Home Assistant 2026.5.4**.

## Authentication

`hass-cli` reads `HASS_SERVER` and `HASS_TOKEN` from the environment. Verify connectivity before anything else:

```bash
hass-cli config release   # prints the HA version
```

If that fails with a connection error, the server is unreachable. If it fails with 401, the token is bad or expired — a new one comes from HA → profile → Security → Long-Lived Access Tokens.

## Option Placement — read this first

**Global options must come BEFORE the subcommand.** This is the single most common mistake, and the error message (`No such option '-o'` or a bare usage dump) doesn't explain why.

```bash
hass-cli -o yaml state get light.dining_room_dimmer     # correct
hass-cli state get -o yaml light.dining_room_dimmer     # FAILS

hass-cli --no-headers state list                        # correct
hass-cli state list --no-headers                        # FAILS
```

Global options include `-o/--output`, `--no-headers`, `--columns`, `--sort-by`, `--server`, `--token`, `--timeout`, `--insecure`.

## Filtering — there is no `--filter`

`state list`, `device list`, `area list`, and `service list` take a **positional regex**, not a `--filter` expression. There is no SQL-like filter syntax anywhere in this CLI.

```bash
hass-cli state list '^light\.'        # correct — anchored regex
hass-cli state list --filter '...'    # FAILS — no such option
```

The regex matches the **friendly name as well as the entity_id**, so unanchored patterns over-match:

```bash
hass-cli state list light      # also returns binary_sensor.front_lights_switch_*, switch.front_lights_switch, ...
hass-cli state list '^light\.' # only the light domain
```

**To filter on state (not id), use JSON + jq** — this is what `--filter 'state == "on"'` was reaching for:

```bash
# Lights that are on
hass-cli -o json state list | jq -r '.[] | select(.entity_id|startswith("light.")) | select(.state=="on") | .entity_id'

# Anything currently on
hass-cli -o json state list | jq -r '.[] | select(.state=="on") | "\(.entity_id) \(.state)"'

# Unavailable entities (useful health check)
hass-cli -o json state list | jq -r '.[] | select(.state=="unavailable") | .entity_id'
```

`device list` / `area list` filters are regexes on **name only** — there is no way to filter devices by area from the CLI. They are also **case-sensitive**: device names are Title Case, so `device list 'Front'` returns rows while `device list 'front'` returns none.

## Discovery Workflow

Don't guess entity IDs.

```bash
# What domains exist, and how many of each
hass-cli --no-headers state list | awk '{print $1}' | cut -d. -f1 | sort | uniq -c | sort -rn

# Entities in a domain
hass-cli state list '^switch\.'

# Full detail on one entity (attributes reveal what a service call accepts)
hass-cli -o yaml state get climate.upstairs_thermostat

# Areas and devices
hass-cli area list
hass-cli device list
```

Inspecting attributes before a service call matters: a climate entity's `hvac_modes`, `min_temp`/`max_temp`, and `fan_modes` tell you exactly what values it will accept.

## Reading State

```bash
hass-cli state list                                  # everything
hass-cli state list '^light\.'                       # one domain
hass-cli -o yaml state get light.dining_room_dimmer  # full detail
hass-cli -o json state list                          # for jq

# Custom columns and sorting (both global options)
hass-cli --no-headers --columns ENTITY=entity_id,STATE=state --sort-by state state list '^switch\.'
```

## Control

Two ways. Prefer the shortcuts for plain on/off/toggle:

```bash
# Shortcuts — accept multiple entities
hass-cli state turn_on light.dining_room_dimmer
hass-cli state turn_off switch.front_lights_switch
hass-cli state toggle light.backyard_light_3
hass-cli state turn_off light.backyard_light_3 light.backyard_light_4   # several at once
```

Use `service call` when you need parameters:

```bash
hass-cli service call light.turn_on --arguments 'entity_id=light.dining_room_dimmer,brightness=128'
hass-cli service call climate.set_temperature --arguments 'entity_id=climate.upstairs_thermostat,temperature=72'
hass-cli service call cover.open_cover --arguments entity_id=cover.garage_door
hass-cli service call cover.close_cover --arguments entity_id=cover.garage_door

# What services exist?
hass-cli service list              # all
hass-cli service list switch       # positional regex, not --filter
```

`--arguments` is a comma-separated `key=value` list — quote the whole thing when passing more than one pair.

**Writes produce no useful output.** `state turn_on`/`turn_off` print an empty table (headers only) and `service call` prints `[]`, on success *and* on a no-op. Exit codes don't distinguish either. To confirm a write landed, read the state back — allow a couple of seconds for the device to report:

```bash
hass-cli state turn_on light.dining_room_dimmer
sleep 3
hass-cli --no-headers --columns E=entity_id,S=state state get light.dining_room_dimmer
```

## Templates — server-side queries

`template` renders Jinja2 against live HA state. It takes a **file path**, not an inline string, so write to a temp file first. This is the most powerful query mechanism available:

```bash
cat > /tmp/q.jinja2 <<'EOF'
{{ states.light | selectattr("state","eq","on") | map(attribute="entity_id") | join("\n") }}
EOF
hass-cli template /tmp/q.jinja2

# Count everything that's on
echo '{{ states | selectattr("state","eq","on") | list | count }}' > /tmp/c.jinja2
hass-cli template /tmp/c.jinja2
```

## Devices, Areas, Entities

```bash
hass-cli device list                 # all devices (ID, name, model, manufacturer, area)
hass-cli device list 'Front'         # regex on device name — CASE-SENSITIVE, 'front' matches nothing
hass-cli area list
hass-cli entity list
hass-cli entity assign <entity_id> <area>   # WRITE — changes HA config
```

`entity` also has `rename`, `enable`, `disable`, `delete`. All of these mutate configuration, not just device state — treat them as more dangerous than a service call.

## History

```bash
hass-cli state history --since 50m light.dining_room_dimmer
hass-cli state history --since 2026-06-27 climate.upstairs_thermostat
```

## System & Raw API

```bash
hass-cli config release        # HA version
hass-cli system log            # recent warnings/errors — genuinely useful for debugging
hass-cli raw get /api/states
hass-cli raw get /api/config
```

Note: `hass-cli system health` returns an empty row on this instance — it is not a useful health check. Use `config release` for a liveness check and `system log` for problems.

## Safety Rules

- **Read-only by default** — `state list`, `state get`, `device list`, `area list`, `entity list`, `config`, `system log`, `template`, `raw get` never change anything.
- **Write when instructed** — `service call` and `state turn_on/turn_off/toggle` are fine when the task asks for them. No need to confirm.
- **Confirm before physical-security actions** — locks, garage doors, and sirens have real-world consequences. Ask first unless the user explicitly named the action.
- **Confirm before `entity` mutations** — `assign`, `rename`, `disable`, `delete` change HA configuration, not device state, and are not trivially reversible.
- **Check state before acting** — reading an entity first avoids "turn off" calls on something already off, and avoids acting on an `unavailable` device.

## Common Patterns

```bash
# What's on right now?
hass-cli -o json state list | jq -r '.[] | select(.state=="on") | .entity_id'

# Any open doors/windows?
hass-cli state list '^binary_sensor\.' | grep -i ' on '

# Who's home?
hass-cli state list '^person\.'

# Devices that dropped off the network
hass-cli -o json state list | jq -r '.[] | select(.state=="unavailable") | .entity_id'

# Thermostat overview
hass-cli state list '^climate\.'
```

## Quick Reference

| Task | Command |
|---|---|
| HA version / liveness | `hass-cli config release` |
| Recent errors | `hass-cli system log` |
| All entities | `hass-cli state list` |
| One domain | `hass-cli state list '^light\.'` |
| Inspect entity | `hass-cli -o yaml state get <id>` |
| Filter by state | `hass-cli -o json state list \| jq -r '.[] \| select(.state=="on") \| .entity_id'` |
| Turn on / off | `hass-cli state turn_on <id>` / `hass-cli state turn_off <id>` |
| Toggle | `hass-cli state toggle <id>` |
| Set brightness | `hass-cli service call light.turn_on --arguments 'entity_id=<id>,brightness=128'` |
| Set thermostat | `hass-cli service call climate.set_temperature --arguments 'entity_id=<id>,temperature=72'` |
| Open / close garage | `hass-cli service call cover.open_cover --arguments entity_id=<id>` |
| List services | `hass-cli service list <regex>` |
| Devices / areas | `hass-cli device list` / `hass-cli area list` |
| History | `hass-cli state history --since 50m <id>` |

**Two things to remember:** global options go before the subcommand, and filters are positional regexes — never `--filter`.

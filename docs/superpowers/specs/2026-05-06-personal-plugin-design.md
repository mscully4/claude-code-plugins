# Personal Claude Code Plugin — Design Spec

**Date:** 2026-05-06
**Repo:** git@github.com:mscully4/claude-code-plugins.git

## Purpose

Single Claude Code plugin housing personal skills, commands, and hooks shared across all of Mike's projects and machines. Replaces copy-pasting project-specific Claude config by centralizing reusable automation in one installable plugin.

## Scope

- One plugin (not a mono-repo) — skills are opt-in by design so a Python skill won't fire in a React project
- Scaffold only — content (skills, hooks, commands) added incrementally as needed
- Cross-machine — designed to install cleanly on any machine via git clone

## Plugin Identity

- **Name:** `mscully`
- **Format:** Modern `.claude-plugin/plugin.json` (not legacy `package.json` format)
- **Version:** `0.1.0` (semantic versioning)

## Directory Structure

```
claude-code-plugins/
├── .claude-plugin/
│   └── plugin.json       # Plugin manifest (name, version, author, repo)
├── skills/               # One subdirectory per skill
│   └── <skill-name>/
│       └── SKILL.md      # Frontmatter: name, description, version
├── commands/             # One .md file per slash command
│   └── <command-name>.md
├── hooks/
│   └── hooks.json        # Hook event registry; scripts use ${CLAUDE_PLUGIN_ROOT}/scripts/
├── scripts/              # Bash helper scripts invoked by hooks
├── .gitignore
└── README.md             # Install instructions + usage
```

## Component Conventions

### Skills

Each skill lives at `skills/<name>/SKILL.md` with required frontmatter:

```yaml
---
name: skill-name
description: Use when user asks to "...", mentions "...", or discusses <topic>. <trigger conditions>
version: 0.1.0
---
```

Description field drives auto-triggering — write specific trigger phrases, not vague summaries.

### Commands (Slash Commands)

Each command is `commands/<name>.md` with frontmatter:

```yaml
---
description: Short description shown in /help
argument-hint: <required-arg> [optional-arg]
allowed-tools: [Read, Bash, Write]
---
```

Invoked by user as `/<name>` or `/mscully:<name>`.

### Hooks

`hooks/hooks.json` format:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "...",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/my-hook.sh"
          }
        ]
      }
    ]
  }
}
```

Scripts reference `${CLAUDE_PLUGIN_ROOT}` — never hardcode absolute paths.

## Installation

### Per Machine (initial setup)

```bash
git clone git@github.com:mscully4/claude-code-plugins.git ~/Workplace/claude-code-plugins
```

Then install locally by pointing Claude Code at the directory. Two options:

**Option A — Local path (TBD exact mechanism):**
The exact local install command for Claude Code plugins from a local directory path needs verification. Likely either:
- A `/plugin install path:~/Workplace/claude-code-plugins` CLI command, or
- Manually copying/symlinking into `~/.claude/plugins/cache/`

The README will document the verified steps once confirmed on first machine setup.

**Option B — Custom marketplace (future):**
Add `mscully4/claude-code-plugins` as a custom marketplace entry in `~/.claude/plugins/known_marketplaces.json` once the plugin has stable content and a marketplace index.

### Enabling Per Project

In each project's `.claude/settings.json` or `.claude/settings.local.json`:
```json
{
  "enabledPlugins": {
    "mscully@personal": true
  }
}
```

Or disable selectively by setting to `false`.

## Out of Scope

- MCP server integration (add later if needed)
- Marketplace publishing (personal use only)
- Multi-plugin mono-repo structure

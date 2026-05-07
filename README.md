# mscully — Personal Claude Code Plugin

Personal skills, commands, and hooks shared across projects and machines.

## Structure

```
claude-code-plugins/
├── .claude-plugin/plugin.json   # Plugin manifest
├── skills/                      # Auto-discovered skills (skills/<name>/SKILL.md)
├── commands/                    # Auto-discovered slash commands (commands/<name>.md)
├── hooks/hooks.json             # Hook event registry
├── scripts/                      # Helper scripts for hooks
└── README.md
```

## Adding a Skill

1. Create `skills/<skill-name>/SKILL.md` with frontmatter:

```yaml
---
name: skill-name
description: Use when user asks to "...", mentions "...", or discusses <topic>.
version: 0.1.0
---
```

2. Write the skill content below the frontmatter.
3. Commit and push. Pull on other machines.

## Adding a Command

1. Create `commands/<command-name>.md` with frontmatter:

```yaml
---
description: Short description shown in /help
argument-hint: <arg>
allowed-tools: [Read, Bash]
---
```

2. Write the command instructions below the frontmatter.
3. Invoked as `/<command-name>` or `/mscully:<command-name>`.

## Adding a Hook

1. Add a script to `scripts/` (bash, no extension):

```bash
#!/usr/bin/env bash
# Use ${CLAUDE_PLUGIN_ROOT} for portability — never hardcode absolute paths
echo "hook ran"
```

2. Register it in `hooks/hooks.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/my-hook"
          }
        ]
      }
    ]
  }
}
```

## Installation

### New Machine Setup

```bash
git clone git@github.com:mscully4/claude-code-plugins.git ~/Workplace/claude-code-plugins
```

**Note:** The exact local install command for Claude Code plugins from a local path is TBD pending verification. Options being investigated:
- `/plugin install path:~/Workplace/claude-code-plugins`
- Manual symlink into `~/.claude/plugins/cache/`

Once verified, update this section with exact steps.

### Enable Per Project

In `.claude/settings.json` or `.claude/settings.local.json`:

```json
{
  "enabledPlugins": {
    "mscully@personal": true
  }
}
```

### Keeping Up to Date

```bash
cd ~/Workplace/claude-code-plugins && git pull
```

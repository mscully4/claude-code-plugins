# Personal Plugin Scaffold Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Scaffold a personal Claude Code plugin (`mscully`) with the standard directory structure, ready to receive skills, commands, and hooks.

**Architecture:** Single plugin using the modern `.claude-plugin/plugin.json` manifest format. Empty component directories created with `.gitkeep` placeholders. Hooks registry initialized empty. README documents install flow.

**Tech Stack:** Claude Code plugin system (`.claude-plugin/plugin.json`), bash scripts for future hooks, markdown for skills/commands.

---

### Task 1: Plugin Manifest

**Files:**
- Create: `.claude-plugin/plugin.json`

- [ ] **Step 1: Create manifest**

```bash
mkdir -p /home/mike/Workplace/claude-code-plugins/.claude-plugin
```

Create `.claude-plugin/plugin.json`:

```json
{
  "name": "mscully",
  "version": "0.1.0",
  "description": "Personal skills, commands, and hooks shared across projects and machines",
  "author": {
    "name": "Mike Scully",
    "email": "catplusmike@gmail.com",
    "url": "https://github.com/mscully4"
  },
  "homepage": "https://github.com/mscully4/claude-code-plugins",
  "repository": "https://github.com/mscully4/claude-code-plugins",
  "license": "MIT"
}
```

- [ ] **Step 2: Verify JSON is valid**

```bash
python3 -c "import json; json.load(open('.claude-plugin/plugin.json')); print('valid')"
```

Expected output: `valid`

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "feat: add plugin manifest"
```

---

### Task 2: Component Directories

**Files:**
- Create: `skills/.gitkeep`
- Create: `commands/.gitkeep`
- Create: `scripts/.gitkeep`

- [ ] **Step 1: Create directories with gitkeep placeholders**

```bash
mkdir -p /home/mike/Workplace/claude-code-plugins/skills
mkdir -p /home/mike/Workplace/claude-code-plugins/commands
mkdir -p /home/mike/Workplace/claude-code-plugins/scripts
touch /home/mike/Workplace/claude-code-plugins/skills/.gitkeep
touch /home/mike/Workplace/claude-code-plugins/commands/.gitkeep
touch /home/mike/Workplace/claude-code-plugins/scripts/.gitkeep
```

- [ ] **Step 2: Verify directories exist**

```bash
ls -la /home/mike/Workplace/claude-code-plugins/
```

Expected: `skills/`, `commands/`, `scripts/` all present.

- [ ] **Step 3: Commit**

```bash
git add skills/.gitkeep commands/.gitkeep scripts/.gitkeep
git commit -m "feat: add component directories"
```

---

### Task 3: Hooks Registry

**Files:**
- Create: `hooks/hooks.json`

- [ ] **Step 1: Create hooks directory and empty registry**

```bash
mkdir -p /home/mike/Workplace/claude-code-plugins/hooks
```

Create `hooks/hooks.json`:

```json
{
  "hooks": {}
}
```

- [ ] **Step 2: Verify JSON is valid**

```bash
python3 -c "import json; json.load(open('hooks/hooks.json')); print('valid')"
```

Expected output: `valid`

- [ ] **Step 3: Commit**

```bash
git add hooks/hooks.json
git commit -m "feat: add empty hooks registry"
```

---

### Task 4: Gitignore

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Create .gitignore**

Create `.gitignore`:

```
# macOS
.DS_Store

# Python
__pycache__/
*.pyc
*.pyo
.venv/

# Node
node_modules/

# Editor
.idea/
.vscode/
*.swp

# Plugin local settings (machine-specific, never commit)
.claude/mscully.local.md
```

- [ ] **Step 2: Commit**

```bash
git add .gitignore
git commit -m "chore: add gitignore"
```

---

### Task 5: README with Install Instructions

**Files:**
- Create: `README.md`

- [ ] **Step 1: Create README**

Create `README.md`:

```markdown
# mscully — Personal Claude Code Plugin

Personal skills, commands, and hooks shared across projects and machines.

## Structure

```
claude-code-plugins/
├── .claude-plugin/plugin.json   # Plugin manifest
├── skills/                      # Auto-discovered skills (skills/<name>/SKILL.md)
├── commands/                    # Auto-discovered slash commands (commands/<name>.md)
├── hooks/hooks.json             # Hook event registry
├── scripts/                     # Helper scripts for hooks
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
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README with install and usage instructions"
```

---

### Task 6: Push to GitHub

- [ ] **Step 1: Push main branch**

```bash
git push -u origin main
```

Expected: branch pushed, no errors.

- [ ] **Step 2: Verify on GitHub**

Open `https://github.com/mscully4/claude-code-plugins` and confirm all files are present:
- `.claude-plugin/plugin.json`
- `skills/`, `commands/`, `scripts/`, `hooks/` directories
- `.gitignore`
- `README.md`
- `docs/superpowers/` (design spec and this plan)

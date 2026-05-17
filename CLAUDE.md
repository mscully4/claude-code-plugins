# mscully — Personal Claude Code Plugin

Personal skills, commands, and hooks for Claude Code, shared across machines.

## Structure

- `skills/<name>/SKILL.md` — auto-discovered skills
- `commands/<name>.md` — slash commands (`/<name>` or `/mscully:<name>`)
- `hooks/hooks.json` — hook event registry
- `scripts/` — bash scripts invoked by hooks (use `${CLAUDE_PLUGIN_ROOT}`, never hardcode paths)

## Adding Content

Follow the conventions in each directory. Commit and push — other machines pull to update.

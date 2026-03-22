---
title: 'GitHub Copilot CLI Slash Commands'
description: 'A reference guide to GitHub Copilot CLI slash commands, including session management, file operations, and agent controls.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-03-22
estimatedReadingTime: '6 minutes'
tags:
  - copilot-cli
  - slash-commands
  - reference
relatedArticles:
  - ./installing-and-using-plugins.md
  - ./automating-with-hooks.md
  - ./understanding-mcp-servers.md
prerequisites:
  - GitHub Copilot CLI installed
---

GitHub Copilot CLI provides a rich set of slash commands that control sessions, manage files, interact with plugins, and more. This page is a quick reference for the most useful commands.

## Session Management

### `/undo`

*(Added in v1.0.10)*

Undoes the **last turn** and reverts any file changes the agent made in that turn. This is useful when a response went in the wrong direction and you want to start fresh from your previous prompt.

```
/undo
```

The command removes the last exchange from the conversation history and restores any files that were modified in that turn to their prior state. You can then rephrase your prompt and try again.

> **Tip**: `/undo` reverts file changes but does not undo terminal commands the agent ran. If the agent executed side-effecting shell commands, you may need to reverse those manually.

### `/quit`

Ends the current Copilot session and exits the CLI.

### `/login` and `/logout`

Authenticate with GitHub. The `/login` command uses the device flow and works correctly in Codespaces and remote terminal environments.

```
/login
/logout
```

## File Operations

### `/copy`

Copies the last assistant response to the clipboard. On Windows, `/copy` writes formatted HTML so you can paste rich text directly into Word, Outlook, or Teams.

```
/copy
```

## Agent and Model Controls

### `/agent`

Switch to a different agent profile for the current session.

```
/agent terraform-expert
```

### Model Selection

Use the **model picker** (accessible via `/model` or the interactive model menu) to change the AI model. The picker organizes models into tabs — **Available**, **Blocked/Disabled**, and **Upgrade** — based on your GitHub plan and organization policy, making it easy to see which models you can use.

Use `--effort` (or the full `--reasoning-effort`) to control how much reasoning the model applies:

```bash
copilot --effort high
```

## Plugin Management

### `/plugin list`

Lists all installed plugins. Plugins loaded via `--plugin-dir` appear under a separate **External Plugins** section.

```
/plugin list
/plugin install my-plugin@awesome-copilot
/plugin marketplace browse awesome-copilot
```

See [Installing and Using Plugins](../installing-and-using-plugins/) for full details.

## Task and Session Visibility

### `/tasks`

Shows active subagent tasks. Idle subagents are automatically hidden after 2 minutes of inactivity to keep the view clean.

### `--resume`

Resume a previous session or task. Accepts either a session ID or a task ID:

```bash
copilot --resume <session-id>
copilot --resume <task-id>
```

## Terminal Setup

### `/terminal-setup`

Sets up shell integration (tab-completions, prompt indicators) for your terminal. This command is safe to run on WSL.

## Further Reading

- [GitHub Copilot CLI Releases](https://github.com/github/copilot-cli/releases) — Full changelog for all versions
- [Installing and Using Plugins](../installing-and-using-plugins/) — Managing plugins from the CLI
- [Automating with Hooks](../automating-with-hooks/) — Running scripts at lifecycle events
- [Understanding MCP Servers](../understanding-mcp-servers/) — Extending Copilot with external tool integrations

---

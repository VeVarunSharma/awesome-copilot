---
title: 'Managing Copilot CLI Sessions'
description: 'Learn how to run multiple concurrent sessions, use worktrees for parallel development, undo Copilot changes with /rewind, and control approval modes.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-08
estimatedReadingTime: '8 minutes'
tags:
  - copilot-cli
  - sessions
  - worktrees
  - fundamentals
relatedArticles:
  - ./using-copilot-coding-agent.md
  - ./agents-and-subagents.md
  - ./copilot-configuration-basics.md
prerequisites:
  - GitHub Copilot CLI installed and authenticated
---

GitHub Copilot CLI lets you run multiple independent coding sessions at the same time, isolate work in separate git worktrees, undo changes without git history, and control how much autonomy you give the agent. This article walks through each of these session-management capabilities.

## Multiple Concurrent Sessions

Starting with v1.0.79, Copilot CLI supports managing multiple sessions side by side from a **Sessions tab** in the sidebar. Each session has its own conversation history, tool calls, and agent context — you can switch between them without losing progress.

### Opening and Switching Sessions

Use the Sessions tab (visible in the split-view sidebar) to:

- **Open a new session** without ending your current one
- **Switch between sessions** by selecting them in the tab list
- **Close a session** with `x` (a confirmation prompt prevents accidental closure)

This is particularly useful when you want to:

- Work on two features simultaneously without mixing their conversation contexts
- Keep a long-running research session open while starting a separate implementation session
- Compare two different approaches without sharing a context window

### Resuming Sessions

Previous sessions are preserved and can be resumed:

```bash
copilot --resume
```

The `--resume` picker lists all recoverable sessions — including remote sessions from the coding agent — so you can reconnect to any of them without needing to know the session ID.

> **Note**: Switching between sessions no longer restarts MCP servers or rebuilds hook state, so background work in one session is never interrupted when you switch to another.

## Worktrees: Parallel Development Branches

Worktrees let you start a Copilot session in a **separate git worktree** — an isolated checkout of your repository at a different branch — without disrupting your current working directory. This is the recommended pattern for parallel feature work.

### Starting a Session in a New Worktree

From within an existing Copilot session:

```
/worktree new
```

This creates a new git worktree and opens a new Copilot session inside it. Each worktree has its own branch, so changes in one session don't affect the others.

You can also start with a worktree from the command line using the `--worktree` flag:

```bash
copilot --worktree
```

### Controlling the Worktree Base Branch

By default, new worktrees start from `HEAD`. You can change this behavior with the `worktreeBaseRef` setting in your Copilot settings:

```json
{
  "worktreeBaseRef": "main"
}
```

With this set, `--worktree` and `/worktree new` will branch off from `main` (or whichever remote default branch you configure) rather than your current `HEAD`.

### When to Use Worktrees

| Scenario | Without Worktrees | With Worktrees |
|----------|------------------|----------------|
| Two features at once | Must stash and switch branches | Two independent sessions with no stashing |
| Experimenting with an approach | Risk polluting your current branch | Isolated branch, easy to abandon |
| Parallel agent tasks | One agent at a time | Multiple agents, each in its own branch |

Worktrees pair naturally with [multiple concurrent sessions](#multiple-concurrent-sessions) — you can have one session per worktree, all visible in the Sessions tab.

## Undoing Copilot Changes with `/rewind`

When a session goes in an unexpected direction, `/rewind` lets you restore files that Copilot modified — without requiring git history or a clean working tree.

### How `/rewind` Works

```
/rewind
```

You'll be prompted to choose between:

- **Conversation only** — Clear the conversation history and return to the start of the session, without touching files
- **Conversation + files** — Clear the conversation and restore every file that Copilot wrote during the session

`/rewind` tracks the files Copilot touched and restores them to their pre-session state. It skips any file whose contents have been further modified since Copilot last wrote them, so manual edits you made on top of Copilot's changes are preserved.

> **Tip**: If you want to undo only some changes, use your editor's diff view or standard git commands to selectively revert. `/rewind` is designed for a full rollback of everything Copilot did in the session.

### When to Use `/rewind`

- The agent went down a wrong path and you want to start fresh
- You were experimenting and didn't want changes to persist
- You want to replay the conversation with a different approach from the beginning

## Controlling Approval Modes with `/permissions`

By default, Copilot asks for your approval before executing tool calls that modify files or run commands. `/permissions` lets you switch between approval modes without restarting the session.

```
/permissions
```

This opens a mode picker with options like:

| Mode | Behavior |
|------|----------|
| **Default** | Copilot asks for approval on sensitive operations |
| **Allow-all** | Copilot acts autonomously without asking for approval |
| **Allow-all auto** | Copilot uses a safety-judge model to decide when to ask |

> **Note**: In `allow-all auto` mode, the safety-judge model is selected automatically by Copilot — you cannot configure which model it uses. This mode offers a balance between speed and safety for routine tasks.

Approval-mode changes apply only to the current session. New sessions always start in the default mode.

### Why Control Permissions?

- **During exploration**: Keep the default mode so you can review each step
- **During well-understood tasks**: Switch to `allow-all` to let the agent work without interruptions
- **For automated pipelines**: Use `allow-all auto` so the agent handles routine steps unattended but still pauses on risky operations

## Combining `--plan` with Autopilot Mode

You can combine planning and autonomous execution in a single invocation:

```bash
copilot --plan --mode autopilot
```

With this combination, Copilot:

1. First generates a plan for the task without executing any changes
2. Presents the plan for you to review
3. After approval, executes the plan autonomously in autopilot mode — without pausing to ask at each step

This gives you a review checkpoint before handing off execution, which is useful for larger or higher-stakes tasks where you want visibility into the approach before committing to it.

## Quick Reference

| Command / Flag | What It Does |
|----------------|-------------|
| Sessions tab | View and switch between concurrent sessions |
| `copilot --resume` | Resume a previous or remote session |
| `/worktree new` | Start a new session in a new git worktree |
| `copilot --worktree` | Launch with a new worktree from the command line |
| `worktreeBaseRef` setting | Control which branch worktrees start from |
| `/rewind` | Restore files Copilot changed (with optional conversation reset) |
| `/permissions` | Switch approval mode for the current session |
| `--plan --mode autopilot` | Plan first, then execute autonomously after review |

## Next Steps

- **Parallel agent tasks**: [Agents and Subagents](../agents-and-subagents/) — Learn how `/fleet` and subagents can parallelize larger workloads
- **Autonomous coding**: [Using the Copilot Coding Agent](../using-copilot-coding-agent/) — Run agents on issues with full autonomy
- **Session hooks**: [Automating with Hooks](../automating-with-hooks/) — Trigger scripts on session start and end events

---

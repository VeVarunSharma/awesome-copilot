---
title: 'Copilot Memory'
description: 'Learn how GitHub Copilot Memory persists facts across sessions, how to control it with /memory commands, and how user and repository scopes differ.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-05-20
estimatedReadingTime: '7 minutes'
tags:
  - memory
  - configuration
  - fundamentals
relatedArticles:
  - ./copilot-configuration-basics.md
  - ./defining-custom-instructions.md
  - ./understanding-copilot-context.md
prerequisites:
  - Basic familiarity with GitHub Copilot CLI
---

Copilot Memory lets GitHub Copilot remember facts, preferences, and context across sessions — so you don't have to repeat yourself every time you start a new conversation. When memory is enabled, Copilot stores snippets of information it learns during your sessions and automatically surfaces them in future conversations where they're relevant.

This article explains how memory works, how to manage it with `/memory` commands, and the difference between user-scoped and repository-scoped memories.

## How Memory Works

When Copilot Memory is active, Copilot can store information it discovers during your session — things like your preferred frameworks, team conventions, frequently used project paths, or personal workflow preferences. In a future session, Copilot retrieves memories relevant to the current context and includes them in its reasoning automatically.

Memory is **not** injected wholesale into every prompt. Copilot selects relevant memories based on the current conversation, keeping context concise and focused.

### Memory vs Instructions vs Skills

| | Memory | Instructions | Skills |
|---|---|---|---|
| **Who creates it** | Copilot (automatically) | You (manually authored files) | You (manually authored folders) |
| **How it's activated** | Automatic, based on relevance | Automatic, based on file patterns | Manual invocation or automatic matching |
| **Scope** | User-wide or per-repository | Repository or workspace | Repository or user |
| **Best for** | Personal preferences, ad hoc facts | Coding standards, architectural decisions | Specialized task guidance |

Use instructions and skills for team-wide standards that should be version-controlled and shared. Use memory for personal facts and preferences that are specific to you.

## Managing Memory with `/memory`

Copilot CLI provides a `/memory` slash command for controlling memory directly from a session.

### Check Memory Status

```
/memory show
```

Displays whether memory is currently enabled, lists stored memories, and shows the scope of each entry (user-wide or repository-specific). Also shows documentation links for learning more about memory management.

### Enable or Disable Memory

```
/memory on
/memory off
```

These toggles are **persistent** — the setting is remembered across sessions until you change it again. Disabling memory stops new memories from being stored but does not delete existing memories.

### Memory Scopes

When Copilot stores a memory, it assigns it a scope:

| Scope | Description |
|-------|-------------|
| **User** | Visible to you across all repositories and sessions. Shown as `(for user)` in timeline entries and permission prompts. |
| **Repository** | Shared with all collaborators of a specific repository (`owner/repo`). Shown as `(shared with repository collaborators)`. |

When Copilot asks for permission to store a memory, the prompt explicitly names who can see it — either you only, or all repository collaborators. Timeline entries also display the scope so you can audit what was stored.

> **Privacy note**: Repository-scoped memories are visible to collaborators with access to the repository. Avoid storing sensitive personal information as repository-scoped memories.

## Managing Stored Memories

You can view and delete stored memories through the GitHub Copilot Memory UI:

- **Repository memories**: Repository Settings → Copilot → Memory
- **User memories**: [GitHub personal Copilot settings](https://github.com/settings/copilot/memory)

Copilot cannot delete its own memories from within a session — deletion must be done through the GitHub UI.

## Memory and Repository Context

Memory storage behaves slightly differently depending on whether a repository context is present:

- **With a repository**: Both user-scoped and repository-scoped memories are available for storage.
- **Without a repository** (e.g., running the CLI outside any repo): Only user-scoped memories can be stored — repository scope is unavailable.

This means memories captured in a general terminal session will always be user-scoped, while memories captured while working inside a repository can be either user or repository scope, depending on what Copilot determines is appropriate.

## When to Use Memory

Memory works best for:

- **Personal workflow preferences** — editor settings, preferred command patterns, or output formats you always request
- **Project shortcuts** — frequently used file paths, scripts, or environment details for a specific repository
- **Team conventions** — quick facts the team has agreed on, captured organically during sessions

Memory is not a replacement for:

- **`.github/instructions/`** files — use these for team-wide standards that should be version-controlled and applied automatically
- **Skills** — use these for structured, multi-step task guidance with bundled scripts
- **`.mcp.json`** — use this for tool and server configurations

## Further Reading

- [Copilot Configuration Basics](../copilot-configuration-basics/) — overview of all configuration layers
- [Defining Custom Instructions](../defining-custom-instructions/) — author persistent, file-pattern-based instructions
- [Understanding Copilot Context](../understanding-copilot-context/) — how Copilot assembles context for each response
- [GitHub Copilot Memory documentation](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/copilot-memory) — official docs for managing memories

---

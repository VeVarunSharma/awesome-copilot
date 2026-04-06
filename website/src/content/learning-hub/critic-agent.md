---
title: 'Critic Agent'
description: 'Learn how the Critic agent automatically reviews plans and complex implementations using a complementary AI model to catch errors early.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-04-06
estimatedReadingTime: '5 minutes'
tags:
  - agents
  - quality
  - experimental
relatedArticles:
  - ./building-custom-agents.md
  - ./using-copilot-coding-agent.md
prerequisites:
  - Basic understanding of GitHub Copilot chat
  - Familiarity with Copilot agents
---

The Critic agent is a built-in feature introduced in GitHub Copilot CLI v1.0.18 that automatically reviews plans and complex implementations using a complementary AI model. It acts as an independent second opinion, catching errors in reasoning and logic before the main agent proceeds.

## What Is the Critic Agent?

When you ask Copilot to tackle a complex task — write a detailed implementation plan, architect a system, or produce a long sequence of code changes — the Critic agent kicks in **before execution begins**. It uses a different AI model to evaluate the main agent's plan, looking for:

- **Logical errors** — flawed assumptions, incorrect reasoning
- **Missing edge cases** — scenarios the original plan doesn't account for
- **Implementation risks** — steps that are likely to fail or produce incorrect results
- **Inconsistencies** — contradictions in the plan's own logic

If the Critic finds significant issues, it feeds this feedback to the main agent, giving it a chance to revise the plan before any code is written or commands are run.

## How It Works

The Critic uses a **complementary model** — deliberately different from the main agent's model — to provide independent evaluation. This diversity of models helps avoid the "blind spots" that come from a single model reviewing its own output.

```
User request
     │
     ▼
Main agent creates plan
     │
     ▼
Critic agent reviews plan  ← different AI model
     │
     ├── Issues found → Main agent revises plan → Critic re-reviews
     │
     └── Plan approved → Main agent executes
```

This is similar to the peer review process in software development: having a second set of eyes on a plan before committing to it.

## Availability

The Critic agent is currently available in **experimental mode** and supports **Claude models** (Anthropic). It activates automatically for tasks where the main agent produces a detailed plan — you don't need to invoke it manually.

> **Note**: The Critic is an opt-in experimental feature. Check the GitHub Copilot CLI documentation or run `gh copilot config` to verify it's enabled in your environment.

## When the Critic Helps Most

The Critic is most valuable for:

| Task Type | Why Critic Helps |
|-----------|-----------------|
| **Architectural decisions** | Catches structural flaws before building the wrong thing |
| **Multi-step refactors** | Identifies steps that would break dependencies or introduce regressions |
| **Database migrations** | Flags missing rollback steps or data integrity risks |
| **API design** | Spots inconsistencies in resource naming, versioning, or error handling |
| **Security-sensitive code** | Catches common vulnerability patterns in the plan |

For simple, one-step tasks (editing a single file, answering a question), the Critic typically doesn't activate — it's designed for complex multi-step operations.

## Relationship to Custom Agents

If you're building a custom agent, the Critic adds a layer of review on top of any agent — it works with the main Copilot agent regardless of which custom agent or skill set you're using. You don't need to modify your `.agent.md` files to benefit from it.

## Further Reading

- [GitHub Copilot CLI Release Notes v1.0.18](https://github.com/github/copilot-cli/releases/tag/v1.0.18) — Critic agent announcement
- [Building Custom Agents](../building-custom-agents/) — Create agents that work alongside the Critic
- [Using the Copilot Coding Agent](../using-copilot-coding-agent/) — Run autonomous agent sessions with the Critic as a safety layer

---

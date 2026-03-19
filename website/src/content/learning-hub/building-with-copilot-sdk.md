---
title: 'Building with the GitHub Copilot SDK'
description: 'Learn how to use the GitHub Copilot SDK to embed agentic AI capabilities into your own applications across Python, TypeScript, Go, and .NET.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-03-19
estimatedReadingTime: '10 minutes'
tags:
  - sdk
  - development
  - agentic
  - fundamentals
relatedArticles:
  - ./building-custom-agents.md
  - ./understanding-mcp-servers.md
  - ./using-copilot-coding-agent.md
prerequisites:
  - GitHub Copilot CLI installed and authenticated
  - Familiarity with at least one of: Node.js/TypeScript, Python, Go, or .NET
---

The **GitHub Copilot SDK** lets you embed the same agentic engine powering GitHub Copilot directly into your own applications. Instead of building your own AI orchestration layer — planning, tool invocation, multi-turn conversations, streaming — you define agent behavior and let the SDK handle the rest.

This article explains what the Copilot SDK is, how to get started in your language of choice, and key patterns for building production-quality agentic applications.

> **Status**: The Copilot SDK is in **Technical Preview**. APIs may change. Not recommended for production use yet.

## What Is the Copilot SDK?

The SDK exposes the Copilot CLI's agent runtime as a programmable interface. Your application creates a session, sends prompts, and receives streaming responses — the SDK manages all communication with GitHub's model infrastructure.

```
Your Application
       │
  SDK Client  ── JSON-RPC ──  Copilot CLI (server mode)
                                     │
                              GitHub (models, auth)
```

The SDK manages the CLI process lifecycle automatically. Your code never talks directly to model APIs — authentication, token management, and model routing all happen in the CLI layer.

**What you can do with the SDK**:
- Embed AI assistants in internal tools, CLIs, and web applications
- Create domain-specific agents with custom tools and instructions
- Build multi-turn conversational experiences with persistent sessions
- Attach files, code, and context programmatically
- Connect to MCP servers for external tool access

## Prerequisites

1. **GitHub Copilot CLI** installed and authenticated:
   ```bash
   copilot --version   # verify installation
   copilot auth login  # authenticate
   ```

2. **Language runtime**: Node.js 18+, Python 3.8+, Go 1.21+, or .NET 8.0+

## Installation

### Node.js / TypeScript

```bash
mkdir my-copilot-app && cd my-copilot-app
npm init -y
npm install @github/copilot-sdk tsx
```

### Python

```bash
pip install github-copilot-sdk
```

### Go

```bash
mkdir my-copilot-app && cd my-copilot-app
go mod init my-copilot-app
go get github.com/github/copilot-sdk/go
```

### .NET

```bash
dotnet new console -n MyCopilotApp && cd MyCopilotApp
dotnet add package GitHub.Copilot.SDK
```

## Quick Start

### TypeScript

```typescript
import { CopilotClient } from "@github/copilot-sdk";

const client = new CopilotClient();
const session = await client.createSession({ model: "gpt-4.1" });

const response = await session.sendAndWait({
  prompt: "Explain the difference between async/await and Promises in JavaScript"
});
console.log(response?.data.content);

await client.stop();
```

### Python

```python
from github_copilot_sdk import CopilotClient

client = CopilotClient()
session = client.create_session(model="gpt-4.1")

response = session.send_and_wait(
    prompt="Explain the difference between async/await and Promises in JavaScript"
)
print(response.content)

client.stop()
```

### Go

```go
package main

import (
    "fmt"
    copilot "github.com/github/copilot-sdk/go"
)

func main() {
    client := copilot.NewClient()
    defer client.Stop()

    session, _ := client.CreateSession(copilot.SessionOptions{Model: "gpt-4.1"})
    response, _ := session.SendAndWait("Explain async/await vs Promises in JavaScript")
    fmt.Println(response.Content)
}
```

### .NET

```csharp
using GitHub.Copilot.SDK;

var client = new CopilotClient();
var session = await client.CreateSessionAsync(new SessionOptions { Model = "gpt-4.1" });

var response = await session.SendAndWaitAsync(
    "Explain the difference between async/await and Promises in JavaScript"
);
Console.WriteLine(response.Content);

await client.StopAsync();
```

## Core Concepts

### Sessions

A **session** is a stateful conversation context. Sessions maintain message history, so you can have multi-turn conversations:

```typescript
const session = await client.createSession({ model: "gpt-4.1" });

// Turn 1
await session.sendAndWait({ prompt: "My name is Alice, I'm building a task manager" });

// Turn 2 — Copilot remembers context from turn 1
const response = await session.sendAndWait({
  prompt: "What data model would you suggest for my app?"
});
// Response: "For your task manager, Alice, I'd suggest..."
```

Sessions persist until you explicitly end them or the client stops. For web applications, use a custom session ID to correlate sessions with user sessions:

```typescript
const session = await client.createSession({
  model: "gpt-4.1",
  sessionId: `user-${userId}-${Date.now()}`
});
```

### Streaming Responses

For interactive applications, use streaming to show responses as they're generated:

```typescript
const session = await client.createSession({ model: "gpt-4.1" });

session.on((event) => {
  if (event.type === "content_block_delta") {
    process.stdout.write(event.data.delta);  // print each token as it arrives
  }
  if (event.type === "session.idle") {
    console.log("\n[Done]");
  }
});

await session.send({ prompt: "Write a short story about a robot learning to code" });
```

### File Attachments

Attach files to provide context for your prompt:

```typescript
const response = await session.sendAndWait({
  prompt: "Review this file for security issues",
  attachments: [{
    type: "file",
    path: "./src/auth.ts",
    displayName: "Authentication Module"
  }]
});
```

### Custom Tools

Define tools that Copilot can call during a conversation — similar to MCP server tools, but defined inline:

```typescript
const session = await client.createSession({
  model: "gpt-4.1",
  tools: [
    {
      name: "get_user",
      description: "Retrieve a user by ID from the database",
      parameters: {
        type: "object",
        properties: {
          userId: { type: "string", description: "The user's unique ID" }
        },
        required: ["userId"]
      },
      handler: async ({ userId }) => {
        const user = await db.users.findById(userId);
        return JSON.stringify(user);
      }
    }
  ]
});

const response = await session.sendAndWait({
  prompt: "What's the email address of user ID abc123?"
});
// Copilot calls get_user("abc123") and uses the result in its response
```

### Connecting to MCP Servers

Configure MCP servers to give the agent access to external systems:

```typescript
const session = await client.createSession({
  model: "gpt-4.1",
  mcpServers: [{
    name: "github",
    command: "npx",
    args: ["-y", "@github/mcp-server"],
    env: { GITHUB_TOKEN: process.env.GITHUB_TOKEN }
  }]
});

const response = await session.sendAndWait({
  prompt: "What are the open issues in the github/awesome-copilot repo?"
});
// Copilot uses the GitHub MCP server to fetch real issue data
```

## Common Patterns

### Graceful Shutdown

Always clean up the client when your application exits:

```typescript
// Handle Ctrl+C and other termination signals
process.on("SIGINT", async () => {
  console.log("Shutting down...");
  await client.stop();
  process.exit(0);
});

// Always use try-finally in request handlers
try {
  const session = await client.createSession({ model: "gpt-4.1" });
  const response = await session.sendAndWait({ prompt: userInput });
  return response?.data.content;
} finally {
  await client.stop();
}
```

### Timeout Handling

Set timeouts for long operations to avoid hanging indefinitely:

```typescript
const timeoutId = setTimeout(() => {
  session.abort();
  console.error("Request timed out");
}, 60_000); // 60-second timeout

session.on((event) => {
  if (event.type === "session.idle") {
    clearTimeout(timeoutId);
  }
});
```

### Error Handling

```typescript
try {
  const response = await session.sendAndWait({ prompt: userInput });
  return response?.data.content;
} catch (error) {
  if (error.code === "ECONNREFUSED") {
    throw new Error("Cannot connect to Copilot CLI. Is it installed and authenticated?");
  }
  throw error;
}
```

### Querying Available Models

List models at runtime to let users choose or to verify availability:

```typescript
const models = await client.getModels();
// Returns: ["gpt-4.1", "gpt-4o", "claude-sonnet-4.5", ...]
console.log("Available models:", models);
```

## Building a CLI Tool

Here's a complete example of a CLI tool that wraps Copilot for a specific domain (code review):

```typescript
import { CopilotClient } from "@github/copilot-sdk";
import { readFileSync } from "fs";

async function reviewCode(filePath: string): Promise<void> {
  const client = new CopilotClient();

  try {
    const session = await client.createSession({
      model: "gpt-4.1",
      systemPrompt: `You are a senior code reviewer. Focus on:
        - Security vulnerabilities
        - Performance issues  
        - Code clarity and maintainability
        Format your response with clear sections and actionable suggestions.`
    });

    // Stream the review as it's generated
    session.on((event) => {
      if (event.type === "content_block_delta") {
        process.stdout.write(event.data.delta);
      }
    });

    await session.send({
      prompt: `Please review this code for issues:`,
      attachments: [{ type: "file", path: filePath, displayName: filePath }]
    });

    // Wait for completion
    await new Promise<void>((resolve) => {
      session.on((event) => {
        if (event.type === "session.idle") resolve();
      });
    });

  } finally {
    await client.stop();
  }
}

// Usage: npx tsx review.ts src/auth.ts
reviewCode(process.argv[2]);
```

## Best Practices

- **Always clean up**: Call `client.stop()` in a `finally` block or signal handler. Leaked processes consume resources.
- **Set timeouts**: Use `sendAndWait` with a timeout for production applications. Long-running sessions can stall.
- **Handle events**: Subscribe to error events for robust error handling in streaming scenarios.
- **Use streaming for UX**: Streaming responses feel faster and more interactive than waiting for the full response.
- **Persist session IDs**: For multi-turn web applications, tie session IDs to user sessions so conversations survive page reloads.
- **Define clear tools**: Write descriptive tool names and detailed descriptions — Copilot uses these to decide when and how to call your tools.

## Further Reading

- **[GitHub Copilot SDK](https://github.com/github/copilot-sdk)** — Official SDK repository with samples and tutorials
- **[Getting Started Tutorial](https://github.com/github/copilot-sdk/blob/main/docs/tutorials/first-app.md)** — Step-by-step guide to building your first app
- **[SDK Cookbook](https://github.com/github/copilot-sdk/tree/main/cookbook)** — Recipes for common use cases
- **[copilot-sdk plugin](https://github.com/github/awesome-copilot/tree/main/plugins/copilot-sdk)** — Awesome Copilot plugin with language-specific SDK instructions
- **[Understanding MCP Servers](../understanding-mcp-servers/)** — Learn how to connect Copilot to external tools
- **[Building Custom Agents](../building-custom-agents/)** — Create agents that work alongside your SDK applications
- **[GitHub MCP Server](https://github.com/github/github-mcp-server)** — The MCP server for GitHub API access

---

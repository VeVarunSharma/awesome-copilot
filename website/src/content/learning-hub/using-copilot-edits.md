---
title: 'Using Copilot Edits for Multi-File Changes'
description: 'Learn how to use GitHub Copilot Edits to apply AI-assisted changes across multiple files in a single session.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-03-31
estimatedReadingTime: '9 minutes'
tags:
  - copilot-edits
  - multi-file
  - vs-code
  - fundamentals
relatedArticles:
  - ./using-copilot-coding-agent.md
  - ./understanding-copilot-context.md
  - ./building-custom-agents.md
prerequisites:
  - GitHub Copilot extension installed in VS Code
  - Active GitHub Copilot subscription
---

GitHub Copilot Edits is a multi-file editing mode in VS Code that lets you apply coordinated AI-suggested changes across your entire working set in a single session. Instead of asking Copilot to change one file at a time in Chat, Edits lets you describe a broader goal—like "add input validation to all API endpoints" or "migrate these components from class-based to functional style"—and Copilot proposes changes across every relevant file simultaneously.

This article explains how Copilot Edits works, how to use it effectively, and when to reach for it instead of Chat or inline completions.

## How Copilot Edits Differs from Chat

GitHub Copilot has three primary interaction modes in VS Code, each suited to a different type of work:

| Mode | Best For | Output |
|------|----------|--------|
| **Inline completions** | Writing new code as you type | Completions at cursor |
| **Copilot Chat** | Questions, explanations, planning | Conversation + code snippets |
| **Copilot Edits** | Applying coordinated changes across files | Diffs applied directly to your files |

The fundamental difference is that Edits is **edit-oriented**, not conversation-oriented. You describe what you want changed, and Copilot directly modifies files in your working set—showing the proposed diffs for you to review and accept or reject.

## Opening Copilot Edits

Open Copilot Edits in VS Code using any of these methods:

- **Keyboard shortcut**: `Ctrl+Shift+I` (Windows/Linux) or `Cmd+Shift+I` (macOS)
- **Command Palette**: Run `Copilot: Open Copilot Edits`
- **Chat icon**: Click the chat icon in the Activity Bar, then switch to the **Edits** tab

You'll see a panel with an input area at the bottom and a list of files in your working set above it.

## Building Your Working Set

The **working set** is the collection of files Copilot is allowed to view and edit during your session. Start with just the files relevant to your task:

### Adding Files

Click **Add Files** or drag files from the Explorer into the Edits panel. You can also type `#` in the Edits input to reference specific files or folders.

```
Add the following files:
- src/api/users.ts
- src/api/products.ts
- src/api/orders.ts
- src/middleware/validation.ts
```

> **Tip**: Start with a focused set of files. You can add more as you go. A smaller working set reduces ambiguity and generally produces better results.

### What Copilot Can See

Copilot can read any file in your working set and any file already open in your editor. It uses this context to understand dependencies, shared types, and existing patterns before making changes.

## Making Change Requests

Describe what you want to change in plain English. Be specific about the goal, the approach, and any constraints:

**Example: Adding validation across API handlers**

```
Add Zod input validation to the POST handlers in users.ts, products.ts, 
and orders.ts. Use the existing schema patterns from validation.ts.
Return HTTP 400 with a structured error body if validation fails.
Do not change any GET handlers.
```

**Example: Refactoring class components**

```
Convert the React class components in the components/ folder to 
functional components using hooks. Keep the same props interface
and maintain all existing behavior. Use useCallback for event handlers
and useMemo for expensive computations.
```

Copilot processes your request across all files in the working set and proposes diffs for each file it wants to modify.

## Reviewing and Accepting Changes

Copilot presents proposed changes as diffs—a side-by-side view showing the original and the new version. You have several options:

### Per-File Review

Each modified file appears in the working set panel with a change indicator. Click any file to open the diff view:

- **Accept**: Apply the proposed changes to this file
- **Discard**: Keep the original and remove the proposed changes
- **Open File**: View the full file with changes applied (for context)

### Bulk Actions

- **Accept All**: Apply all proposed changes across every file
- **Discard All**: Remove all proposed changes and start over

> **Best practice**: Review each file individually before accepting. Even when a change looks correct in isolation, check it against the other modified files to ensure consistency.

## Iterating on Changes

Copilot Edits supports natural iteration. After reviewing an initial proposal, you can:

**Refine the request**: Add a follow-up instruction without discarding the accepted changes:

```
Good. Now also add the validation to the PATCH handlers I missed earlier,
and add JSDoc comments to each validation schema.
```

**Fix a specific file**: If one file's changes aren't right, discard just that file's changes and provide more specific guidance:

```
The orders.ts change is wrong — the orderId field should remain required 
even when updating. Fix that validation schema only.
```

**Expand the working set**: Add files that turn out to be relevant:

```
Also update the OpenAPI spec in docs/api.yaml to reflect the new 
validation constraints you added.
```

## Practical Examples

### Example 1: Adding Error Handling

**Goal**: Add consistent error handling to API service methods

**Working set**: `src/services/user-service.ts`, `src/services/product-service.ts`, `src/services/order-service.ts`, `src/utils/error-handler.ts`

**Request**:
```
Wrap all public async methods in each service file with try/catch. 
On error, call logError from error-handler.ts and re-throw a 
ServiceError with the original message and a service-specific error code.
See the existing pattern in error-handler.ts for the error structure.
```

### Example 2: Migrating Configuration

**Goal**: Replace hardcoded config values with environment variables

**Working set**: `src/config/database.ts`, `src/config/auth.ts`, `src/config/storage.ts`, `.env.example`

**Request**:
```
Replace all hardcoded values in the config files with process.env lookups.
Add default values where appropriate. Also update .env.example to document
each new environment variable with a comment explaining its purpose.
```

### Example 3: Test Coverage

**Goal**: Add unit tests for new utility functions

**Working set**: `src/utils/date-formatter.ts`, `src/utils/currency-converter.ts`, `tests/utils/date-formatter.test.ts`, `tests/utils/currency-converter.test.ts`

**Request**:
```
Write comprehensive unit tests for the utility functions in date-formatter.ts
and currency-converter.ts. Follow the patterns in the existing test files.
Cover: happy path, edge cases (null, undefined, empty string), and boundary values.
Use the same test framework already present in the working set.
```

## When to Use Edits vs Other Modes

### Use Copilot Edits when:

- ✅ You want to apply the **same type of change** across multiple files (consistent patterns, uniform migrations)
- ✅ You have a **clear, bounded task** with a defined working set of files
- ✅ You want to see diffs **before committing** to any change
- ✅ You're doing a **refactoring** that spans multiple files
- ✅ You need changes to be **coordinated**—changes in file A must be consistent with changes in file B

### Use Copilot Chat when:

- 💬 You have **questions** or want explanations about existing code
- 💬 You're **exploring options** before committing to an approach
- 💬 You need **targeted changes** to a single file or a few lines
- 💬 You want to iterate conversationally with code snippets

### Use inline completions when:

- ⌨️ You're actively **writing new code** and want suggestions as you type
- ⌨️ You need **quick completions** within the current context
- ⌨️ The change is **local** to the file you're editing

### Edits vs the Coding Agent

Copilot Edits runs locally in your IDE—it's a **synchronous, interactive** session where you review and control each change. The Copilot coding agent, by contrast, runs autonomously in a cloud environment, works on GitHub issues, and opens PRs for review.

Use Edits when you're actively driving the session. Use the coding agent for hands-off autonomous work on well-defined tasks. See [Using the Copilot Coding Agent](../using-copilot-coding-agent/) for details.

## Best Practices

### Craft Specific Requests

Vague: `"Improve the code in these files"`

Specific: `"Add TypeScript strict null checks and replace all any types with proper interfaces in these service files. Do not change existing function signatures."`

Specific requests produce coherent diffs. Vague requests produce uncertain changes that are harder to review.

### Keep Working Sets Focused

Add only the files directly related to your task. A 3–5 file working set typically yields better results than a 20-file one. If your task is genuinely that broad, break it into multiple Edits sessions.

### Review Before Accepting

Always read the diff before clicking **Accept All**. Pay attention to:

- Files that changed more than expected
- Imports that were added or removed
- Logic that changed in subtle but significant ways

### Use Custom Instructions

If your repository has custom instructions (`.github/instructions/*.instructions.md`), Copilot Edits reads them and applies your coding standards automatically. This means you don't need to spell out style rules in every request.

See [Defining Custom Instructions](../defining-custom-instructions/) for how to set these up.

### Combine with Skills

If your team has skills for specific tasks (like `generate-tests` or `add-error-handling`), reference them in your Edits requests to invoke the skill's bundled guidance and templates. This keeps your Edits sessions consistent with your team's established workflows.

## Common Questions

**Q: Can Copilot Edits create new files?**

A: Yes. If your request requires a new file (for example, a test file that doesn't exist yet), Copilot will propose creating it. New files appear in the working set panel with a "new" indicator.

**Q: What if I accept changes I didn't want?**

A: Use Git to undo. Copilot Edits applies changes directly to your files, so `git checkout` or your IDE's source control view will restore the previous state of any file.

**Q: Does Copilot Edits work with non-TypeScript projects?**

A: Yes. Copilot Edits works with any language or file type that VS Code can edit—Python, Go, Java, Ruby, Terraform, YAML, and more.

**Q: How does Copilot Edits differ from editing via Chat?**

A: Chat can suggest code changes as part of a conversation, but you must manually apply them to files. Copilot Edits shows diffs directly in the file and applies changes with a single click, making large-scale changes much more efficient.

**Q: Can I use Copilot Edits with custom agents?**

A: Custom agents are available in Copilot Chat. Copilot Edits uses the same underlying model as your default or configured Copilot session but does not use the agent picker. For complex multi-file tasks requiring a specialized persona, you can prompt the agent in Chat to make specific file changes or combine Chat guidance with Edits execution.

## Next Steps

- **Understand Context**: [Understanding Copilot Context](../understanding-copilot-context/) — Learn what Copilot "sees" to improve your requests
- **Add Instructions**: [Defining Custom Instructions](../defining-custom-instructions/) — Make Edits automatically follow your coding standards
- **Automate More**: [Using the Copilot Coding Agent](../using-copilot-coding-agent/) — Go hands-off with autonomous issue resolution
- **Build Agents**: [Building Custom Agents](../building-custom-agents/) — Create specialized assistants for Chat-based workflows

---

---
title: 'Using Copilot Spaces'
description: 'Learn how to use GitHub Copilot Spaces to bring project-specific context, curated knowledge, and shared workflows into your Copilot conversations.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-03-19
estimatedReadingTime: '9 minutes'
tags:
  - spaces
  - context
  - collaboration
  - fundamentals
relatedArticles:
  - ./understanding-copilot-context.md
  - ./building-custom-agents.md
  - ./understanding-mcp-servers.md
prerequisites:
  - Basic understanding of GitHub Copilot chat
---

A **Copilot Space** is a shared, curated knowledge base that grounds Copilot responses in your team's actual code, documentation, and internal standards. Instead of pasting context into every conversation, you attach repositories, files, and instructions to a Space once — and Copilot draws on that context automatically whenever the Space is loaded.

This article explains what Spaces are, how to create and manage them, and how to use them to get more accurate, project-aware answers from Copilot.

## What Is a Copilot Space?

A Space is a named collection of resources and instructions that can be loaded into a Copilot conversation on demand. Think of it as a reusable context package:

```
Copilot Space: "API Platform"
├── Instructions: "Always follow our REST conventions in docs/api-standards.md"
├── Repository: github/api-platform (code search, file browsing)
├── File: docs/architecture.md
├── File: docs/authentication-guide.md
└── Issue: #1234 (current quarter initiatives)
```

When a developer opens this Space, Copilot has immediate access to the codebase, knows the architecture, understands the team's standards, and can answer questions grounded in actual project knowledge — not generic web information.

**Key characteristics**:
- Curated by a team member or organization admin
- Accessible to invited collaborators or public
- Resources auto-update as the underlying repos and files change
- Can include custom instructions that guide Copilot's behavior within the Space
- Accessible via MCP tools or the REST API

### When to Use Spaces

| Scenario | Why a Space Helps |
|----------|-------------------|
| Onboarding new developers | Load architecture, conventions, and setup guides in one command |
| Cross-team collaboration | Share a Space with project context instead of long briefs |
| Domain-specific Q&A | Answer security, compliance, or standards questions using internal docs |
| Structured workflows | Encode multi-step processes (PM templates, release checklists) in Space instructions |
| Context-heavy tasks | Load the right repos and files without repetitive copy-pasting |

## Loading a Space

Spaces can be loaded in any Copilot Chat conversation using the `copilot-spaces` skill or through the MCP GitHub server.

### With the copilot-spaces Skill

Install the skill from Awesome Copilot to get natural language access to Spaces:

```bash
copilot plugin install copilot-spaces@awesome-copilot
```

Once installed, you can ask Copilot to work with Spaces directly:

```
"Load the API Platform space and help me design a new authentication endpoint"
"What spaces are available for our team?"
"Using the security space, what's our policy on dependency scanning?"
```

### With MCP Tools (Programmatic)

The GitHub MCP server exposes read-only Space tools:

| Tool | Purpose |
|------|---------|
| `mcp__github__list_copilot_spaces` | List all Spaces accessible to the current user |
| `mcp__github__get_copilot_space` | Load a Space's full context by owner and name |

**List available spaces**:
```
Call mcp__github__list_copilot_spaces
→ Returns spaces with name, owner, and description
```

**Load a specific space**:
```
Call mcp__github__get_copilot_space with:
  owner: "myorg"
  name: "API Platform"
→ Returns full context: docs, code, instructions, resources
```

> **Note**: Space names are case-sensitive. Use the exact name returned by `list_copilot_spaces`.

## Creating and Managing Spaces

### Creating a Space

Use the `gh api` command to create a Space (full CRUD operations require the REST API):

```bash
# Create a user Space
gh api users/{username}/copilot-spaces \
  -X POST \
  -f name="API Platform" \
  -f description="Context for the API Platform team" \
  -f general_instructions="Always follow our REST conventions documented in docs/api-standards.md. Use camelCase for JSON fields." \
  -f visibility="private"
```

For organization Spaces, use `/orgs/{org}/copilot-spaces/` instead of `/users/{username}/copilot-spaces/`.

**Visibility options**:
- `"private"` — only invited collaborators can access
- `"public"` — anyone in the organization can discover and load

### Adding Resources

Attach repositories, files, issues, and free-text notes to a Space:

```json
{
  "resources_attributes": [
    {
      "resource_type": "github_file",
      "metadata": {
        "repository_id": 12345,
        "file_path": "docs/architecture.md"
      }
    },
    {
      "resource_type": "github_issue",
      "metadata": {
        "repository_id": 12345,
        "number": 1234
      }
    },
    {
      "resource_type": "free_text",
      "metadata": {
        "name": "Team Norms",
        "text": "We use trunk-based development. All PRs need two approvals."
      }
    }
  ]
}
```

```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  --input resources.json
```

> **Important**: Resource updates replace the entire resource list. To add a resource without removing existing ones, include all existing resources plus the new one in the request. To remove a specific resource, include it with `"_destroy": true`.

### Updating Space Instructions

The `general_instructions` field is where you define how Copilot should behave when working within this Space:

```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  -f general_instructions="You are an expert in our microservices architecture. \
When answering questions: \
1. Always check the attached architecture doc first \
2. Prefer patterns established in the existing codebase \
3. Flag any suggestions that require cross-team coordination"
```

### Managing Collaborators

Add team members to a private Space:

```bash
# Add a collaborator
gh api users/{username}/copilot-spaces/{number}/collaborators \
  -X POST \
  -f username="teammate" \
  -f role="reader"
```

Available roles: `"reader"` (can load and use), `"writer"` (can edit resources and instructions).

## Using Spaces Effectively

### Follow the Breadcrumbs

Space content often references external resources — issues, dashboards, repositories, or other tools. When Copilot loads a Space, it should proactively follow these references to gather complete context:

- A space mentions an initiative tracking issue → fetch the latest comments with `issue_read`
- A space links to a project board → check current status with project tools
- A space references a repository's masterplan → read it with `get_file_contents`

This "breadcrumb following" is what makes Spaces powerful: a well-designed Space acts as a map that guides Copilot to the right context automatically.

### Spaces as Workflow Engines

Spaces aren't just for Q&A — they can encode structured workflows. If a Space's instructions contain a step-by-step process, Copilot will follow it:

**Example: PM Weekly Update Space**

```
Instructions:
1. Pull the last 7 days of closed issues from the attached repo
2. Check the milestone progress in the project board
3. Draft a weekly update using this template:
   ## Progress
   [summary of completed work]
   ## Blockers
   [any blockers or risks]
   ## Next Week
   [planned work]
4. Show the draft for review before finalizing
```

A developer asks: "Write my weekly update using the PM Updates space."  
Copilot loads the Space, follows the workflow, fetches the relevant data, and produces a draft — all from a single prompt.

### Space Instructions as Directives

Treat Space instructions as authoritative guides, not suggestions. If a Space says "always check compliance before recommending a third-party dependency," Copilot will follow that rule in every response within that Space.

This makes Spaces ideal for encoding:
- Coding standards and architectural constraints
- Security and compliance requirements
- Team-specific patterns and preferences
- Workflow templates and checklists

## Scope Requirements

Creating and writing Spaces requires the `user` PAT scope. If you encounter 404 errors on write operations:

```bash
gh auth refresh -h github.com -s user
```

Read operations (listing and loading Spaces) only require `read:user`.

## Best Practices

- **Keep instructions focused**: Write instructions that are specific to the Space's purpose. Broad, generic instructions belong in `.github/copilot-instructions.md` instead.
- **Update resources regularly**: Attach living documents (issues, PRs) rather than static copies. Spaces auto-update as underlying content changes.
- **Name Spaces clearly**: Use descriptive names like "API Platform", "Security Standards", or "Frontend Conventions" — not abbreviations. Names are case-sensitive.
- **Start small**: Begin with a focused Space for one team or domain before creating organization-wide Spaces.
- **Encode workflows**: If your team follows a repeatable process, encode it in the Space's `general_instructions` so Copilot can run it end-to-end.

## Common Questions

**Q: How is a Space different from a `.github/copilot-instructions.md` file?**

A: Instructions files apply automatically to all Copilot conversations in a repository. Spaces are loaded explicitly and are shareable across repositories and teams. Use instruction files for repo-level conventions; use Spaces for cross-repo or team-level knowledge bases.

**Q: Can Spaces include code from private repositories?**

A: Yes. Collaborators with access to the Space can use the attached private repos as context. They only see data they already have permission to access.

**Q: How large can a Space be?**

A: Space content can be large (20KB+). When returned as a temp file, use `grep` or view specific sections rather than reading everything at once.

**Q: Is the Spaces API stable?**

A: Spaces are functional but the REST API is not yet in the public API docs and may require the `copilot_spaces_api` feature flag. Functionality is subject to change.

## Further Reading

- **[copilot-spaces skill](https://github.com/github/awesome-copilot/tree/main/skills/copilot-spaces)** — The Awesome Copilot skill for working with Spaces via natural language
- **[Understanding Copilot Context](../understanding-copilot-context/)** — How Copilot uses context from code, workspace, and conversation
- **[Defining Custom Instructions](../defining-custom-instructions/)** — Repo-level instructions that complement Spaces
- **[Building Custom Agents](../building-custom-agents/)** — Create agents that can load and act on Space context

---

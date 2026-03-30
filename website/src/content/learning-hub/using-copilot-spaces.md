---
title: 'Using Copilot Spaces'
description: 'Learn how to use GitHub Copilot Spaces to bring curated, project-specific context into your conversations and ground Copilot responses in your team's actual knowledge.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-03-30
estimatedReadingTime: '9 minutes'
tags:
  - spaces
  - context
  - fundamentals
  - collaboration
relatedArticles:
  - ./understanding-copilot-context.md
  - ./understanding-mcp-servers.md
  - ./building-custom-agents.md
prerequisites:
  - Basic understanding of GitHub Copilot
  - GitHub Copilot CLI installed
---

Copilot Spaces are shared knowledge bases that bring curated, project-specific context into your Copilot conversations. A Space bundles repositories, files, documentation, and custom instructions into a single unit—so when you reference a Space, Copilot's responses are grounded in your team's actual code, standards, and knowledge rather than generic training data.

This article explains what Spaces are, how to discover and use them, and how to create and manage your own.

## What Is a Copilot Space?

A Space is a named collection of context resources that you or your organization curate once and share with the whole team. When you load a Space into a conversation, Copilot uses its contents—attached documentation, code context, and custom instructions—to give you answers specific to your project.

**What a Space can contain**:
- **GitHub files**: Specific files from one or more repositories
- **GitHub issues**: Tracked work items for ongoing context
- **Free text**: Inline documentation, guidelines, and notes written directly in the Space
- **Custom instructions**: Behavioral directives that change how Copilot responds when the Space is active

**Why Spaces are useful**:
- **Consistent context for teams**: Everyone gets the same grounding, not just the developer who knows which files to open
- **Onboarding acceleration**: New developers can ask Copilot questions using the Space and get project-accurate answers immediately
- **Workflow templates**: Spaces can contain step-by-step process instructions that Copilot follows as a workflow engine
- **Always current**: Spaces auto-update as the underlying repos change

### Spaces vs. Instructions vs. Skills

| Feature | Instructions | Skills | Spaces |
|---------|-------------|--------|--------|
| Scope | File-pattern based, auto-applied | Task-based, invoked on demand | Conversation-based, loaded explicitly |
| Content | Markdown rules and standards | Instructions + bundled assets | Repos, files, issues, and free text |
| Managed by | Developers (repo files) | Developers (repo folders) | Users or organizations (GitHub.com) |
| Best for | Coding standards, style guides | Repeatable task workflows | Project knowledge bases, onboarding |

## Discovering Available Spaces

When working with an agent that has the GitHub MCP server connected, you can ask Copilot to list available Spaces:

```
What Copilot Spaces are available for our team?
```

The agent calls `mcp__github__list_copilot_spaces`, which returns all Spaces the current user can access. Each result includes a `name` and `owner_login` identifying who owns the Space (a user or an organization).

You can also filter for a specific context:

```
Find a Copilot Space for our API documentation
```

## Loading a Space

Once you know which Space you need, ask Copilot to load it:

```
Load the Accessibility Copilot Space
```

The agent calls `mcp__github__get_copilot_space` with the owner and name, then uses the returned content to ground its responses. Space names are **case-sensitive**.

### Spaces as Workflow Engines

Spaces aren't just reference material—they can define structured workflows that Copilot follows step by step. For example, a "PM Weekly Updates" Space might contain a template format and instructions for gathering data from tracking issues, then drafting a report.

When you load a workflow Space, Copilot:
1. Reads the instructions and template
2. Fetches external resources referenced by the Space (issues, dashboards, etc.)
3. Follows the defined steps, showing progress after each one
4. Produces the final output in the format the Space specifies

## Managing Spaces with the REST API

The GitHub REST API gives you full CRUD (create, read, update, delete) control over Spaces. Use the `gh api` command to manage Spaces programmatically.

> **Note**: The Copilot Spaces API requires the `read:user` scope for reads and the `user` scope for write operations. Run `gh auth refresh -h github.com -s user` if needed.

### Creating a Space

**Personal Space** (visible to you):
```bash
gh api users/{username}/copilot-spaces \
  -X POST \
  -f name="My Project Space" \
  -f description="Context for my project" \
  -f general_instructions="Help me with..." \
  -f visibility="private"
```

**Organization Space** (shared with your org):
```bash
gh api orgs/{org}/copilot-spaces \
  -X POST \
  -f name="Engineering Onboarding" \
  -f description="Context for new engineering hires" \
  -f general_instructions="Answer questions about our tech stack and architecture"
```

### Listing and Finding Spaces

```bash
# List your personal spaces
gh api users/{username}/copilot-spaces

# List org spaces
gh api orgs/{org}/copilot-spaces
```

### Updating a Space

Get the Space's `number` from the list endpoint first, then update:

```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  -f name="Updated Name" \
  -f description="Updated description" \
  -f general_instructions="Updated instructions here"
```

**Updatable fields**: `name`, `description`, `general_instructions`, `icon_type`, `icon_color`, `visibility` (`"private"` or `"public"`), `base_role` (`"no_access"` or `"reader"`), `resources_attributes`

### Attaching Resources

Resources define the content that gets loaded into context when the Space is used. **This replaces the entire resource list**, so include all existing resources plus any new ones:

```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  --input - <<'EOF'
{
  "resources_attributes": [
    {
      "resource_type": "free_text",
      "metadata": { "name": "Architecture Notes", "text": "Our backend uses..." }
    },
    {
      "resource_type": "github_issue",
      "metadata": { "repository_id": 12345, "number": 42 }
    },
    {
      "resource_type": "github_file",
      "metadata": { "repository_id": 12345, "file_path": "docs/architecture.md" }
    }
  ]
}
EOF
```

To remove a resource, include it with `"_destroy": true`:
```json
{ "id": 123, "_destroy": true }
```

### Deleting a Space

```bash
gh api users/{username}/copilot-spaces/{number} -X DELETE
```

### Managing Collaborators

Add, list, update, and remove collaborators using the collaborators sub-resource:

```bash
# List collaborators
gh api users/{username}/copilot-spaces/{number}/collaborators

# Add a collaborator
gh api users/{username}/copilot-spaces/{number}/collaborators \
  -X POST \
  -f username="collaborator-login" \
  -f role="reader"
```

## Using the Copilot Spaces Skill

The [copilot-spaces](../../skills/?q=copilot-spaces) community skill on Awesome Copilot teaches Copilot how to discover, load, and manage Spaces using both the MCP tools and the REST API. Install it in your repository to ensure Copilot consistently follows the best workflow when working with Spaces.

```bash
# Copy the skill into your repository
cp -r .github/skills/copilot-spaces .github/skills/
```

Or install the skill via the GitHub Copilot CLI if you have the plugin installed.

## Use Cases and Examples

### Team Onboarding Space

Create an organization Space with:
- Architecture overview (free text)
- Key repository files (README, ADRs, runbooks)
- Linked issues for current initiatives

New developers load the Space and ask Copilot project-specific questions without needing a guide.

### Compliance and Standards Space

Create a Space with your organization's:
- Security requirements (free text)
- Code review checklists (GitHub files)
- Policy documents (free text or files)

Developers load the Space when working on compliance-sensitive code to ensure Copilot's suggestions align with requirements.

### Project Dashboard Space

Combine a Space with workflow instructions to create a living dashboard:
- Link to open initiative tracking issues
- Include reporting templates (free text)
- Add instructions for the agent to gather metrics and draft a summary

Run the workflow with: "Using the Dashboard Space, generate this week's status report."

## Tips and Best Practices

- **Space names are case-sensitive**. Use the exact name from `list_copilot_spaces`.
- **Spaces can be large** (20KB+ of content). Ask Copilot to find specific sections rather than reading everything.
- **Workflow Spaces should include explicit step-by-step instructions** in `general_instructions` or attached free-text resources so Copilot can follow them consistently.
- **Combine Spaces with agents**: Create a custom agent that automatically loads a specific Space at session start for specialized workflows.
- **Public Spaces** are visible to anyone. Use `visibility: private` for sensitive internal content.
- **Spaces auto-update**: Attached repository files reflect the latest commits automatically—you don't need to re-attach files after changes.

## Further Reading

- **[Understanding Copilot Context](../understanding-copilot-context/)** — How Copilot uses context from files, workspace, and conversation
- **[Understanding MCP Servers](../understanding-mcp-servers/)** — The protocol that powers Spaces discovery in agents
- **[Building Custom Agents](../building-custom-agents/)** — Create agents that leverage Spaces for specialized workflows
- **[copilot-spaces skill](../../skills/?q=copilot-spaces)** — Community skill for working with Spaces

---

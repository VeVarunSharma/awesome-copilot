---
title: 'Using the /pr Command'
description: 'Learn how to use the /pr slash command in GitHub Copilot CLI to create pull requests, fix CI failures, address review comments, and resolve merge conflicts.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-03-18
estimatedReadingTime: '7 minutes'
tags:
  - copilot-cli
  - pull-requests
  - automation
  - fundamentals
relatedArticles:
  - ./using-copilot-coding-agent.md
  - ./agentic-workflows.md
prerequisites:
  - GitHub Copilot CLI installed and authenticated
  - Working knowledge of Git and pull requests
---

The `/pr` command is a built-in slash command in the GitHub Copilot CLI that brings AI-assisted pull request workflows directly into your terminal. It can create new PRs, automatically fix CI failures, address review feedback, and resolve merge conflicts — all without leaving your shell.

## Overview

```
/pr [subcommand] [options]
```

| Subcommand | What It Does |
|------------|-------------|
| `/pr` | Create a PR from the current branch (interactive) |
| `/pr view` | View PR status and details locally |
| `/pr view web` | Open the PR in your browser |

## Creating a Pull Request

Run `/pr` from inside a Copilot CLI session when your branch is ready to be reviewed:

```
/pr
```

Copilot will:

1. Inspect the diff between your branch and the base branch
2. Read your commit messages and any related issue references
3. Generate a descriptive PR title and body
4. Open the PR on GitHub and return the URL

You can guide the output by providing context in the prompt before running `/pr`:

```
I've implemented rate limiting on the login endpoint. The approach uses Redis sliding-window counters.

/pr
```

Copilot uses your description as additional context when generating the PR body.

### What a Generated PR Includes

A Copilot-generated PR body typically contains:

- **Summary**: What the change does and why
- **Implementation notes**: Key design decisions or trade-offs
- **Testing**: How the change was verified
- **Related issues**: Linked via `Closes #N` or `Relates to #N`

You can edit the draft before Copilot submits it, or ask Copilot to revise specific sections.

## Fixing CI Failures

When a CI run fails on your PR, open a Copilot session in your repository and use `/pr` to investigate and fix:

```
The CI pipeline is failing on my PR. Please investigate and fix.

/pr
```

Copilot will:

1. Fetch the CI failure logs
2. Identify the root cause (failing tests, lint errors, build failures, etc.)
3. Propose and apply fixes
4. Push the fixes to the existing PR branch

### Common CI Fix Scenarios

**Failing unit tests**:
```
/pr fix the failing tests in the CI run
```

**Lint errors**:
```
/pr the ESLint check is failing with unused variable warnings
```

**Build errors**:
```
/pr the TypeScript build is failing due to type errors introduced by my changes
```

## Addressing Review Comments

When reviewers leave comments on your PR, you can use Copilot to address them systematically:

```
There are review comments on my PR. Please address them.

/pr
```

Copilot reads the review comments, proposes code changes, applies them to the branch, and pushes an updated commit. You retain full control — review the diff before Copilot pushes.

### Tips for Effective Review Addressing

- **Provide context**: If a reviewer's comment is ambiguous, explain the intent before asking Copilot to address it
- **Review the output**: Always inspect the diff before accepting — Copilot's interpretation of a comment may not match the reviewer's intent
- **Iterate**: After Copilot pushes, you can ask it to refine specific changes further

## Resolving Merge Conflicts

When your branch has merge conflicts with the base branch:

```
My PR has merge conflicts with main. Please resolve them.

/pr
```

Copilot will:

1. Pull the latest base branch
2. Identify conflicting sections
3. Apply semantically correct resolutions
4. Push the resolved branch

> **Tip**: For complex conflicts involving interleaved logic changes, provide context about the intent of your changes: "My changes add rate limiting middleware that should run _before_ the authentication check added in main."

## Viewing PR Status

### Local View

```
/pr view
```

Shows the PR's current status inline in the terminal: title, description, CI status, and review state.

### Web View

```
/pr view web
```

Opens the PR in your default browser for a full GitHub.com view.

## Common Questions

**Q: Does /pr require a GitHub remote?**

A: Yes. Your repository must have a GitHub remote configured (`origin` pointing to a github.com or GitHub Enterprise repo). The CLI uses `gh` under the hood for GitHub API calls.

**Q: Can /pr create draft PRs?**

A: Yes. Tell Copilot you want a draft: "Create a draft PR — this is still a work in progress."

**Q: What if I want to write the PR description myself?**

A: Run `/pr view` after creation to open an editor. Alternatively, draft the description in your prompt before running `/pr` and Copilot will use your text as the starting point.

**Q: Does /pr work with GitHub Enterprise Server?**

A: Yes, as long as your `gh` CLI is authenticated against your GHES instance.

## Next Steps

- **Agentic Workflows**: [Agentic Workflows](../agentic-workflows/) — Automate PR-related tasks on a schedule with AI-powered Actions workflows
- **Coding Agent**: [Using the Copilot Coding Agent](../using-copilot-coding-agent/) — Let Copilot implement entire features and open PRs autonomously
- **Hooks**: [Automating with Hooks](../automating-with-hooks/) — Run linters and formatters automatically before Copilot creates a PR

---

---
title: 'Rubber Duck Mode: Getting Adversarial Feedback'
description: 'Learn how to use GitHub Copilot's Rubber Duck feature to get critical, adversarial feedback on your code and design decisions before they reach production.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-06-03
estimatedReadingTime: '6 minutes'
tags:
  - rubber-duck
  - code-review
  - copilot-cli
  - fundamentals
relatedArticles:
  - ./building-custom-agents.md
  - ./automating-with-hooks.md
  - ./before-after-customization-examples.md
prerequisites:
  - GitHub Copilot CLI installed and authenticated
---

The rubber duck debugging technique has been a developer favourite for decades: explaining your code to an inanimate rubber duck forces you to articulate your thinking clearly, often revealing bugs or design flaws in the process. GitHub Copilot brings this technique into the AI era with `/rubber-duck` — an adversarial review mode that actively challenges your code and design decisions rather than simply validating them.

## What Is Rubber Duck Mode?

The `/rubber-duck` command activates a built-in Copilot agent that takes a deliberately adversarial stance toward your work. Unlike the default Copilot experience — which is collaborative and tries to help you accomplish your goal — the Rubber Duck agent:

- Questions your assumptions and design choices
- Points out potential edge cases and failure modes
- Challenges implementation decisions with "why did you choose this?"
- Probes for security vulnerabilities and performance issues
- Surfaces hidden complexity you may have overlooked

The core insight is that adversarial feedback — even when simulated — helps you build more robust software. When you have to defend your choices against a critical reviewer, you strengthen your thinking and catch problems before they reach production.

Rubber Duck mode is **enabled by default** as of Copilot CLI v1.0.58.

## Getting Started

To activate Rubber Duck mode during an interactive Copilot CLI session:

```
/rubber-duck
```

After running the command, describe your code, design decision, or architecture. The agent will challenge it.

### Enabling and Disabling

You can control the Rubber Duck feature via `copilot config`:

```bash
# Disable rubber duck
copilot config set builtInAgents.rubberDuck false

# Re-enable rubber duck
copilot config set builtInAgents.rubberDuck true
```

## When to Use Rubber Duck

| Scenario | How Rubber Duck Helps |
|----------|----------------------|
| Code review preparation | Surface issues before sharing with teammates |
| Architecture decisions | Challenge design choices before committing to them |
| Security-sensitive code | Probe for vulnerabilities and edge cases |
| Debugging difficult problems | Articulate the problem and have your assumptions questioned |
| Pre-PR self-review | Catch edge cases before opening a pull request |
| API design | Challenge interface decisions before they become permanent |

Rubber Duck mode is especially valuable when:
- You are working on a complex feature and want a sanity check
- You are about to make a significant design decision
- You are reviewing security-sensitive code
- You are debugging a subtle problem and need to think out loud

## Example: Rubber Duck in Action

Here is what a Rubber Duck session might look like:

```
> /rubber-duck

I'm implementing a rate limiter for the login API. It uses Redis to track
request counts per IP with a fixed 5-minute window and a maximum of 10 requests.

Rubber Duck: Several things to probe here. First, why a fixed window rather
than a sliding window? A user could make 10 requests at 11:59pm and another
10 at 12:00am — that's 20 requests in two minutes, effectively bypassing your
limit. Is that an acceptable tradeoff?

Also, how are you handling Redis connection failures? If Redis goes down, does
your rate limiter fail open (allow all requests) or fail closed (block all)?
Either choice has significant security implications that you should decide
explicitly rather than by accident.

Finally, you're tracking by IP address alone. What about users behind NAT or
corporate proxies who share an IP? You may be rate-limiting legitimate users
who happen to share an address with a bad actor. Have you considered pairing
IP with user ID for authenticated requests?
```

Notice how the Rubber Duck doesn't just validate your approach — it probes the assumptions, edge cases, and tradeoffs you may not have considered. That's the point.

## Rubber Duck vs Standard Copilot

| | Standard Copilot | Rubber Duck Mode |
|---|---|---|
| **Tone** | Helpful, collaborative | Adversarial, critical |
| **Goal** | Help you complete a task | Help you find problems |
| **Feedback style** | "Here is how to improve this" | "Why did you make this choice?" |
| **Best for** | Building things | Validating and hardening things |

Think of Rubber Duck as a mode switch from *implementation assistant* to *adversarial reviewer*. Both are valuable — use them at different stages of development.

## Combining Rubber Duck with Voice Input

GitHub Copilot CLI v1.0.59 introduced the `/voice` command, which lets you **dictate prompts using local speech-to-text models** instead of typing. Voice input pairs naturally with Rubber Duck sessions — you can explain your code or design verbally, the way you'd actually talk to a rubber duck, without interrupting your flow to type.

To start voice input:

```
/voice
```

The CLI listens using a local on-device speech-to-text model. Your audio is processed locally — it is not sent to a cloud transcription service — and the transcribed text is placed in your prompt ready to send.

### When to Use Voice Input

- During Rubber Duck sessions to explain complex designs conversationally
- When you want to quickly articulate context without switching to the keyboard
- In longer sessions where typing slows down your thinking

> **Note**: `/voice` requires a supported local speech-to-text model. See the [Copilot CLI releases page](https://github.com/github/copilot-cli/releases/tag/v1.0.59) for details on supported models and platform requirements.

## Scheduled Prompts (Experimental)

Copilot CLI v1.0.58 introduced scheduled prompt execution as an experimental feature. You can use `/every` and `/after` to schedule prompts that run automatically:

```
/experimental on
/every 30m check if there are any new issues opened in the repo
/after my tests pass summarize the changes I made
```

Enable the experimental flag first with `/experimental on`. Scheduled prompts run asynchronously in the background and surface results in your session when they complete.

> **Note**: Scheduled prompts are experimental and the API may change in future releases.

## Further Reading

- [GitHub Copilot CLI Releases](https://github.com/github/copilot-cli/releases) — full changelog for v1.0.56–v1.0.59 where these features landed
- [Building Custom Agents](../building-custom-agents/) — create your own specialized review agents with custom adversarial personas
- [Automating with Hooks](../automating-with-hooks/) — add deterministic guardrails alongside Rubber Duck reviews
- [Before/After Customization Examples](../before-after-customization-examples/) — see code review workflow patterns in practice

---

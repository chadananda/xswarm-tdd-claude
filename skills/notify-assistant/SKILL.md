---
name: notify-assistant
description: Notify installed AI assistants (OpenClaw) of completed features. Use after finishing a task, feature, or deployment to keep the user's personal AI assistant informed of project progress. Trigger when user says "notify assistant", "tell OpenClaw", "update my assistant", or after major task completion.
---

# Notify AI Assistant of Completed Work

Send a structured notification to the user's personal AI assistant so it stays informed about project progress. Currently supports OpenClaw; designed to be extensible to other assistants.

## Why This Matters

Personal AI assistants like OpenClaw maintain long-term context about your projects, preferences, and goals. When Claude Code finishes a feature, your assistant doesn't know unless you manually tell it. This skill bridges that gap automatically --- your coding agent informs your personal agent, so your assistant can track progress, remind you of next steps, or answer questions about recent changes without you playing messenger.

## Detection

Check for OpenClaw in this order:

1. **CLI available:** `which openclaw` succeeds
2. **Config exists:** `~/.openclaw/openclaw.json` is present
3. **Gateway running:** `curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:18789/ 2>/dev/null` returns a response

If none of these pass, skip notification silently --- the user may not have an assistant installed.

## Sending the Notification

### Via CLI (preferred --- simplest, works if daemon is running)

```bash
openclaw agent --message "PROJECT UPDATE: [message]"
```

### Via message send (if targeting a specific channel)

```bash
openclaw message send --message "PROJECT UPDATE: [message]"
```

## Message Format

Keep notifications concise and structured. The assistant needs facts, not prose:

```
PROJECT UPDATE: [repo-name]

Completed: [what was done — 1-2 sentences]
Files changed: [count] ([key files])
Tests: [passing/failing] ([count] tests)
Commit: [short hash] on [branch]
Next: [what's planned next, if known]
```

**Example:**

```
PROJECT UPDATE: xswarm-tdd

Completed: Added BDD/Playwright skill with ARIA-first locators and axe-core accessibility audits.
Files changed: 4 (agents/TDD.md, agents/reviewer.md, agents/tester.md, skills/bdd-playwright/SKILL.md)
Tests: all passing (47 tests)
Commit: 2e95414 on main
Next: Security scan integration with xswarm-ai-sanitize
```

## When to Use

- After completing a major feature or task group
- After a deployment or release
- After a significant refactor
- When the user explicitly asks to notify their assistant

## Workflow

1. **Detect** --- Check if OpenClaw (or another assistant) is available
2. **Compose** --- Build a structured message from the current git state and task context
3. **Send** --- Deliver via CLI
4. **Confirm** --- Report to the user that the notification was sent (or skipped if no assistant found)

### Quick recipe for composing from git state

```bash
# Get repo name
REPO=$(basename "$(git rev-parse --show-toplevel)")

# Get last commit info
COMMIT=$(git log -1 --format="%h %s")

# Get changed files count
FILES=$(git diff --name-only HEAD~1 | wc -l | tr -d ' ')

# Get key files
KEY_FILES=$(git diff --name-only HEAD~1 | head -5 | tr '\n' ', ')
```

## Extensibility

This skill is designed to support additional assistants. To add a new one:

1. Add a detection check (CLI, config file, or running process)
2. Add a send method (CLI command, API call, or WebSocket message)
3. Follow the same structured message format

Potential future targets: custom webhooks, Slack bots, ntfy.sh, Home Assistant.

## Remote Question Routing (Stuck Agent Integration)

The most powerful use of this skill is surfacing stuck agent questions to OpenClaw so you can answer from your phone while Claude works unattended. This turns "blocked waiting at terminal" into "pinged on WhatsApp."

### How It Works

When the stuck agent fires, it can notify OpenClaw *before* blocking in the terminal:

1. **Stuck agent detects a problem** and prepares the question
2. **Stuck agent checks for OpenClaw** via `which openclaw`
3. **If found, sends the question** as a high-priority notification:
   ```bash
   openclaw agent --message "CLAUDE CODE NEEDS INPUT: [repo-name]

   Problem: [brief description]
   Options:
   1. [option A]
   2. [option B]
   3. [option C]

   Reply with your choice or come back to the terminal."
   ```
4. **Stuck agent also blocks in terminal** via AskUserQuestion as usual
5. **User responds** from whichever is more convenient --- phone or terminal

### Message Format for Questions

```
CLAUDE CODE NEEDS INPUT: [repo-name]

Problem: [1-2 sentence description]
Context: [error message or key detail]
Options:
1. [option] — [brief explanation]
2. [option] — [brief explanation]
3. [option] — [brief explanation]

Reply with a number or your own answer.
```

### Limitations

- Currently one-way notification: OpenClaw alerts you, but the response still comes through the terminal's AskUserQuestion. Full round-trip (answer via messaging app, response flows back to Claude Code) requires OpenClaw webhook/callback support --- a future enhancement.
- If OpenClaw is not running, the stuck agent falls back silently to terminal-only mode.

## Rules

- **Never send notifications without the user requesting it or the workflow calling for it.** Unsolicited messages to external systems violate the principle of least surprise.
- **Skip silently if no assistant is detected.** Not everyone has OpenClaw installed; failing loudly would be annoying noise.
- **Keep messages factual and structured.** The receiving assistant needs parseable information, not conversational filler.
- **Never include secrets, tokens, or credentials in notifications.** Messages may traverse external channels; treat them as potentially public.
- **Always still block in terminal via AskUserQuestion.** OpenClaw notification is an alert, not a replacement for the terminal response path. The pipeline must not proceed without a confirmed human decision.

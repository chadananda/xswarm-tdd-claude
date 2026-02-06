# Agents & Skills

## Agents

Agents in `.claude/agents/` are auto-loaded. Key agents:
- **coder** - Implementation (follows TDD.md)
- **reviewer** - Code review + refactoring (merged critic+refactor)
- **tester** - Unit test verification + visual testing (web/TUI)
- **doc** - Documentation (JSDoc + README)
- **team-lead** - Orchestrates mini-team per task (coder -> reviewer -> tester -> doc)
- **stuck** - Human escalation (only agent that pauses workflow)

## Skills Reference

Invoke with `/skill-name`:
- **/plan** - Requirements gathering and task decomposition
- **/frontend-design** - Distinctive, production-grade UI with bold aesthetics
- **/security-scan** - Detect secrets before commit (mandatory for coder agent)
- **/project-cleanup** - Organize messy project structure

## Project Type Detection

**TUI Projects** (textual, blessed, ink, bubbletea, etc.):
- **NEVER run TUI in main terminal.** TUI apps take over stdin/stdout and corrupt the orchestrator's terminal state, making recovery impossible without restarting.
- **Always use tmux/screen/background or VHS for testing.** Isolation keeps the orchestrator responsive and produces reproducible visual artifacts.
- See tui-viewer skill for visual verification.

**Web Projects** (package.json, playwright.config):
- **Use Playwright MCP or webapp-testing skill for browser testing.** Browser automation provides deterministic, repeatable verification that manual inspection cannot.

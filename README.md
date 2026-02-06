# xswarm-tdd-claude

> *"I spent months yelling at an AI about Test-Driven Development. This is what survived."*

Hi. This is my actual `~/.claude` folder --- the one I use every day for real work. It's two things at once:

1. **My global Claude Code config** --- a multi-agent orchestration system that makes Claude do TDD whether it wants to or not. Twenty curated skills, custom hooks, and hard-won opinions about how AI should write code. It's opinionated. It's been through hundreds of iterations. It works.

2. **The experimental playground for [xswarm](https://xswarm.ai) coding agents** --- I'm spending 2026 building autonomous agent swarms for software development, and this is where those ideas get prototyped and battle-tested before they graduate into the xswarm framework. The team-lead pipeline, the stuck protocol, the TDD enforcement architecture --- these all started here as experiments and evolved into patterns I trust enough to ship.

I'm sharing it because open source is good karma, and because the best way to build better AI coding agents is to do it in the open where smart people can tell you what you're doing wrong.

**Author:** Chad Jones / [xswarm.ai](https://xswarm.ai) / [chadananda@gmail.com](mailto:chadananda@gmail.com)

---

## The Problem (or: Why Claude Needs a Babysitter)

Claude Code is *brilliant*. It's also a golden retriever that desperately wants to please you. Ask it to do TDD and it will cheerfully write the test and the implementation in the same breath, skip the red phase entirely, and beam at you like "look what I made!" It's endearing. It's also not TDD.

And here's the uncomfortable truth nobody talks about: **without very specific instructions, Claude Code treats you as its free human QA department.** It'll write code confidently, hand it to you, and expect *you* to test it, find the bugs, report back, and iterate. You become the unpaid tester in an infinite feedback loop. The AI writes, you verify. The AI guesses, you correct. The AI moves fast, you clean up.

We're trying to invert that relationship entirely. *Claude* writes the tests. *Claude* verifies its own work. *Claude* catches its own bugs. You make decisions. You approve plans. You don't manually test anything.

Left to its own devices, Claude will also:

- Write 200 lines when 40 would do (it's *generous*)
- Generate tests that suspiciously mirror the implementation rather than driving it
- "Forget" about test-first after the third feature because the conversation is now 200k tokens long
- Guess when confused instead of asking you (it doesn't want to bother you, bless its heart)

You can put "DO TDD" in your system prompt. It works... for a while. Then context pressure wins and Claude reverts to its helpful-but-undisciplined self. Prompt engineering is necessary but not sufficient.

This system solves the problem *architecturally*. You don't ask Claude to do TDD. You make it structurally impossible to skip.

---

## How It Actually Works

### `CLAUDE.md` --- The World's Tiniest Boss

```
CLAUDE.md (8 lines total. Seriously.)
  |
  +-- @CLAUDE_conventions.md   "where stuff goes"
  +-- @CLAUDE_workflow.md      "how to work"
  +-- @CLAUDE_agents.md        "who does what"
```

The orchestrator's entire job is routing. Simple task? Handle it. Complex project? Call `/plan`. Tasks come back? Farm them out to team-leads running in parallel. That's it. Eight lines. Every byte in CLAUDE.md gets loaded into every conversation, so it's on a strict diet.

### `/plan` --- The Planning Skill That Replaced 1,200 Lines of Pain

The old planning system was a 1,213-line agent that wrote JSON files to a `context/` directory. It worked! It also burned more tokens per invocation than the *entire current system combined*. RIP.

The `/plan` skill is 180 lines. It gathers requirements (with your approval before a single line of code gets written), decomposes work into tasks with proper dependency ordering, and includes an anti-hallucination checklist so the coder agent doesn't try to `import magic_module_that_doesnt_exist`.

### The Agent Pipeline --- A Tiny Software Company

Each task gets its own team. Think of it as the world's smallest and most obedient dev shop:

```
    +-------------+     +------------+     +----------+     +-------+
    |   CODER     | --> |  REVIEWER  | --> |  TESTER  | --> |  DOC  |
    | "build it"  |     | "fix it &  |     | "prove   |     | "write|
    | (strict TDD)|     |  shrink it"|     |  it"     |     |  it up|
    +-------------+     +------------+     +----------+     +-------+
          |                   |                  |               |
          v                   v                  v               v
      Tests MUST          Tests run           Conditional:    Always
      fail first,         after EVERY         Web/TUI only.   runs.
      then pass.          single change.      Libraries get   Every.
      No exceptions.      Code shrinks        a free pass     Single.
                          until it stops.     (already tested Time.
                                              twice).
```

**The Coder** gets a complete spec and builds exactly what it says. No research, no architectural decisions, no creative liberties. Just TDD. Red, green, refactor. It's a *very focused* employee.

**The Reviewer** used to be two agents (critic + refactor) that played telephone with each other. Now it's one agent that reviews, fixes, and minimizes in a single pass. Tests run after *every* change. If code doesn't shrink, it stops trying. Efficient.

**The Tester** only shows up for web and TUI projects where visual correctness matters. For a library of pure functions? The coder already tested it (TDD, remember) and the reviewer re-tested it after every change. That's two verification passes. Playwright would just be overkill.

**The Doc Agent** writes JSDoc and a README for every deliverable. Always. No exceptions. Future-you will thank present-you.

### The Stuck Protocol --- Claude's Panic Button

> *Every* agent is hardwired to hit the panic button the moment anything goes wrong.

No fallbacks. No "let me try something else" loops. No silent failures where Claude wanders off into the weeds for 10 minutes. The `stuck` agent is the *only* agent allowed to ask you questions. It presents the problem clearly with options. You make a 5-second decision. Work resumes.

It sounds rigid. In practice, it's the best part of the whole system. Problems surface immediately instead of compounding silently. It's like a GPS that says "I'm lost" instead of confidently driving you into a lake.

This is the other half of inverting the human-AI testing relationship. Without the stuck protocol, Claude guesses when confused and hands you broken output to debug. *You* become the debugger. With the stuck protocol, Claude stops the moment it's unsure and asks you a *specific question with options*. You make a 5-second decision, not a 20-minute debugging session. Your role shifts from "unpaid QA" to "technical decision-maker." Which is what you should have been doing all along.

**Error handling is tiered:**

| Tier | What Happens | Example |
|------|-------------|---------|
| **Tier 1: Self-fix** | Agent fixes it. One attempt, 30 seconds max. | Typo, wrong path, missing import |
| **Tier 2: Stuck** | Instant escalation. Human decides. | Design ambiguity, missing spec, 2+ failed attempts |

---

## The TDD Protocol (The Hill I Will Die On)

Lives at `agents/TDD.md`. Loaded by every coding agent via `@TDD.md`. Non-negotiable.

```
                    +--------+
                    |  RED   |  Write ONE failing test.
                    |  ....  |  Watch it fail. (This is the important part.)
                    +---+----+
                        |
                        v
                    +--------+
                    | GREEN  |  Write the MINIMUM code to make it pass.
                    |  ....  |  Not the code you think you'll need later.
                    +---+----+  The minimum. Seriously.
                        |
                        v
                    +--------+
                    |REFACTOR|  Clean up. Tests still pass?
                    |  ....  |  Good. If not, UNDO.
                    +---+----+
                        |
                        +-----> Repeat forever. One test at a time.
```

**The Seven Commandments:**

1. **Never write implementation without a failing test.** (Yes, *never*. Not "usually." *Never.*)
2. **One test at a time.** Not two. Not "a few related ones." One.
3. **Run tests at every phase.** Don't say "this would fail" --- prove it.
4. **Minimum code to pass.** Future tests will ask for more. Wait for them.
5. **Fix the code, not the test.** Only change a test if the expectation was genuinely wrong. State why first.
6. **Refactoring changes structure, not behavior.** Test breaks during refactor? Undo. No debate.
7. **Report each phase.** RED: what failed. GREEN: what changed. REFACTOR: what improved.

### Why Not Just Say "Do TDD" in the Prompt?

Because by the fifth feature, that instruction is buried under 200k tokens of conversation history and Claude has quietly reverted to its natural "write everything at once" instinct. Prompts drift. Architecture doesn't.

Each agent invocation gets a *fresh* context with TDD loaded at high priority. The coder's entire identity is "implement from spec using TDD." It doesn't have competing instructions about planning or reviewing or being creative. Separation of concerns isn't just for code --- it's for attention.

---

## What's In the Box

```
~/.claude/
|
|-- CLAUDE.md                     # 8-line orchestrator (it delegates)
|-- CLAUDE_conventions.md         # "put files HERE, not THERE"
|-- CLAUDE_workflow.md            # "work like THIS"
|-- CLAUDE_agents.md              # "these are your coworkers"
|-- settings.json                 # Permissions, hooks, status line
|-- notify.py                     # Dings your desktop when agents finish
|-- .gitignore                    # Keeps Claude's runtime mess out of git
|
|-- agents/                       # The team
|   |-- TDD.md                   # The shared religion (39 lines)
|   |-- coder.md                 # Builder (95 lines, down from 513)
|   |-- reviewer.md              # Critic + minimizer (113 lines)
|   |-- tester.md                # Verifier, shows up when needed (105 lines)
|   |-- team-lead.md             # Manager, runs on Opus (111 lines)
|   |-- doc.md                   # Documentarian (390 lines, it's thorough)
|   +-- stuck.md                 # The panic button (144 lines)
|
|-- commands/                     # Slash commands (/commit, /review, etc.)
|   |-- commit.md                # Trunk-style git workflow
|   |-- review.md                # Multi-agent code review
|   |-- seo.md                   # Markdown SEO analysis
|   |-- validate-product.md      # Startup idea validation
|   +-- cleanup-tmp.md           # Janitor duty
|
|-- hooks/                        # Automated guardrails
|   |-- warn-root-files.py       # "HEY! Don't put that in root!"
|   +-- cleanup-tmp-scripts.py   # Tidies up after tasks finish
|
+-- skills/                       # 20 on-demand capabilities
    |
    |-- [Development Workflow]
    |   |-- plan/                 # Requirements + task decomposition
    |   |-- bdd-playwright/       # Gherkin + ARIA locators + axe-core a11y
    |   |-- systematic-debugging/ # 4-phase root cause analysis (Superpowers)
    |   |-- using-git-worktrees/  # Parallel branch isolation (Superpowers)
    |   |-- property-based-testing/ # Hypothesis/QuickCheck (Trail of Bits)
    |   +-- security-scan/        # Gitleaks secret detection
    |
    |-- [Language & Framework]
    |   |-- modern-python/        # uv/ruff/ty tooling (Trail of Bits)
    |   |-- frontend-design/      # Bold UI, no "AI slop" (Anthropic)
    |   +-- web-artifacts-builder/ # React/Tailwind/shadcn (Anthropic)
    |
    |-- [Documents & Formats]
    |   |-- doc-coauthoring/      # Structured doc writing (Anthropic)
    |   |-- docx/                 # Word docs (Anthropic)
    |   |-- pdf/                  # PDFs (Anthropic)
    |   |-- pptx/                 # PowerPoint (Anthropic)
    |   +-- xlsx/                 # Spreadsheets (Anthropic)
    |
    +-- [Meta & Tooling]
        |-- agent-builder/        # Build your own agents
        |-- skill-creator/        # Build your own skills (Anthropic)
        |-- mcp-builder/          # MCP server dev guide (Anthropic)
        |-- project-cleanup/      # Organize messy projects
        |-- tui-viewer/           # TUI screenshot verification
        +-- webapp-testing/       # Playwright browser testing (Anthropic)
```

### Skills: They're Lazy (In a Good Way)

Skills only wake up when they're needed. Claude scans each skill's description (~100 tokens) and ignores the rest until it detects relevance. Twenty skills cost almost nothing at idle. The trick is proper scoping in each skill's `description` field:

| Category | Skills | When They Wake Up |
|----------|--------|-------------------|
| **Coding workflow** | bdd-playwright, systematic-debugging, git-worktrees, property-based-testing, security-scan | BDD/E2E tests, bugs, branch work, test writing, pre-commit |
| **Language-specific** | modern-python, frontend-design, web-artifacts-builder | Only their language/framework |
| **Document formats** | docx, pdf, pptx, xlsx, doc-coauthoring | Only when touching those formats |
| **Meta/tooling** | plan, skill-creator, agent-builder, etc. | Rare, specific triggers |

**Skill sources and licenses:**
- [Anthropic](https://github.com/anthropics/skills) --- the official collection
- [Superpowers](https://github.com/obra/superpowers) (MIT, 46K+ stars) --- battle-tested dev workflows
- [Trail of Bits](https://github.com/trailofbits/skills) (CC BY-SA 4.0) --- security-focused excellence

---

## Getting Started

### The Quick Way

```bash
# Back up your existing config (if any)
[ -d ~/.claude ] && mv ~/.claude ~/.claude.backup

# Clone this repo as your global config
git clone https://github.com/chadananda/xswarm-tdd-claude.git ~/.claude
```

### The Picky Way

Cherry-pick what you want. The system is modular:
- Just want TDD? Grab `agents/TDD.md` and reference it from your agents.
- Just want the agent pipeline? Take the `agents/` folder.
- Just want skills? Copy individual skill folders into your `~/.claude/skills/`.
- Want it all? Clone the whole thing.

### Requirements

- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** CLI, installed and authenticated
- **`gitleaks`** for secret scanning (`brew install gitleaks` on macOS)
- *Optional:* Playwright MCP for web testing, VHS for TUI testing

### Configuration Notes

**`settings.json`** has three concerns:

- **Permissions:** One `"Bash"` entry allows all shell commands. (An earlier version had 24 specific entries that were all redundant. Ask me how I know.)
- **Hooks:** Block writes to `/tmp` (use `./tmp/` instead), warn on root-level file creation, desktop notification when agents finish, cleanup after task completion.
- **Status line:** Directory, model, version, git branch at a glance.

---

## Design Decisions (a.k.a. Mistakes I Made So You Don't Have To)

<details>
<summary><b>Why merge Critic + Refactor into one Reviewer?</b></summary>

The original pipeline was: coder -> tester -> critic -> coder (again!) -> refactor -> doc. Six steps. The critic would write a review, then the coder would be re-invoked to apply suggestions (often missing context), then the refactorer would minimize (without the review context). It was like a game of telephone but with tokens.

The merged Reviewer does all three jobs in one invocation with full context. Review, fix, minimize. Tests after every change. Half the token cost, better results. Sometimes less really is more.
</details>

<details>
<summary><b>Why is the Tester conditional?</b></summary>

For a library of pure functions, spinning up Playwright to "visually verify" is like hiring a food photographer to check if your toast is done. The coder already tested it (TDD). The reviewer re-tested after every change. That's two passes. The tester only adds value when *visual correctness* is the feature --- web projects and TUIs.
</details>

<details>
<summary><b>Why Opus for the Team-Lead but Sonnet for everyone else?</b></summary>

The team-lead makes *judgment calls*: Is this a web project? Is the coder's output complete enough? Should we invoke the tester? Stronger reasoning pays off here. The coding agents are executing detailed specs mechanically --- they don't need to reason about *what* to do, just do it well. Sonnet is perfect for that.
</details>

<details>
<summary><b>Why a shared TDD.md file instead of inline rules?</b></summary>

Earlier versions embedded TDD rules in each agent. Maintenance nightmare --- update one, forget the others. Plus the same 40 lines loaded three times in the pipeline. Now it's one file, `@TDD.md`, loaded by agents that need it, ignored by the rest. DRY applies to agent configuration too.
</details>

<details>
<summary><b>Why Claude's native Task system instead of JSON files?</b></summary>

The Task system already provides DAG dependencies, status tracking, and blocking semantics. The old approach reimplemented all of that with JSON files and custom checklists. We deleted ~1,600 lines of planning machinery and replaced it with 180. When the platform already does the thing, let the platform do the thing.
</details>

---

## The Great Consolidation (How 4,100 Lines Became 1,300)

This system grew organically and then went on a diet. Here's the before/after:

| What | Before | After | What Happened |
|------|--------|-------|---------------|
| Planning agent | 1,213 lines | 0 | Replaced by 180-line `/plan` skill. RIP. |
| Critic agent | 299 lines | 0 | Merged into Reviewer. Two jobs, one agent. |
| Refactor agent | 344 lines | 0 | Also merged into Reviewer. Three jobs, one agent. |
| Coder agent | 513 lines | 95 lines | Cut the novel-length examples. |
| Tester agent | 456 lines | 105 lines | Added unit test mode, condensed visual testing. |
| Team-lead | 356 lines | 111 lines | 4-step pipeline, dropped the redundant second coder pass. |
| CLAUDE.md | 122 lines | 8 + 83 lines | Split into topic files. The orchestrator *orchestrates*. |
| settings.json | 93 lines | 70 lines | Turns out `"Bash"` covers all bash commands. Who knew. |
| Context directory | ~400 lines | 0 | Stale artifacts from the old planning system. Goodbye. |
| Gemini API scripts | ~250 lines | 0 | Opus 4.6 replaced Gemini for planning. Progress. |

**The lesson:** Agent files get loaded into context on every invocation. A 500-line agent with extensive examples burns 2,000+ tokens before doing any actual work. Those examples were useful while developing the system. Once it was stable, they were just expensive nostalgia.

---

## The xswarm Connection

This repo is the petri dish for [xswarm](https://xswarm.ai) --- an autonomous agent swarm framework I'm building in 2026. The idea is simple but ambitious: instead of one AI assistant that does everything, you orchestrate *teams* of specialized agents that hold each other accountable.

Everything in this repo started as an experiment:

- **The team-lead pipeline** was "what if one agent managed others instead of doing all the work?" Turns out managers work better when they can't write code --- they focus on routing and quality instead of getting lost in implementation details.
- **The stuck protocol** was "what if agents couldn't guess?" This single constraint eliminated more wasted time than any other change. It's now a core xswarm pattern.
- **The TDD enforcement** was "what if test-first was structural, not aspirational?" Making it the only path through the pipeline --- not a suggestion in a prompt --- changed everything.
- **The consolidation from 4,100 to 1,300 lines** taught us that token efficiency is an architecture concern, not an optimization. Agent instructions should be as small as possible because they're loaded on every invocation.

If you're interested in where this is heading, watch the [xswarm.ai](https://xswarm.ai) space. The patterns here are graduating into something more general --- multi-agent orchestration for any software development task, not just Claude Code. Same philosophy: small agents, strict protocols, humans make decisions, robots do work.

---

## Making It Your Own

**Don't like strict TDD?** Edit `agents/TDD.md`. Keep test-first but drop one-at-a-time. Or go wild and delete it entirely (I'll try not to cry).

**Want different models?** Each agent has a `model:` field in its frontmatter. Swap `sonnet` for `opus` where you want brains, or `haiku` where you want speed. Doc is a good haiku candidate.

**Want to add agents?** Create a `.md` file in `agents/` with YAML frontmatter (name, description, tools, model). Reference it from team-lead's pipeline or invoke it ad-hoc.

**Want project-specific behavior?** Create a `.claude/CLAUDE.md` in your project. Project-level instructions override global ones. Great for project-specific test commands, directory structures, or "we use tabs here and I will fight you about it."

---

## Things I Learned the Hard Way

**Constraints make agents reliable, not limited.** The coder *can't* make architectural decisions because it doesn't have the context or instructions. This isn't a bug. This is the feature.

**Token budgets are architecture decisions.** That 1,200-line planning agent consumed more tokens per invocation than the entire current system combined. Conciseness isn't aesthetics --- it's the difference between running 10 tasks and running 3.

**The stuck protocol is the most important agent.** Everything else is tunable. The guarantee that problems surface immediately instead of compounding silently? That's what makes you trust the system enough to go make coffee while it works.

**Parallel execution is free performance.** Properly decomposed tasks with dependency mapping run in parallel at the same token cost --- they just finish sooner. Planning overhead pays for itself on any project with 3+ tasks.

**Documentation should be mandatory.** Making doc the final pipeline step means every deliverable arrives documented. Six months later, both you *and* Claude can understand what was built and why.

---

## License

MIT. Take what you want. Credit appreciated but not required. If it helps you build something cool, that's payment enough.

---

<p align="center">
<i>Built with Claude Code. Refined through hundreds of iterations. Shared in the spirit of "we're all figuring this out together."</i>
<br><br>
<b>Chad Jones</b> -- <a href="https://xswarm.ai">xswarm.ai</a>
<br>
Contributions, ideas, and strongly-worded opinions about testing welcome.
</p>

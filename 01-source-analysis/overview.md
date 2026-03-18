# Source Repository Analysis — Overview

**Project:** Everything Claude Code (ECC)  
**Version Analyzed:** 1.8.0  
**Analysis Date:** 2026-03-17  
**Category:** Cloud-based agentic coding assistant harness  

---

## 1. What Is ECC?

Everything Claude Code is a **production-grade agent harness performance system** originally built for Claude Code (Anthropic's AI coding assistant). It won the Anthropic Hackathon and has since expanded to support Cursor, OpenCode, Codex, and other AI agent harnesses.

ECC is **not a standalone application** — it is a plugin/overlay installed into an existing AI coding harness. It provides:

- **25 specialized sub-agents** — each a standalone AI system with a defined role, tool set, and model preference
- **100+ domain skills** — reusable knowledge modules injected into context on demand
- **Lifecycle hooks** — event-driven automation at 6 lifecycle points
- **52 slash commands** — explicit user-facing workflows
- **20+ MCP server integrations** — external service connectors
- **Always-follow rules** — guidelines injected permanently into the AI's system context

---

## 2. Architecture Classification

| Dimension | Value |
|---|---|
| Architecture style | Plugin-based agent orchestration |
| Orchestration model | Hierarchical delegation with autonomous sub-agents |
| Execution model | Event-driven (hooks) + on-demand (commands/agents) |
| State management | SQLite session persistence via hooks |
| Learning model | Continuous — sessions auto-extract patterns into skills |
| Deployment model | Local installation via `npm install` / `install.sh` |

---

## 3. Core Architectural Layers

ECC is organized into 7 layers, each with a distinct responsibility:

### Layer 1: Agent Layer (`agents/`)

Specialized AI sub-agents implemented as **Markdown files with YAML frontmatter**. Each agent has:

```yaml
---
name: <identifier>
description: <when to use this agent — governs auto-invocation>
tools: [<tool list>]
model: opus | sonnet | haiku
---
<system prompt body>
```

Agents are the **primary execution units**. They are invoked:
- **Proactively** — when the host AI decides the task matches the agent's description
- **Explicitly** — when a slash command invokes a specific agent
- **Automatically** — when a hook triggers agent activation

**25 agents** are currently defined. See `02-agents/` for full documentation.

### Layer 2: Skill Layer (`skills/`)

Reusable knowledge modules stored as `SKILL.md` files. Skills provide:

- Domain knowledge (e.g., `coding-standards`, `api-design`)
- Workflow patterns (e.g., `tdd-workflow`, `verification-loop`)
- Technology guides (e.g., `kotlin-patterns`, `django-tdd`)
- Business domain skills (e.g., `customs-trade-compliance`, `logistics-exception-management`)

Skills are **passive** — they inject context into the AI but do not execute code. They are activated via:
- Slash commands (`/skill-create`, `/tdd`)
- Explicit invocation in user prompts (`skill: tdd-workflow`)
- Automatic detection by the AI

**100+ skills** are defined across 3 categories: workflow, technology, and business domain.

### Layer 3: Hook Layer (`hooks/hooks.json` + `scripts/hooks/`)

Lifecycle automation using Claude Code's event system. Six lifecycle events are intercepted:

| Hook Event | Trigger | Examples |
|---|---|---|
| `SessionStart` | New conversation begins | Load prior session context, detect package manager |
| `PreToolUse` | Before any tool call | Security checks, compact suggestion, tmux reminder |
| `PostToolUse` | After any tool call | Auto-format, TypeScript check, quality gates |
| `PreCompact` | Before context compression | Save state to SQLite |
| `Stop` | After each AI response | Persist session, cost tracking, pattern extraction |
| `SessionEnd` | Conversation ends | Final marker, cleanup |

Hooks execute **Node.js scripts** from `scripts/hooks/`. They support:
- `async: true` — non-blocking execution
- `timeout: N` — maximum execution time
- Profile gating: `ECC_HOOK_PROFILE=minimal|standard|strict`

### Layer 4: Command Layer (`commands/`)

Slash commands are **Markdown files with a description frontmatter**:

```markdown
---
description: Run test-driven development workflow
---
<command instructions>
```

They are invoked by typing `/command-name` in the chat interface.
**52 commands** are available covering TDD, planning, E2E testing, code review, security scanning, session management, and more.

### Layer 5: Rules Layer (`rules/`)

Always-follow guidelines injected into the AI's permanent system context. Organized as:

```
rules/
├── common/          — Security, testing, patterns, git workflow (all languages)
├── typescript/      — TypeScript-specific rules
├── python/          — Python-specific rules
├── golang/          — Go-specific rules
├── kotlin/          — Kotlin-specific rules
├── swift/           — Swift-specific rules
├── php/             — PHP/Laravel rules
└── perl/            — Perl-specific rules
```

Rules are **passive** — they shape AI behavior without triggering execution.

### Layer 6: MCP Layer (`mcp-configs/mcp-servers.json`)

Configuration for 20+ Model Context Protocol servers:

| Server | Purpose |
|---|---|
| `github` | GitHub PRs, issues, repos |
| `supabase` | PostgreSQL database operations |
| `firecrawl` | Web scraping and crawling |
| `playwright` | Browser automation |
| `memory` | Cross-session persistent memory |
| `sequential-thinking` | Chain-of-thought reasoning |
| `exa-web-search` | Neural web search |
| `insaits` | AI-to-AI security monitoring |
| `fal-ai` | Image/video/audio generation |
| `context7` | Live library documentation |
| `vercel` | Deployment operations |
| `railway` | Railway deployments |
| `cloudflare` | Cloudflare Workers and docs |

### Layer 7: Script Layer (`scripts/`)

Cross-platform Node.js utilities:

- **Install system** — `install-plan.js`, `install-apply.js`
- **CLI entry point** — `ecc.js` (invoked as `npx ecc`)
- **Session management** — `session-manager.js`, `sessions-cli.js`
- **Orchestration** — `orchestrate-worktrees.js`, `orchestration-status.js`
- **Hook scripts** — Individual lifecycle automation scripts

---

## 4. Orchestration Logic

### Agent Delegation Model

The **host AI** (Claude Code, Cursor, etc.) acts as the **orchestrator**. It reads all agent descriptions and decides which agent to delegate to based on the user's task. This is a form of **prompt-based routing** — the agent's `description` frontmatter serves as a routing key.

### Invocation Patterns

**Sequential delegation** (most common):
```
User request → planner → tdd-guide → code-reviewer → security-reviewer
```

**Parallel review** (for code changes):
```
Code change → code-reviewer ┐
              security-reviewer ┤ (simultaneous)
              go-reviewer ┘
```

**Autonomous loops** (long-running tasks):
```
Task → loop-operator → [monitor] → [intervene if stalled] → harness-optimizer
```

**Build recovery**:
```
Build failure → build-error-resolver → tdd-guide → code-reviewer
```

### Continuous Learning Pipeline

ECC implements an **observer-pattern learning system**:

```
Tool use event → observe.sh hook → session-end.js → evaluate-session.js
                                                   → skill-create-output.js
                                                   → new SKILL.md
```

Each session's patterns are automatically evaluated and can be promoted to reusable skills.

---

## 5. Platform Capability Comparison

### ECC (Claude Code) vs GitHub Copilot Agent Mode

| Capability | ECC (Claude Code) | GitHub Copilot |
|---|---|---|
| Sub-agent delegation | 25 named agents | Implicit (no named sub-agents) |
| Lifecycle hooks | 6 events (PreTool, PostTool, Stop, etc.) | Limited (no PostToolUse, Stop) |
| Slash commands | 52 custom commands | Limited built-ins |
| Session persistence | SQLite across sessions | No cross-session memory |
| Continuous learning | Auto-extract skills from sessions | Not available |
| MCP integration | 20+ server configs | Limited MCP support |
| Cost tracking | Per-session token/cost metrics | Not available |
| Model routing | Per-agent model tier (opus/sonnet/haiku) | Single model |
| Custom skills | 100+ reusable knowledge modules | Not available |
| Multi-harness support | Claude Code, Cursor, OpenCode, Codex | GitHub.com + VS Code only |

### ECC vs IntelliJ AI Assistant

| Capability | ECC (Claude Code) | IntelliJ AI |
|---|---|---|
| Custom agents | 25 configurable agents | Predefined AI Actions |
| Lifecycle hooks | Event-driven automation | No hook system |
| Project-level rules | `CLAUDE.md` + `rules/` | No equivalent |
| Language coverage | 9 languages with dedicated agents | IDE-native language support |
| Test-driven workflow | TDD agent enforces red/green/refactor | Not enforced |
| Slash commands | 52 workflow commands | Built-in AI chat only |

### ECC vs CLI Agents (Codex CLI, etc.)

| Capability | ECC | CLI Agents |
|---|---|---|
| Agent definitions | Markdown with YAML frontmatter | `AGENTS.md` (Codex), `.agent` files |
| Hook system | JSON config with scripts | Limited/none |
| State persistence | SQLite | File-based |
| Cross-platform | Node.js scripts (Windows/Mac/Linux) | Platform-dependent |

---

## 6. Key Design Patterns

### Pattern 1: Description-Based Routing

Agents are auto-invoked based on their `description` field. The orchestrating AI reads all descriptions and matches user intent to agent purpose. This is a **declarative routing** approach — no code dispatches to agents.

### Pattern 2: Context Injection

Skills and rules are implemented as **context injection** — they add knowledge to the AI's working context without executing code. This makes them lightweight and harness-agnostic.

### Pattern 3: Minimal Tool Surface per Agent

Each agent declares only the tools it needs. Planning agents (`planner`, `architect`) use only `Read/Grep/Glob` — no write access. Implementation agents add `Write/Edit/Bash`. This is **least-privilege** design.

### Pattern 4: Async Non-Blocking Hooks

Performance-sensitive hooks (learning capture, cost tracking) run asynchronously with timeouts. This prevents hook overhead from slowing the primary AI interaction.

### Pattern 5: Profile-Based Hook Gating

The `ECC_HOOK_PROFILE=minimal|standard|strict` environment variable enables progressive hook activation. This allows the same configuration to work in low-overhead environments (CI, minimal) and full-featured environments (strict).

### Pattern 6: Continuous Learning

The system treats each session as a learning opportunity. Observations are captured, evaluated, and promoted to reusable skills automatically. This creates a **flywheel effect** where the system improves with use.

---

## 7. Identified Migration Targets

When adapting ECC to a platform-agnostic monorepo architecture, the following components are highest-value:

| Component | Migration Priority | Notes |
|---|---|---|
| Agent definitions | HIGH | Portable — just Markdown + YAML |
| Rules/guidelines | HIGH | Portable — inject into any AI context |
| Skill knowledge | HIGH | Portable — context injection pattern works everywhere |
| Hook system | MEDIUM | Requires platform support for lifecycle events |
| Slash commands | MEDIUM | Platform-specific invocation syntax |
| MCP configs | LOW | Platform-specific (Claude Code / Cursor only today) |
| SQLite persistence | LOW | Platform-specific |

---

## 8. Technical Metadata

| Attribute | Value |
|---|---|
| Runtime | Node.js ≥ 18 |
| Package name | `ecc-universal` |
| Install method | `npm install` / `install.sh` / `install.ps1` |
| Config entry point | `hooks/hooks.json` |
| Primary language | Markdown (agents/skills/commands) + Node.js (scripts) |
| Test count | 997 |
| License | MIT |
| Supported harnesses | Claude Code, Cursor, OpenCode, Codex |

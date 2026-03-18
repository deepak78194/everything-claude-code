# Agent Architecture — Everything Claude Code (ECC)

This directory documents all 25 agents in the ECC system in a **platform-agnostic format** that can be adapted for use with GitHub Copilot, IntelliJ AI Assistant, CLI agents, and other AI coding harnesses.

---

## Agent Inventory

| Agent | Model Tier | Category | File |
|---|---|---|---|
| [architect](./architect.md) | opus | Planning | Design and scalability decisions |
| [planner](./planner.md) | opus | Planning | Implementation plans and step breakdowns |
| [chief-of-staff](./chief-of-staff.md) | opus | Productivity | Multi-channel communication triage |
| [code-reviewer](./code-reviewer.md) | sonnet | Quality | Code quality, security, maintainability |
| [security-reviewer](./security-reviewer.md) | sonnet | Security | OWASP, secrets, input validation |
| [tdd-guide](./tdd-guide.md) | sonnet | Testing | Test-driven development enforcement |
| [e2e-runner](./e2e-runner.md) | sonnet | Testing | End-to-end test generation and execution |
| [build-error-resolver](./build-error-resolver.md) | sonnet | Build | Fix TypeScript/compilation errors |
| [refactor-cleaner](./refactor-cleaner.md) | sonnet | Quality | Dead code cleanup, deduplication |
| [doc-updater](./doc-updater.md) | haiku | Docs | Keep docs and codemaps current |
| [docs-lookup](./docs-lookup.md) | sonnet | Docs | Live library documentation via MCP |
| [database-reviewer](./database-reviewer.md) | sonnet | Data | PostgreSQL schema and query review |
| [language-reviewers](./language-reviewers.md) | sonnet | Quality | Go, Python, Java, Kotlin, Rust, C++ reviewers |
| [loop-operator](./loop-operator.md) | sonnet | Orchestration | Manage autonomous loops safely |
| [harness-optimizer](./harness-optimizer.md) | sonnet | Orchestration | Tune agent harness configuration |

---

## Model Tier Summary

| Tier | Agents | Use Case |
|---|---|---|
| `opus` | architect, planner, chief-of-staff | Complex reasoning, long-horizon planning |
| `sonnet` | All others | Standard implementation and review tasks |
| `haiku` | doc-updater | Fast, low-cost repetitive tasks |

---

## Orchestration Patterns

### Sequential Delegation (most common)

```
User Request
  → planner          (create detailed plan)
  → tdd-guide        (implement with tests)
  → code-reviewer    (review quality)
  → security-reviewer (review security)
```

### Parallel Review (code changes)

```
Code Change
  → code-reviewer      ─┐
  → security-reviewer  ─┤ (simultaneous)
  → go-reviewer        ─┘ (if Go code)
```

### Build Recovery Loop

```
Build Failure
  → build-error-resolver  (analyze and fix)
  → tdd-guide             (verify tests still pass)
  → code-reviewer         (review the fix)
```

### Autonomous Loop with Monitoring

```
Long-Running Task
  → loop-operator (start)
  → loop-operator (monitor checkpoints)
  → loop-operator (intervene on stall)
  → harness-optimizer (tune cost/throughput)
```

### Documentation Sync

```
Code Change
  → doc-updater   (update codemaps)
  → docs-lookup   (verify library references)
```

---

## Platform Adaptation Guide

### GitHub Copilot (GitHub.com + VS Code)

GitHub Copilot does not support named sub-agents natively. Adaptations:

1. **Agent instructions → Copilot custom instructions**: Convert agent system prompts to `github/copilot-instructions.md` in the repository
2. **Slash commands → `/` menu**: Register high-value commands as Copilot custom instructions or workspace shortcuts
3. **Hooks → GitHub Actions**: Migrate lifecycle hooks to GitHub Actions workflows (push, PR, review events)
4. **Rules → `.github/copilot-instructions.md`**: Inject always-follow rules into Copilot's project context

### IntelliJ AI Assistant

1. **Agent instructions → AI Actions**: Create custom AI Actions for each agent role
2. **Slash commands → Live templates**: Convert commands to IntelliJ Live Templates
3. **Rules → `.editorconfig` + AI context**: Inject rules into project-level AI settings
4. **Hooks → IDE event listeners**: Map lifecycle events to IntelliJ plugin events

### CLI Agents (Codex CLI, etc.)

ECC already supports Codex CLI natively via:
- `.codex/AGENTS.md` — agent instructions in Codex format
- `.codex/config.toml` — runtime configuration
- `scripts/sync-ecc-to-codex.sh` — sync ECC config to Codex format

### Monorepo Deployment

For a monorepo with multiple tech stacks:

1. **Root-level agents**: Place universal agents (planner, architect, code-reviewer, security-reviewer) at repo root
2. **Per-stack agents**: Place language-specific agents in stack subdirectories (e.g., `services/api/agents/java-reviewer.md`)
3. **Shared skills**: Keep skills at repo root — they are context injection, not execution
4. **Per-stack rules**: Place language-specific rules in stack subdirectories
5. **Shared hooks**: Use profile gating (`ECC_HOOK_PROFILE`) to select appropriate hooks per context

---

## Agent Definition Format

All agents use a consistent Markdown format with YAML frontmatter:

```markdown
---
name: <agent-name>
description: <when to use — serves as auto-invocation routing key>
tools: [<comma-separated tool list>]
model: opus | sonnet | haiku
---

# <Agent Title>

## Role
...

## Responsibilities
...

## Workflow
...

## Inputs / Outputs
...
```

### Tool Capabilities

| Tool | Purpose |
|---|---|
| `Read` | Read file contents |
| `Write` | Create new files |
| `Edit` | Modify existing files |
| `Bash` | Execute shell commands |
| `Grep` | Search file contents |
| `Glob` | Find files by pattern |
| `mcp__*` | MCP server operations |

---

## Migration from ECC Agents to GitHub Copilot

### Step 1: Export agent system prompts

Each `agents/<name>.md` file contains a system prompt after the YAML frontmatter. Extract the body and use it as a custom instruction or prompt template.

### Step 2: Map agent descriptions to triggers

The agent `description` field in ECC controls when agents are auto-invoked. Map these to:
- GitHub Copilot: `copilot-instructions.md` conditional sections
- IntelliJ: AI Action trigger conditions
- CLI: `AGENTS.md` agent sections

### Step 3: Migrate slash commands

High-value ECC slash commands can be converted to:
- Copilot workspace commands
- IntelliJ Live Templates
- CLI aliases

### Step 4: Migrate hooks to CI/CD

ECC lifecycle hooks handle automation that in a CI/CD-first workflow belongs in:
- GitHub Actions (push, PR, review, merge events)
- Pre-commit hooks (via `pre-commit` or `husky`)
- IDE plugins (format-on-save, typecheck-on-save)

### Step 5: Convert skills to documentation

ECC skills are knowledge modules injected into AI context. In platforms without skill injection:
- Add relevant skill content to `CLAUDE.md` / `copilot-instructions.md`
- Reference skills in PR templates
- Include skill patterns in code review checklists

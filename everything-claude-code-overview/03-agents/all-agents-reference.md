# All Agents Reference

## Complete Agent Inventory

ECC ships 18 specialized sub-agents. Each is defined in `agents/<name>.md` with YAML frontmatter specifying model tier and tools.

---

## Agent Summary Table

| Agent | Model | Tools | Purpose |
|---|---|---|---|
| `planner` | opus | Read, Grep, Glob | Create detailed implementation plans before coding |
| `architect` | opus | Read, Grep, Glob | System design, scalability, architectural decisions |
| `tdd-guide` | sonnet | Read, Write, Edit, Bash, Grep | Enforce test-first development (Red→Green→Refactor) |
| `code-reviewer` | sonnet | Read, Grep, Glob, Bash | Code quality, security, and maintainability review |
| `security-reviewer` | sonnet | Read, Write, Edit, Bash, Grep, Glob | OWASP Top 10, secrets detection, input validation |
| `build-error-resolver` | sonnet | Read, Write, Edit, Bash, Grep | Diagnose and fix build/compilation errors |
| `e2e-runner` | sonnet | Read, Write, Edit, Bash, Glob | Generate and execute Playwright E2E tests |
| `refactor-cleaner` | sonnet | Read, Write, Edit, Bash, Grep, Glob | Remove dead code, clean up technical debt |
| `doc-updater` | sonnet | Read, Write, Edit, Grep, Glob | Keep documentation and code maps in sync |
| `database-reviewer` | sonnet | Read, Grep, Glob | PostgreSQL/Supabase schema design, query optimization |
| `python-reviewer` | sonnet | Read, Grep, Glob, Bash | Python-specific code review |
| `go-reviewer` | sonnet | Read, Grep, Glob, Bash | Go-specific code review |
| `go-build-resolver` | sonnet | Read, Write, Edit, Bash, Grep | Fix Go build/compilation errors |
| `kotlin-reviewer` | sonnet | Read, Grep, Glob, Bash | Kotlin-specific code review |
| `kotlin-build-resolver` | sonnet | Read, Write, Edit, Bash, Grep | Fix Kotlin/Gradle build errors |
| `chief-of-staff` | sonnet | Read, Write, Bash | Communication triage: email, Slack, LINE, Messenger |
| `loop-operator` | sonnet | Read, Write, Edit, Bash, Grep | Manage autonomous loops safely, detect stalls |
| `harness-optimizer` | sonnet | Read, Grep, Glob | Tune harness config for reliability, cost, throughput |

---

## Agent Orchestration Patterns

### Pattern 1: Sequential Delegation

The most common pattern — one agent hands off to the next:

```
User Request
  → planner (create plan)
  → tdd-guide (implement with tests)
  → code-reviewer (review code)
  → security-reviewer (if security concerns)
```

### Pattern 2: Parallel Review

Multiple review agents activated simultaneously:

```
Code Change
  → code-reviewer    (parallel)
  → security-reviewer (parallel)
  → go-reviewer       (parallel, if Go code)
```

### Pattern 3: Build Recovery Loop

Triggered when build fails:

```
Build Failure
  → build-error-resolver (analyze + fix)
  → tdd-guide (verify tests still pass)
  → code-reviewer (review the fix)
```

### Pattern 4: Autonomous Loop

Long-running task with monitoring:

```
Long Task
  → loop-operator (start loop)
  → loop-operator (monitor progress)
  → loop-operator (detect stalls, intervene)
  → harness-optimizer (tune for cost/speed)
```

---

## Model Tier Selection

ECC uses model tiers strategically:

| Tier | Model | Used For |
|---|---|---|
| **Tier 1** | `opus` | Planning, architecture (complex reasoning, no execution) |
| **Tier 2** | `sonnet` | Reviews, implementation, testing (balanced) |
| **Tier 3** | `haiku` | Lightweight tasks, fast lookups |

The `harness-optimizer` agent can recommend model routing changes to reduce cost without losing quality.

---

## Agent Format

Every agent follows this format:

```markdown
---
name: agent-name
description: One-line description used for auto-invocation matching
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: opus|sonnet|haiku
---

[System prompt content — defines role, workflow, output format]
```

The `description` field is critical — it determines when the agent is **automatically invoked** by the AI harness based on task matching.

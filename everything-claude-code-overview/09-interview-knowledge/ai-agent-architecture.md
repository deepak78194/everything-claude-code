# AI Agent Architecture — Interview Knowledge Base

## Overview

This document synthesizes the architectural patterns, design decisions, and engineering lessons from the Everything Claude Code system. It is intended for developers studying AI agent architecture and preparing for technical discussions on the topic.

---

## 1. Agent Orchestration Models

### 1.1 Monolithic vs. Multi-Agent

**Monolithic approach:**
- Single AI handles all tasks
- Simple to set up
- Degrades as context grows
- No specialization

**Multi-agent approach (ECC pattern):**
- Each agent has a focused role
- Appropriate model tier per task
- Agents can be independently improved
- Context isolation between agents

**Key insight:** Agents should be scoped to complete one focused task. When an agent's description covers more than one domain, split it.

---

### 1.2 Implicit vs. Explicit Orchestration

**Explicit orchestration** (hard-coded routing):
```python
if task_type == "planning":
    call_planner_agent(task)
elif task_type == "review":
    call_reviewer_agent(task)
```

**Implicit orchestration (ECC pattern):**
```yaml
---
name: planner
description: Activate for feature implementation, architectural changes, or complex refactoring
---
```

The AI harness reads agent descriptions and activates agents automatically based on task understanding.

**Trade-offs:**
| | Explicit | Implicit |
|---|---|---|
| Predictability | High | Medium |
| Flexibility | Low | High |
| Maintenance | High | Low |
| Debugging | Easy | Harder |

---

### 1.3 Orchestration Topologies

**Pipeline (sequential):**
```
Input → Agent A → Agent B → Agent C → Output
```
Best for: workflows with clear phases (plan → implement → review)

**Fan-out/Fan-in (parallel):**
```
        ┌→ Agent A ─┐
Input → ┼→ Agent B ─┼→ Merge → Output
        └→ Agent C ─┘
```
Best for: independent reviews, parallel analysis

**Loop with circuit breaker:**
```
Input → Agent → Check → ──success──→ Output
                  │
                  └──failure (< N)──→ Agent (retry)
                  └──failure (≥ N)──→ Escalate
```
Best for: build error recovery, iterative refinement

---

## 2. Tool Calling Patterns

### 2.1 Principle of Least Privilege

Agents should only receive tools they need for their task:

```yaml
# Planning agent: read-only
tools: ["Read", "Grep", "Glob"]

# Implementation agent: full access
tools: ["Read", "Write", "Edit", "Bash", "Grep"]

# Review agent: read + run
tools: ["Read", "Bash", "Grep", "Glob"]
```

**Why this matters:**
- Reduces risk of unintended modifications
- Makes agent behavior more predictable
- Easier to audit and debug

### 2.2 Hook-Mediated Tool Use

Rather than building safety logic into every agent, ECC intercepts tool use at the framework level:

```
Agent → Tool Request → Hook (validate/augment) → Tool → Hook (analyze result) → Agent
```

Benefits:
- Safety logic is centralized
- Agents remain simple
- New safety checks added without modifying agents
- Same checks apply across all agents

### 2.3 Async Tool Side Effects

Non-blocking side effects run after tool completion:

```
Tool completes → Result returned to agent (immediately)
             → Background: format, type-check, observe, persist
```

This pattern achieves:
- Fast response time (agent isn't blocked on formatting)
- Eventual consistency for quality checks
- Non-blocking learning capture

---

## 3. Modular Agent Design

### 3.1 Agent as a Document

ECC encodes agents as Markdown documents with structured sections:
1. Frontmatter (metadata, tool config, model selection)
2. Role statement
3. Workflow steps
4. Output format template
5. Worked examples
6. Edge cases

This design is:
- **Human-readable**: Engineers can understand agent behavior without running it
- **Version-controllable**: Changes tracked in git with clear diffs
- **Testable**: CI validators check structure and content
- **Composable**: Agents can reference other agents by name

### 3.2 Skill vs. Agent Separation

| | Skill | Agent |
|---|---|---|
| Definition | `SKILL.md` | `<name>.md` + YAML |
| Has tools | No | Yes |
| Executes code | No | Yes |
| Purpose | Knowledge injection | Task execution |
| Reuse | By reference in context | Via delegation |
| State | None | None |

**When to use a Skill:** Domain knowledge, patterns, guidelines — things an agent needs to know but not execute.

**When to use an Agent:** Tasks requiring tool use, multi-step workflows, or specialized reasoning.

### 3.3 Model Tier Selection

Not all agents need the most powerful model:

| Tier | Model | Best For |
|---|---|---|
| Premium | Claude Opus | Complex planning, architectural reasoning, synthesis |
| Standard | Claude Sonnet | Reviews, implementation, balanced tasks |
| Fast | Claude Haiku | Simple lookups, classification, short tasks |

**Cost optimization principle:** Use the cheapest model that can reliably complete the task. ECC's `harness-optimizer` agent audits model assignments and suggests downgrades where appropriate.

---

## 4. Session Memory and Continuity

### 4.1 The Context Problem

LLM context windows have finite capacity. In multi-step agent workflows:
- Sub-agents start with no context about what the main agent has done
- Context fills up, forcing compaction (loss of detailed history)
- Sessions end, losing all state

### 4.2 ECC's Memory Architecture

```
SQLite State Store (persistent, cross-session)
  ├── session_summaries (what happened)
  ├── instincts (learned patterns)
  ├── cost_metrics (token/cost tracking)
  └── skill_observations (tool use patterns)

Context Window (ephemeral, per-session)
  ├── SessionStart injection (loads from state store)
  ├── Active skills (injected by commands)
  └── Current agent context

Pre-compaction save (preserves state before context reset)
Post-response save (persists every turn)
```

### 4.3 Instinct System

Instincts are patterns extracted from sessions and stored for future context enrichment:

1. **Capture**: Every tool use generates an observation (async, non-blocking)
2. **Evaluate**: At session end, transcript is analyzed for extractable patterns
3. **Store**: Worthy patterns stored as instincts in state store
4. **Apply**: Next session's `SessionStart` loads and injects relevant instincts

This implements a form of **episodic memory** for AI agents — learning from past experiences to improve future behavior.

---

## 5. Advantages and Limitations

### Advantages

| Advantage | Description |
|---|---|
| **Specialization** | Each agent optimized for one domain, using appropriate model tier |
| **Composability** | Agents combine to handle complex workflows |
| **Modularity** | Individual agents can be updated without affecting others |
| **Cost efficiency** | Use cheaper models for simple tasks, premium only where needed |
| **Observability** | Hook system provides full visibility into tool use |
| **Continuity** | State store enables memory across sessions |
| **Cross-harness** | Same concepts work across Claude Code, Cursor, OpenCode, Codex |

### Limitations

| Limitation | Description |
|---|---|
| **Context overhead** | Each agent delegation consumes context tokens for handoff |
| **Implicit activation uncertainty** | Auto-activation based on description matching can be unpredictable |
| **Eventual consistency** | Async hooks mean quality feedback arrives one turn late |
| **State store coupling** | Hook scripts depend on state store schema; schema changes require migration |
| **Harness dependence** | Hook system works only on supported AI harnesses |
| **Learning lag** | Instincts only available in the next session, not the current one |

---

## 6. Key Interview Topics

### Q: How does ECC prevent agents from going off-task?

**Answer:** Multiple mechanisms: (1) Each agent has precisely scoped tool access (planner gets only read tools). (2) The `loop-operator` agent monitors long-running loops for stalls and drift. (3) Circuit breakers in build-error agents prevent infinite retry loops. (4) The harness-optimizer audits agent configurations for scope creep.

### Q: How does ECC handle the context window limit?

**Answer:** Via a multi-layer approach: (1) The `suggest-compact` hook monitors edit frequency and suggests manual compaction at logical intervals. (2) The `pre-compact` hook saves state to SQLite before compaction. (3) The `session-start` hook injects only a compact summary, not full history. (4) The `strategic-compact` skill teaches optimal compaction timing to the AI.

### Q: How is ECC different from LangChain or AutoGen?

**Answer:** ECC is a plugin for AI coding assistants, not a standalone orchestration framework. It delegates orchestration to the host AI harness (Claude Code) rather than hard-coding routing logic. Agents are Markdown documents, not Python classes. The system is hook-driven rather than programmatic. ECC's primary value is the curated content (agents, skills, rules) — the framework is minimal.

### Q: What is the continuous learning mechanism?

**Answer:** An observer-pattern pipeline: `observe.sh` (async hook) captures every tool use as an observation. At Stop events, `evaluate-session.js` analyzes the transcript for extractable patterns. Worthy patterns are stored as "instincts" in SQLite. The next session's `SessionStart` loads these instincts and injects them into context. This enables the AI to remember what worked in past sessions.

### Q: How does ECC achieve cross-harness compatibility?

**Answer:** By maintaining separate configuration trees for each harness (`.cursor/`, `.opencode/`, `.codex/`) with format-appropriate adaptations. Claude Code uses Markdown + YAML. OpenCode uses plain-text `.txt` prompts. Codex uses TOML. Cursor uses JavaScript hook adapters. The semantic content is identical; only the format changes.

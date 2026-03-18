# Prompt Engineering in ECC

## Overview

ECC uses a highly structured approach to prompt engineering for its agent system. Rather than monolithic system prompts, each agent is a focused Markdown document with a YAML frontmatter header and a carefully layered prompt body.

---

## Agent Prompt Architecture

### Frontmatter as Metadata

```yaml
---
name: planner
description: Expert planning specialist for complex features and refactoring. Use PROACTIVELY when users request feature implementation, architectural changes, or complex refactoring. Automatically activated for planning tasks.
tools: ["Read", "Grep", "Glob"]
model: opus
---
```

The `description` field serves a dual purpose:
1. **Auto-invocation trigger** — Claude Code reads this description to decide when to activate the agent automatically
2. **Human documentation** — explains when to use the agent

This is an example of **declarative activation** — the agent declares when it should run, rather than the orchestrator hard-coding activation rules.

### Prompt Structure Pattern

Every ECC agent prompt follows this layered structure:

```
1. Role Statement ("You are a [role] specializing in [domain]")
2. Core Responsibilities (bullet list)
3. Workflow Steps (numbered, detailed)
4. Output Format (template with example)
5. Best Practices (dos and don'ts)
6. Worked Example (complete, realistic)
7. Edge Cases / Red Flags
```

This pattern ensures:
- **Clarity**: The agent knows its exact role
- **Repeatability**: The workflow steps produce consistent outputs
- **Calibration**: Worked examples set the expected quality bar
- **Guardrails**: Red flags prevent common mistakes

---

## Confidence-Based Filtering (Code Reviewer)

The code reviewer agent implements a sophisticated filtering pattern:

```
Report if you are >80% confident it is a real issue.
Skip stylistic preferences unless they violate project conventions.
Skip issues in unchanged code unless CRITICAL security issues.
Consolidate similar issues (e.g., "5 functions missing error handling" not 5 separate findings).
```

This is **prompt-level noise filtering** — it reduces cognitive load by preventing the AI from flagging marginal issues, while ensuring critical problems are always surfaced.

---

## Severity Taxonomy

ECC uses a consistent four-level severity taxonomy across all review agents:

| Level | Definition | Action Required |
|---|---|---|
| CRITICAL | Can cause real damage (data loss, security breach, system failure) | Block merge, fix immediately |
| HIGH | Significant quality or correctness issue | Warn, fix before merge |
| MEDIUM | Performance or maintainability concern | Note, fix soon |
| LOW | Style or documentation issue | Note, fix when convenient |

This taxonomy appears in: `code-reviewer`, `security-reviewer`, `database-reviewer`, `go-reviewer`, `python-reviewer`, `kotlin-reviewer`.

---

## System Prompt Layering

ECC uses a three-level system prompt layering:

```
Layer 1: Rules (always-follow guidelines)
  ↓ Applied at session start
Layer 2: Skills (domain knowledge)
  ↓ Injected per task
Layer 3: Agent Prompts (task-specific behavior)
  ↓ Activated per sub-task
```

Each layer augments the previous:
- Rules set baseline behavior (immutability, error handling, security)
- Skills add domain-specific knowledge (TDD cycle, API design patterns)
- Agent prompts define precise task execution behavior

---

## Worked Example as Ground Truth

The planner agent includes a complete worked example (Stripe subscription billing) that:
- Shows exactly what level of detail is expected
- Demonstrates proper phase decomposition
- Illustrates risk assessment format
- Sets quality expectations without explicit instructions

This is **example-driven prompting** — rather than explaining abstractly "be detailed", the prompt shows what "detailed" looks like.

---

## Cross-Harness Prompt Translation

The same logical agent prompt exists in multiple formats:

| Harness | Format | Location |
|---|---|---|
| Claude Code | Markdown + YAML frontmatter | `agents/planner.md` |
| OpenCode | Plain text prompt | `.opencode/prompts/agents/planner.txt` |
| Codex | TOML with instructions | `.codex/agents/` |

The OpenCode `.txt` format strips YAML and Markdown syntax for harnesses that expect plain text prompts. The semantic content is identical.

---

## Cost-Aware Prompt Design

In v1.8, ECC introduced cost awareness into agent prompts:

```
Cost-awareness check:
- Flag workflows that escalate to higher-cost models without clear reasoning need.
- Recommend defaulting to lower-cost tiers for deterministic refactors.
```

This is reflected in the model tier selection:
- `opus` only for reasoning-heavy tasks (planning, architecture)
- `sonnet` for execution tasks (reviews, implementation)
- `haiku` for lightweight lookups

The `harness-optimizer` agent can audit an entire harness configuration and recommend model routing changes to reduce cost without quality loss.

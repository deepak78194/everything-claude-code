# Agent Orchestration Patterns in ECC

## Overview

ECC implements several agent orchestration patterns that enable complex multi-agent workflows. Unlike heavyweight orchestration frameworks, ECC's patterns are lightweight — they rely on the AI harness's built-in agent delegation mechanism and structured prompts that define interaction protocols.

---

## Pattern 1: Sequential Delegation

The most common pattern. Agents are activated one after another, each producing output that feeds the next.

```
User Request
  → planner (analyze + plan)
  → tdd-guide (implement with tests)
  → code-reviewer (review)
  → security-reviewer (if needed)
```

**Implementation:** Each agent's `description` field in its YAML frontmatter signals to the AI harness when to activate it. The AI naturally sequences agents by recognizing that planning precedes implementation, and review follows implementation.

**Key insight:** This is **implicit orchestration** — there is no explicit orchestration code. The AI itself decides to sequence agents based on task understanding and agent descriptions.

---

## Pattern 2: Parallel Review

Multiple review agents activate simultaneously for independent analysis.

```
Code Change
  ├── code-reviewer (quality + correctness)
  ├── security-reviewer (vulnerabilities)
  └── language-reviewer (language-specific issues)
```

**Implementation:** The AI can activate multiple agents in parallel by calling them as sub-tasks. Each agent reads the same git diff independently and produces its own report.

**Benefit:** Parallel reviews catch more issues than sequential reviews (no anchoring bias between reviewers) and complete faster.

---

## Pattern 3: Build Recovery Loop

An autonomous recovery pattern triggered by build or test failures.

```
Build Failure Detected
  → build-error-resolver activated
    → Read error output
    → Diagnose root cause
    → Apply fix (Edit/Write)
    → Re-run build (Bash)
    → If still failing: iterate (max 3 attempts)
    → If fixed: hand off to code-reviewer
```

**Implementation:** `build-error-resolver` and `go-build-resolver` implement this pattern internally. The agent's system prompt includes explicit iteration logic: attempt fix → verify → iterate → escalate.

**Circuit breaker:** The agent escalates (reports to user) after 3 failed fix attempts rather than looping indefinitely.

---

## Pattern 4: Autonomous Loop

Long-running agent execution with stall detection and intervention.

```
/loop-start command
  → loop-operator monitors execution
    → Agent runs task continuously
    → loop-operator checks for stalls (no progress)
    → loop-operator checks for drift (off-task behavior)
    → Intervention if stalled or drifted
    → /loop-status reports progress
```

**Implementation:** The `loop-operator` agent implements the monitoring pattern. It uses a state store to track progress metrics and detect when an agent loop has stalled.

**Stall definition:** No meaningful tool use (Edit/Write/Bash with real output) for N consecutive turns.

---

## Pattern 5: Harness Optimization Loop

Meta-level optimization of the ECC configuration itself.

```
/harness-audit command
  → harness-optimizer activated
    → Read current hook configuration
    → Read agent model assignments
    → Analyze recent cost metrics from state store
    → Identify optimization opportunities
    → Report recommendations
```

**Implementation:** The `harness-optimizer` agent has read access to `hooks/hooks.json`, agent frontmatter, and the SQLite state store (via read-only queries). It produces a structured audit report with ranked recommendations.

---

## Pattern 6: Observer-Pattern Learning

Passive background monitoring with deferred learning.

```
Every Tool Use
  → observe.sh fires (async, non-blocking)
    → Captures tool type, input, output summary
    → Stores in observation log

Every Stop Event
  → evaluate-session.js fires (async)
    → Reads session transcript
    → Applies heuristics to find extractable patterns
    → If worthy: stores as instinct in state store

Next SessionStart
  → session-start.js loads instincts
  → Injects learned patterns into context
```

**Implementation:** This is a fully decoupled observer pattern. The observation capture (`observe.sh`) has no dependency on the evaluation logic (`evaluate-session.js`). The learning pipeline is eventually consistent — instincts become available on the next session, not the current one.

---

## Pattern 7: Context Handoff

Session continuity across context resets.

```
Context Nearing Limit
  → suggest-compact.js suggests /checkpoint

/checkpoint command
  → Save session state to state store
  → Compact context (drop detailed history)
  → Inject compact summary into fresh context

Next Session
  → session-start.js reads last checkpoint
  → Injects: last task, decisions, open items, git state
```

**Implementation:** The state store acts as a persistent memory layer. The `pre-compact.js` hook saves state before Claude Code's automatic compaction. The `session-start.js` hook restores state at the start of new sessions.

---

## The Context Problem

ECC addresses what it calls "The Context Problem" — the fundamental challenge that sub-agents start with empty context and must re-discover what the main agent already knows.

**Solution approaches implemented in ECC:**

1. **Structured handoff**: The main agent passes a concise brief to sub-agents (file paths, what was tried, what succeeded)
2. **Context injection via SessionStart**: Load the most relevant state at session start to minimize re-discovery
3. **Skill injection**: Instead of re-explaining patterns, inject a skill that encodes the knowledge
4. **State store**: Persist critical decisions so any agent can retrieve them

---

## Multi-Instance Orchestration

For large parallel workloads, ECC provides:

### Git Worktree Orchestration (`/orchestrate`, `scripts/orchestrate-worktrees.js`)

```
/orchestrate 4 tasks
  → Create 4 git worktrees (isolated working directories)
  → Launch 4 Claude Code instances in tmux panes
  → Each instance works on one task independently
  → orchestration-status.js monitors all instances
  → Results merged back to main branch
```

### Codex Worker Orchestration (`scripts/orchestrate-codex-worker.sh`)

Similar pattern for Codex CLI instances running in parallel tmux sessions.

**Cascade Method** (documented in ECC guides):
1. Start with one instance
2. When it completes Phase 1, start Phase 2 with output as input
3. Multiple cascaded instances process sequentially complex, large tasks

---

## Orchestration Anti-Patterns (Documented in ECC)

ECC explicitly documents patterns to avoid:

| Anti-Pattern | Problem | Better Approach |
|---|---|---|
| "Mega-agent" | One agent doing everything → context exhaustion | Delegate to specialized agents |
| Chatty agents | Sub-agents producing verbose output → token waste | Structured, compact output formats |
| Re-discovery loops | Sub-agents re-reading all code each time | Pass targeted context in handoffs |
| Uncircuited retry loops | Agents retrying indefinitely | Circuit breakers (max 3 attempts) |
| Parallel without isolation | Multiple agents editing same files | Git worktrees for true isolation |

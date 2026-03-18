# Agent: loop-operator

**Category:** Orchestration  
**Model Tier:** `sonnet`  
**Source:** `agents/loop-operator.md`

---

## Purpose

The loop-operator agent **runs and monitors autonomous agent loops** safely. It manages long-running tasks that require repeated execution, tracks progress through checkpoints, detects stalls and retry storms, and intervenes when loops get stuck.

---

## When to Use

Invoke this agent:
- When starting a long-running automated task (file processing, bulk operations)
- When monitoring a running loop's progress
- When a loop appears to have stalled
- When implementing autonomous multi-step workflows
- When needing safe stop conditions for iterative processes

---

## Responsibilities

1. **Loop initiation** — Start loops from explicit patterns with defined stop conditions
2. **Progress tracking** — Monitor checkpoint completion
3. **Stall detection** — Identify loops that stop making progress
4. **Retry storm prevention** — Detect and pause infinite retry loops
5. **Safe intervention** — Reduce scope and recover from failures
6. **Observability** — Report loop status clearly

---

## Inputs

- Loop pattern definition (what to repeat, stop conditions)
- Loop mode (finite iterations, time-bounded, event-driven)
- Current loop state (when monitoring)

## Outputs

- Loop execution with checkpoint reporting
- Stall alerts with intervention recommendations
- Progress summaries
- Loop completion or graceful halt report

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read loop state and checkpoint files |
| `Grep` | Search for loop patterns and state |
| `Glob` | Find files to process in a loop |
| `Bash` | Execute loop steps and check progress |
| `Edit` | Update checkpoint state |

---

## Loop Execution Pattern

```
1. Define loop:
   - Pattern: what to iterate over
   - Mode: count-bounded / time-bounded / event-driven
   - Stop conditions: success criteria OR failure threshold
   - Checkpoint interval: how often to save progress

2. Execute:
   LOOP:
     a. Execute one iteration
     b. Validate result
     c. Save checkpoint
     d. Check stop conditions
     e. Sleep if rate-limiting needed
   UNTIL: stop condition met OR failure threshold exceeded

3. Report:
   - Completed iterations
   - Success/failure breakdown
   - Any interventions taken
```

---

## Stall Detection Heuristics

A loop is considered **stalled** when:

| Condition | Threshold |
|---|---|
| No checkpoint progress | Last N iterations produced same state |
| Repeated identical errors | Same error message 3+ consecutive times |
| Execution time explosion | Iteration time > 10x baseline |
| Queue not draining | Backlog growing instead of shrinking |

**Intervention actions:**
1. **Pause loop** — Save state, wait for human review
2. **Reduce scope** — Process smaller batch, continue
3. **Skip and log** — Skip problematic item, continue with rest
4. **Halt and report** — Stop entirely, provide diagnosis

---

## Safe Stop Conditions

Always define explicit stop conditions before starting any loop:

```markdown
Stop when:
- [ ] All items processed (success)
- [ ] 3 consecutive failures (error threshold)
- [ ] 30 minutes elapsed (time limit)
- [ ] Rate limit hit (backoff)
- [ ] User interruption signal received
```

---

## Checkpoint Format

```json
{
  "loop_id": "bulk-migration-2026-03",
  "started_at": "2026-03-17T04:00:00Z",
  "total_items": 1500,
  "processed": 450,
  "succeeded": 448,
  "failed": 2,
  "last_checkpoint": "2026-03-17T04:15:00Z",
  "failed_items": ["item-id-123", "item-id-456"],
  "status": "running"
}
```

---

## Platform Adaptation

### GitHub Copilot / GitHub Actions
For CI/CD loops (e.g., batch processing), use GitHub Actions matrix jobs with explicit retry limits. The loop-operator pattern maps to:
- Matrix strategy for parallelism
- `continue-on-error: false` for fail-fast
- Artifact upload for checkpoint state

### CLI Agents
Works natively in CLI context where bash loops are common. Add explicit checkpoint files and timeout mechanisms.

### Monorepo Usage
Use for cross-service operations (e.g., updating all services to a new dependency version). Define the scope (which services), checkpoint per service, and stop condition (all services or failure threshold).

---

## Dependencies

- Often paired with: **harness-optimizer** (optimize harness during loop execution)
- May trigger: **build-error-resolver** (if loop fails due to build issues)
- Consults: **planner** (plan the loop strategy before starting)

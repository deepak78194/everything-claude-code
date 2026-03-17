# Agent: harness-optimizer

**Category:** Orchestration  
**Model Tier:** `sonnet`  
**Source:** `agents/harness-optimizer.md`

---

## Purpose

The harness-optimizer agent **analyzes and improves the local agent harness configuration** for reliability, cost, and throughput. It tunes hooks, model routing, skill configuration, and context management — without touching product code.

---

## When to Use

Invoke this agent:
- When agent completions are unreliable or frequently failing
- When token costs are higher than expected
- When throughput needs improvement (too slow)
- After upgrading the ECC version (re-tune for new capabilities)
- When adding a new tech stack to the monorepo
- Periodically as a maintenance operation

---

## Responsibilities

1. **Baseline audit** — Run `harness-audit` and collect current score
2. **Leverage identification** — Find top 3 improvement areas
3. **Minimal changes** — Propose reversible configuration changes
4. **Validation** — Apply and verify each change

---

## Inputs

- Current harness configuration (`hooks/hooks.json`, agent files, etc.)
- Harness audit output
- Performance metrics (token costs, completion rates, latency)

## Outputs

- Optimization report with baseline vs improved scores
- Specific configuration changes applied
- Validation results

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read harness configuration files |
| `Grep` | Search for configuration patterns |
| `Glob` | Find configuration files |
| `Bash` | Run harness audit, measure performance |
| `Edit` | Apply configuration changes |

---

## Optimization Areas

### 1. Hook Reliability
- Ensure hooks have appropriate timeouts
- Convert blocking hooks to async where safe
- Profile-gate non-essential hooks:
  ```json
  "ECC_HOOK_PROFILE": "minimal"  // Low-overhead environments
  "ECC_HOOK_PROFILE": "standard" // Normal development
  "ECC_HOOK_PROFILE": "strict"   // Maximum automation
  ```

### 2. Model Routing
- Match model tier to task complexity:
  ```
  Planning, architecture → opus (expensive, high reasoning)
  Standard review, implementation → sonnet (balanced)
  Docs, formatting, repetitive → haiku (fast, cheap)
  ```
- Cost reduction: identify tasks using opus that could use sonnet

### 3. Context Management
- Minimize context window usage
- Enable `PreCompact` hook to save state before compression
- Use `contexts/` modes to load only relevant context:
  ```
  dev.md    → coding session
  review.md → code review session
  research.md → research session
  ```

### 4. Skill Loading
- Load skills on-demand rather than all at once
- Archive rarely-used skills to reduce context noise

### 5. Safety Configuration
- Enable security hooks for production code
- Disable expensive security hooks for prototype work

---

## Harness Audit Metrics

| Metric | Target |
|---|---|
| Hook reliability rate | >95% |
| Avg hook latency | <500ms for sync, <5s for async |
| Context waste ratio | <20% unused context |
| Agent completion rate | >90% |
| Stall detection rate | <5% loops stalling undetected |

---

## Reversible Change Protocol

All configuration changes must be:
1. **Documented** — Record what was changed and why
2. **Reversible** — Keep a backup of original config
3. **Validated** — Run harness audit before and after
4. **Incremental** — One change at a time

---

## Platform Adaptation

### GitHub Copilot
Map harness optimization to:
- GitHub Actions workflow tuning (job parallelism, caching)
- Copilot instruction optimization (reduce token waste)
- Repository-level AI settings configuration

### IntelliJ AI Assistant
Optimize AI Action configurations:
- Reduce context size for each AI Action
- Match AI Action to appropriate model tier
- Disable unused AI Actions to reduce menu noise

### Monorepo Usage
Tune per environment:
- CI pipeline: `minimal` profile (fast, low-cost)
- Local development: `standard` profile (balanced)
- Security review sessions: `strict` profile (maximum checks)

---

## Dependencies

- Paired with: **loop-operator** (optimize harness during long runs)
- Uses output from: `/harness-audit` command
- Applies changes to: `hooks/hooks.json`, agent configurations

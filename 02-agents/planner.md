# Agent: planner

**Category:** Planning  
**Model Tier:** `opus` (highest reasoning capability)  
**Source:** `agents/planner.md`

---

## Purpose

The planner agent creates **comprehensive, actionable implementation plans** before any code is written. It breaks complex features into clearly ordered steps with explicit file paths, dependencies, risks, and success criteria.

---

## When to Use

Invoke this agent:
- When starting a non-trivial feature implementation
- Before refactoring a large module or system
- When estimating scope of a complex task
- When multiple engineers need to work in parallel
- When identifying dependencies and risks before committing to a timeline

---

## Responsibilities

1. **Requirements analysis** — Fully understand the request before planning
2. **Architecture review** — Identify affected components in the existing codebase
3. **Step decomposition** — Break work into granular, independently verifiable steps
4. **Dependency ordering** — Sequence steps to enable incremental testing and delivery
5. **Risk identification** — Flag high-risk steps and propose mitigations
6. **Phase structuring** — Organize steps into independently deliverable phases

---

## Inputs

- User request describing the feature or change
- Access to existing codebase (Read/Grep/Glob)
- Constraints: deadlines, tech stack, team size

## Outputs

A structured Markdown plan with:
- Overview summary
- Requirements list
- Architecture changes
- Implementation steps (grouped into phases)
- Testing strategy
- Risks and mitigations
- Success criteria checklist

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read existing code to understand context |
| `Grep` | Search for related patterns and usage |
| `Glob` | Discover file structure and existing files |

> **No write access** — the planner produces plans only, does not implement.

---

## Plan Output Format

```markdown
# Implementation Plan: <Feature Name>

## Overview
<2-3 sentence summary>

## Requirements
- <requirement 1>
- <requirement 2>

## Architecture Changes
- <path/to/file.ts>: <description of change>

## Implementation Steps

### Phase 1: <Phase Name>
1. **<Step Name>** (File: path/to/file.ts)
   - Action: <specific action>
   - Why: <rationale>
   - Dependencies: None | Requires step N
   - Risk: Low | Medium | High

### Phase 2: <Phase Name>
...

## Testing Strategy
- Unit tests: <what to test>
- Integration tests: <what to test>
- E2E tests: <user journeys>

## Risks & Mitigations
- **Risk**: <description>
  - Mitigation: <how to address>

## Success Criteria
- [ ] <criterion 1>
- [ ] <criterion 2>
```

---

## Planning Quality Standards

Each step must include:
- **Exact file path** (no vague references like "update the service")
- **Specific action** (not "update" but "add `validateInput()` function")
- **Rationale** (why this step is needed)
- **Risk level** (Low/Medium/High with justification for High)

Plans must:
- Be **incrementally deliverable** — each phase can be merged independently
- Have a **testing strategy** at every phase
- Include **backwards-compatible** changes where possible

---

## Phase Sizing Guidance

| Phase | Scope |
|---|---|
| Phase 1 | Minimum viable — smallest slice providing value |
| Phase 2 | Core experience — complete happy path |
| Phase 3 | Edge cases — error handling, validation, polish |
| Phase 4 | Optimization — performance, monitoring, analytics |

---

## Platform Adaptation

### GitHub Copilot
Use in planning sessions before opening a PR. The plan output becomes the PR description body, enabling clear reviewability.

### IntelliJ AI Assistant
Create an "Implementation Plan" AI Action that wraps the planner system prompt. Run before any significant feature branch.

### CLI Agents
Add to `AGENTS.md` as an explicit planning phase before implementation.

### Monorepo Usage
Works across all services. For monorepo planning, include the affected services explicitly in the request: "Plan implementation of X across services/auth and services/api."

---

## Dependencies

- Often preceded by: **architect** (architecture decision before planning)
- Followed by: **tdd-guide** (implement the plan with tests)
- Consults: **security-reviewer** (review security implications of plan)

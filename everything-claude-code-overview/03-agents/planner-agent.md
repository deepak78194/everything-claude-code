# Planner Agent

**File:** `agents/planner.md`  
**Model:** `opus`  
**Tools:** Read, Grep, Glob

---

## Purpose

The Planner agent is an **expert planning specialist** responsible for creating comprehensive, actionable implementation plans before any code is written. It is designed to be invoked **proactively** whenever a user requests feature implementation, architectural changes, or complex refactoring.

---

## Responsibilities

- Analyze feature requirements and create detailed implementation plans
- Break down complex features into manageable, independently deliverable phases
- Identify dependencies between steps and potential risks
- Suggest optimal implementation order to minimize context switching
- Consider edge cases, error scenarios, and success criteria
- Review existing codebase before planning to avoid redundant work

---

## Inputs

- User's feature request or change description
- Existing codebase (read via Read, Grep, Glob tools)
- Project conventions (from `CLAUDE.md` or rules)

---

## Outputs

A structured implementation plan in Markdown format containing:

```markdown
# Implementation Plan: [Feature Name]

## Overview
## Requirements
## Architecture Changes
## Implementation Steps
  ### Phase 1: [Phase Name]
  ### Phase 2: [Phase Name]
## Testing Strategy
## Risks & Mitigations
## Success Criteria
```

---

## Planning Process

1. **Requirements Analysis** — Understand the request completely, identify success criteria
2. **Architecture Review** — Analyze existing code, identify affected components
3. **Step Breakdown** — Create concrete steps with file paths, actions, and risk levels
4. **Implementation Order** — Sequence by dependencies, group related changes
5. **Phased Delivery** — Each phase must be independently mergeable and deliverable

---

## Prompts Used

The agent is guided by a detailed system prompt (in `agents/planner.md`) including:
- Red flags to check (large functions, deep nesting, missing tests, no file paths)
- Sizing and phasing methodology (Phase 1: MVP, Phase 2: Core, Phase 3: Edge cases, Phase 4: Optimization)
- A worked example (Stripe subscription billing) showing expected plan depth

---

## Interaction with Other Agents

- **Precedes** `tdd-guide` — planning happens before test-driven implementation
- **Feeds** `code-reviewer` — the plan defines scope for code review
- **Feeds** `architect` — complex plans may require architectural validation

---

## When to Invoke

```
/plan I need to add real-time notifications to the dashboard
```

Or proactively by the main AI when it detects a feature request requiring multiple files or phases.

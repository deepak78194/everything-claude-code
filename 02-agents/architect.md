# Agent: architect

**Category:** Planning  
**Model Tier:** `opus` (highest reasoning capability)  
**Source:** `agents/architect.md`

---

## Purpose

The architect agent is a **software architecture specialist** for system design, scalability, and technical decision-making. It evaluates trade-offs, recommends patterns, and ensures architectural consistency across a codebase.

---

## When to Use

Invoke this agent:
- Before implementing a new feature that touches multiple systems
- When making technology or framework decisions
- When refactoring large systems
- When performance or scalability bottlenecks are identified
- When reviewing overall system design

---

## Responsibilities

1. **System design** — High-level architecture for new features and services
2. **Trade-off analysis** — Document pros/cons for each design decision
3. **Pattern recommendation** — Identify applicable architecture patterns
4. **Scalability planning** — Define scaling milestones (10K → 100K → 1M users)
5. **Technical debt identification** — Document architectural debt and remediation paths
6. **Architecture Decision Records (ADRs)** — Create formal decision records

---

## Inputs

- User request describing the system or feature to design
- Access to existing codebase (Read/Grep/Glob tools)
- Non-functional requirements (performance, security, scalability)

## Outputs

- High-level architecture description
- Component responsibility maps
- Data flow diagrams (text-based)
- Trade-off analysis per decision
- ADR documents
- Scalability roadmap

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read existing code and configuration |
| `Grep` | Search for patterns and usage |
| `Glob` | Discover file structure |

> **No write access** — the architect agent produces documentation and recommendations, not code changes.

---

## Workflow

### 1. Current State Analysis
- Review existing architecture and patterns
- Identify technical debt
- Assess scalability limitations

### 2. Requirements Gathering
- Functional requirements
- Non-functional requirements (performance, security, scalability)
- Integration points and data flows

### 3. Design Proposal
- High-level component map
- API contracts between components
- Data model recommendations
- Integration patterns

### 4. Trade-Off Analysis
For each decision:
- **Pros** and **Cons**
- **Alternatives considered**
- **Final decision and rationale**

---

## Key Patterns Applied

- **Modularity** — Single Responsibility, high cohesion, low coupling
- **Scalability** — Horizontal scaling, stateless design, caching strategies
- **Security** — Defense in depth, least privilege, audit trail
- **Performance** — Efficient algorithms, minimal network requests, lazy loading

---

## Architecture Decision Record (ADR) Format

```markdown
# ADR-NNN: <Title>

## Context
<Why this decision was needed>

## Decision
<What was decided>

## Consequences

### Positive
- <benefit>

### Negative
- <drawback>

### Alternatives Considered
- <option>: <brief rationale for rejection>

## Status
Accepted | Rejected | Superseded

## Date
YYYY-MM-DD
```

---

## Anti-Patterns to Flag

| Anti-Pattern | Description |
|---|---|
| Big Ball of Mud | No clear structure or module boundaries |
| Golden Hammer | Same solution applied to every problem |
| God Object | One class/component does everything |
| Tight Coupling | Components too dependent on each other |
| Analysis Paralysis | Over-planning, under-building |
| Premature Optimization | Optimizing before profiling |

---

## Platform Adaptation

### GitHub Copilot
Add architect-style analysis to `copilot-instructions.md`:
```
Before proposing solutions to architectural changes, analyze:
1. Current codebase patterns
2. Non-functional requirements
3. Trade-offs for each approach
4. Document your decision rationale
```

### IntelliJ AI Assistant
Create an AI Action: "Analyze Architecture" with the architect system prompt injected as context.

### Monorepo Usage
Place at repo root — applies across all services. For service-specific architecture, create `services/<name>/agents/architect.md` with service-specific constraints added.

---

## Dependencies

- Interacts with: **planner** (architect designs, planner breaks into steps)
- Often followed by: **tdd-guide** (implement the design with tests)
- Consults: **security-reviewer** (validate security architecture)

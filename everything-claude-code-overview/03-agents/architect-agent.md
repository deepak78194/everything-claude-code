# Architect Agent

**File:** `agents/architect.md`  
**Model:** `opus`  
**Tools:** Read, Grep, Glob

---

## Purpose

The Architect agent is a **software architecture specialist** for system design, scalability, and technical decision-making. It is used proactively when planning new features, refactoring large systems, or making architectural decisions.

---

## Responsibilities

- Design system architecture for new features
- Evaluate technical trade-offs
- Recommend patterns and best practices
- Identify scalability bottlenecks
- Plan for future growth
- Ensure consistency across the codebase

---

## Inputs

- Feature or system change request
- Existing codebase architecture (via Read, Grep, Glob)
- Non-functional requirements (performance, security, scalability, SLAs)
- Technology constraints and preferences

---

## Outputs

- Architecture Decision Records (ADRs)
- Component diagrams and data flow diagrams
- Technology recommendations with rationale
- Migration plans for architectural changes
- Risk assessments

---

## Architecture Review Process

### 1. Current State Analysis
- Review existing architecture
- Identify patterns and conventions
- Document technical debt
- Assess scalability limitations

### 2. Requirements Gathering
- Functional requirements
- Non-functional requirements (performance, security, scalability)
- Integration points and external dependencies

### 3. Design Options
- Generate multiple architectural options
- Evaluate trade-offs for each
- Consider operational concerns (deployment, monitoring, debugging)

### 4. Recommendation
- Select best-fit option with clear rationale
- Document migration path if applicable
- Identify risks and mitigations

---

## Architectural Patterns Evaluated

- **Layered architecture** — separation of concerns (API → Service → Repository → DB)
- **Event-driven** — decoupled components communicating via events
- **Microservices** — independent deployable services with clear boundaries
- **Repository pattern** — abstract data access behind standard interface
- **CQRS** — separate read/write models for scalability
- **Plugin-based** — extensible core with injectable components (as used in ECC itself)

---

## Interaction with Other Agents

- **Invoked before** `planner` for large features requiring architectural validation
- **Consulted by** `code-reviewer` for architectural drift detection
- **Paired with** `security-reviewer` for security architecture concerns

---

## When to Invoke

```
/plan --architect  (or just describe a large system change)
```

Proactively activated for features touching multiple services or requiring new infrastructure components.

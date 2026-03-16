# Everything Claude Code — Architecture Overview

This directory contains complete architectural documentation for the **Everything Claude Code (ECC)** system — reverse-engineered from the source code and organized for developers who want to understand how ECC works without reading all the source files.

---

## Documentation Structure

```
everything-claude-code-overview/
├── 01-project-overview/
│   └── repository-structure.md     ← Directory layout, technologies, entry points
├── 02-architecture/
│   └── system-architecture.md      ← Components, interactions, Mermaid diagram
├── 03-agents/
│   ├── all-agents-reference.md     ← Complete agent inventory + orchestration patterns
│   ├── planner-agent.md            ← Planning specialist (opus model)
│   ├── architect-agent.md          ← Architecture specialist (opus model)
│   ├── code-reviewer-agent.md      ← Code quality reviewer (sonnet model)
│   ├── tdd-guide-agent.md          ← Test-driven development guide (sonnet model)
│   └── security-reviewer-agent.md  ← Security vulnerability detector (sonnet model)
├── 04-skills/
│   └── skill-system.md             ← Skill categories, lifecycle, evolution system
├── 05-hooks/
│   └── hook-system.md              ← All hook events, scripts, profile system
├── 06-workflows/
│   └── request-lifecycle.md        ← End-to-end request trace with sequence diagram
├── 07-diagrams/
│   └── all-diagrams.md             ← 7 Mermaid diagrams (architecture, agents, hooks, etc.)
├── 08-deep-dives/
│   ├── prompt-engineering.md       ← Agent prompt design patterns
│   ├── agent-orchestration-patterns.md ← 7 orchestration patterns with examples
│   └── tool-invocation-design.md   ← Tool safety, hooks, MCP integration
└── 09-interview-knowledge/
    └── ai-agent-architecture.md    ← Interview prep: patterns, trade-offs, Q&A
```

---

## Quick Navigation

### "What is ECC?"
→ Start with [01-project-overview/repository-structure.md](01-project-overview/repository-structure.md)

### "How does the system architecture work?"
→ [02-architecture/system-architecture.md](02-architecture/system-architecture.md)

### "What agents are available and how do they interact?"
→ [03-agents/all-agents-reference.md](03-agents/all-agents-reference.md)

### "What happens when I type /tdd?"
→ [06-workflows/request-lifecycle.md](06-workflows/request-lifecycle.md)

### "How do hooks work?"
→ [05-hooks/hook-system.md](05-hooks/hook-system.md)

### "What are skills and how are they different from agents?"
→ [04-skills/skill-system.md](04-skills/skill-system.md)

### "How does ECC learn from sessions?"
→ [08-deep-dives/agent-orchestration-patterns.md](08-deep-dives/agent-orchestration-patterns.md) (Pattern 6: Observer-Pattern Learning)

### "Technical interview prep on AI agents"
→ [09-interview-knowledge/ai-agent-architecture.md](09-interview-knowledge/ai-agent-architecture.md)

---

## Key Facts About ECC

| Attribute | Value |
|---|---|
| Package name | `ecc-universal` |
| Version | 1.8.0 |
| License | MIT |
| Supported harnesses | Claude Code, Cursor, OpenCode, Codex |
| Number of agents | 18 |
| Number of skills | 80+ |
| Number of hook scripts | 20+ |
| Number of commands | 40+ |
| Number of MCP configs | 20+ |
| Test count | 997 |
| Languages supported | TypeScript, Python, Go, Kotlin, Java, PHP, Swift, Perl, C++ |
| Runtime | Node.js ≥ 18 |
| State storage | SQLite (sql.js) |

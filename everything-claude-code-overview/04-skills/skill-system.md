# Skill System — Everything Claude Code

## What Is a Skill?

A **skill** in ECC is a reusable knowledge module encoded as a `SKILL.md` Markdown file. Skills are different from agents:

| | Agent | Skill |
|---|---|---|
| **Format** | Markdown + YAML frontmatter | Markdown (`SKILL.md`) |
| **Execution** | Has tools, can run code | Context injection only |
| **Role** | Performs actions | Informs behavior |
| **Activation** | Auto-invoked or slash command | Slash command or context inject |
| **State** | Stateless | Stateless |

Skills provide domain knowledge, workflow patterns, and technology-specific guidance that the AI references during task execution.

---

## Skill Directory

Skills live in `skills/<skill-name>/SKILL.md`. There are 80+ skills across multiple categories.

---

## Skill Categories

### Development Workflow
| Skill | Purpose |
|---|---|
| `tdd-workflow` | Test-driven development: Red→Green→Refactor, 80%+ coverage |
| `e2e-testing` | Playwright E2E patterns, Page Object Model, CI integration |
| `verification-loop` | Comprehensive verification: lint, test, build, coverage |
| `security-review` | OWASP Top 10, secrets detection, input validation checklist |
| `coding-standards` | Universal TypeScript/JS/React best practices |
| `api-design` | REST API design: naming, status codes, pagination, errors, versioning |
| `backend-patterns` | Node.js/Express/Next.js API routes, database patterns |
| `frontend-patterns` | React, Next.js, state management, performance |
| `deployment-patterns` | CI/CD, container, cloud deployment patterns |

### AI Engineering
| Skill | Purpose |
|---|---|
| `agentic-engineering` | Patterns for building reliable AI agent systems |
| `ai-first-engineering` | Engineering practices optimized for AI-assisted development |
| `autonomous-loops` | Reliable long-running agent loops with stall detection |
| `continuous-agent-loop` | Single-agent continuous execution patterns |
| `eval-harness` | Formal evaluation framework for Claude Code sessions |
| `agent-harness-construction` | Building robust agent harness systems |
| `iterative-retrieval` | RAG and context management patterns |
| `cost-aware-llm-pipeline` | Optimizing LLM costs in production pipelines |
| `prompt-optimizer` | Reducing prompt size while preserving behavior |

### Language-Specific
| Skill | Language | Purpose |
|---|---|---|
| `golang-patterns` | Go | Idiomatic Go patterns |
| `golang-testing` | Go | Go testing with testify, mocks |
| `kotlin-patterns` | Kotlin | Kotlin idiomatic patterns |
| `kotlin-testing` | Kotlin | Kotlin testing patterns |
| `kotlin-coroutines-flows` | Kotlin | Coroutines and Flow patterns |
| `python-patterns` | Python | Pythonic code patterns |
| `python-testing` | Python | pytest patterns |
| `django-patterns` | Python/Django | Django best practices |
| `django-tdd` | Python/Django | TDD in Django |
| `springboot-patterns` | Java/Spring | Spring Boot patterns |
| `swiftui-patterns` | Swift | SwiftUI patterns |
| `swift-concurrency-6-2` | Swift | Swift concurrency model |
| `java-coding-standards` | Java | Java best practices |
| `cpp-coding-standards` | C++ | C++ standards |
| `perl-patterns` | Perl | Perl idioms |

### Content & Media
| Skill | Purpose |
|---|---|
| `article-writing` | Long-form technical writing in a distinctive voice |
| `content-engine` | Multi-platform content creation (X, LinkedIn, TikTok) |
| `crosspost` | Cross-platform content distribution |
| `frontend-slides` | HTML presentation builder (zero-dependency, Mermaid-ready) |
| `video-editing` | AI-assisted video editing workflows |
| `fal-ai-media` | Image/video/audio generation via fal.ai |

### Business & Domain
| Skill | Purpose |
|---|---|
| `investor-materials` | Pitch decks, investor memos, financial models |
| `investor-outreach` | Cold emails, warm intro blurbs, fundraising comms |
| `market-research` | Competitive analysis, market sizing, industry intel |
| `deep-research` | Multi-source research with citations |
| `exa-search` | Neural web search via Exa |
| `x-api` | X/Twitter API integration |

---

## Skill Lifecycle

### 1. Discovery
Skills are discovered at install time or referenced in `manifests/`. The `skills-health.js` script validates skill health and availability.

### 2. Activation

**Via slash command:**
```
/tdd → activates tdd-workflow skill
/security → activates security-review skill
/e2e → activates e2e-testing skill
```

**Via agent invocation:**
An agent's system prompt may reference or inject a skill's content into context.

**Via `SessionStart` hook:**
Frequently-used skills can be loaded automatically at session start via the session context injection system.

### 3. Context Injection
The skill's `SKILL.md` content is read and injected into the active context window. The AI then uses this knowledge to guide its responses for the duration of the task.

### 4. Evolution
The continuous learning system tracks which skills are invoked, evaluates their effectiveness, and surfaces improvement suggestions via the `skill-improvement` library.

---

## Skill Format

```markdown
---
name: skill-name
description: One-line description
origin: ECC | Community | <author>
---

# Skill Title

## When to Activate
[Conditions under which this skill should be used]

## Core Principles / How It Works
[Domain knowledge, patterns, rules]

## Examples
[Code examples, workflows, templates]
```

---

## Skill Evolution System

Located in `scripts/lib/skill-evolution/`, this system tracks:

| Module | Purpose |
|---|---|
| `index.js` | Entry point, coordinates evolution pipeline |
| `tracker.js` | Records skill invocation events |
| `provenance.js` | Tracks origin and modification history |
| `versioning.js` | Semantic versioning for skill updates |
| `health.js` | Health checks: missing fields, broken links |

The `skill-improvement/` library provides:
| Module | Purpose |
|---|---|
| `observations.js` | Captures usage observations from hooks |
| `evaluate.js` | Scores skill effectiveness |
| `amendify.js` | Generates improvement suggestions |
| `health.js` | Reports overall skill health |

---

## Continuous Learning Integration

The `skills/continuous-learning-v2/` skill implements an observer-pattern loop:

1. `hooks/observe.sh` — captures tool use observations (async, triggered on every PreToolUse and PostToolUse)
2. `agents/observer.md` — AI agent that analyzes observations
3. `scripts/instinct-cli.py` — CLI to manage instincts (extract, list, apply)
4. Instincts are stored in the SQLite state store and loaded at SessionStart

This enables ECC to learn from usage patterns and improve skill recommendations over time.

---

## Where Skills Are Used

- **Claude Code** — `skills/` directory
- **Cursor IDE** — `.cursor/skills/` (subset)
- **OpenCode** — `.opencode/commands/` (skill-as-command pattern)
- **Codex** — Referenced in `.codex/` agents
- **OpenAI agent format** — `.agents/skills/<name>/agents/openai.yaml`

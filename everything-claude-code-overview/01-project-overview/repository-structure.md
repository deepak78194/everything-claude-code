# Repository Structure — Everything Claude Code (ECC)

## High-Level Project Description

**Everything Claude Code (ECC)** is a production-grade agent harness performance system for Claude Code and other AI coding tools. It is not simply a configuration pack — it is a complete system providing:

- Specialized AI sub-agents for delegating complex tasks
- Reusable skills (workflow and domain knowledge modules)
- Lifecycle hooks for session automation and memory persistence
- Slash commands invoking specific workflows
- Always-follow coding rules per language
- MCP server configurations for external integrations
- Cross-platform Node.js scripts for orchestration and install management

**Package name:** `ecc-universal`  
**Current version:** 1.8.0  
**License:** MIT  
**Supported harnesses:** Claude Code, Cursor IDE, OpenCode, Codex (OpenAI)  
**Supported languages:** TypeScript, Python, Go, Kotlin, Java, PHP, Swift, Perl, C++

---

## Directory Explanations

```
everything-claude-code/
├── agents/                  # Specialized AI sub-agents (Markdown + YAML frontmatter)
├── commands/                # Slash commands (e.g. /tdd, /plan, /e2e)
├── contexts/                # Context injection files for session enrichment
├── docs/                    # Multi-language translations of README
├── examples/                # Example CLAUDE.md and user configuration files
├── hooks/                   # Claude Code hook definitions (hooks.json)
├── manifests/               # Install manifests listing what gets installed per profile
├── mcp-configs/             # Model Context Protocol server configurations
├── plugins/                 # Plugin descriptors for Claude Code plugin marketplace
├── rules/                   # Always-follow language-specific and common coding rules
├── schemas/                 # JSON Schema validation files for config formats
├── scripts/                 # Node.js utilities: install, hooks, orchestration, CLI
│   ├── ci/                  # CI validation scripts (agents, commands, rules, etc.)
│   ├── hooks/               # Hook script implementations (JS)
│   └── lib/                 # Shared libraries: package manager, session, state store
├── skills/                  # Skill definitions (workflow + domain knowledge)
├── tests/                   # Test suite (Node.js, no framework dependency)
│   ├── ci/                  # Validator tests
│   ├── hooks/               # Hook behavior tests
│   ├── integration/         # End-to-end integration tests
│   ├── lib/                 # Unit tests for lib modules
│   └── scripts/             # Script behavior tests
├── .agents/                 # OpenAI / multi-harness agent skill definitions
├── .claude/                 # Claude-specific config and skills
├── .claude-plugin/          # Claude Code plugin manifest (plugin.json, marketplace.json)
├── .codex/                  # Codex (OpenAI) agent definitions and config
├── .cursor/                 # Cursor IDE hooks, rules, and skills
├── .opencode/               # OpenCode harness prompts, commands, tools, plugins
├── AGENTS.md                # Cross-harness agent summary (for Codex/OpenAI compatibility)
├── CLAUDE.md                # Claude Code project guidance file
├── install.sh               # Shell-based installer entry point
├── package.json             # npm package definition (bin: ecc)
└── README.md                # Main documentation with guides and changelog
```

---

## Important Files and Their Purposes

| File / Path | Purpose |
|---|---|
| `package.json` | npm package definition; defines `ecc` CLI binary via `scripts/ecc.js`; lists all published files |
| `install.sh` | Shell installer; copies components into target project |
| `scripts/ecc.js` | Primary CLI entry point: `npx ecc typescript` installs the TypeScript profile |
| `scripts/install-plan.js` | Resolves which files to install based on profile selection |
| `scripts/install-apply.js` | Applies the install plan: copies files to target directories |
| `hooks/hooks.json` | Main Claude Code hook configuration (PreToolUse, PostToolUse, Stop, SessionStart, etc.) |
| `mcp-configs/mcp-servers.json` | 20+ pre-configured MCP server entries for GitHub, Supabase, Firecrawl, Playwright, etc. |
| `schemas/hooks.schema.json` | JSON Schema validating hook configuration structure |
| `schemas/plugin.schema.json` | JSON Schema validating plugin descriptor format |
| `.claude-plugin/plugin.json` | Claude Code plugin marketplace descriptor |
| `AGENTS.md` | Top-level agent manifest for Codex and cross-harness compatibility |
| `rules/common/` | Language-agnostic coding rules (security, testing, agents, git, patterns) |
| `rules/typescript/` | TypeScript-specific rules (coding-style, patterns, hooks, security, testing) |
| `manifests/` | Install profile manifests mapping profile names to component lists |
| `tests/run-all.js` | Test runner: executes all test files sequentially |

---

## Frameworks and Technologies Used

| Technology | Role |
|---|---|
| **Node.js ≥ 18** | Runtime for all scripts and tests |
| **Markdown + YAML frontmatter** | Agent and skill definition format |
| **JSON** | Hook configs, MCP configs, plugin descriptors, schemas |
| **AJV (Ajv)** | JSON Schema validation in CI validators |
| **ESLint** | JavaScript linting (`eslint.config.js`) |
| **markdownlint** | Markdown linting (`.markdownlint.json`) |
| **c8** | Code coverage (targets 80% lines/branches/functions/statements) |
| **sql.js** | SQLite-based state store for session persistence |
| **Mermaid** | Diagram syntax used in documentation |
| **tmux** | Terminal multiplexer used for parallel agent orchestration |
| **GitHub Actions** | CI/CD pipeline (`.github/workflows/`) |
| **Model Context Protocol (MCP)** | Protocol for connecting Claude Code to external services |

---

## Entry Points

1. **npm install**: `npm install -g ecc-universal` or `npx ecc-universal`
2. **CLI**: `npx ecc <profile>` (e.g., `npx ecc typescript`)
3. **Shell installer**: `bash install.sh typescript`
4. **Claude Code plugin**: Auto-installed via the Claude Code plugin marketplace

---

## Build System

The project has no transpile/build step for the main package. All scripts are plain Node.js CommonJS modules. The `.opencode/` directory contains a TypeScript sub-project with its own `tsconfig.json` and `package.json` for the OpenCode harness integration.

```bash
# Run all tests + CI validators
npm test

# Run only unit/integration tests
node tests/run-all.js

# Run coverage report
npm run coverage

# Lint all JS and Markdown
npm run lint
```

---

## Dependency Management

**Runtime dependency:**
- `sql.js ^1.14.1` — WebAssembly SQLite for the session state store

**Dev dependencies:**
- `@eslint/js`, `eslint`, `globals` — JavaScript linting
- `ajv` — JSON Schema validation
- `markdownlint-cli` — Markdown quality checks
- `c8` — Code coverage

Package manager detection at runtime supports: `npm`, `pnpm`, `yarn`, `bun`. Configurable via `CLAUDE_PACKAGE_MANAGER` environment variable or `.claude/package-manager.json`.

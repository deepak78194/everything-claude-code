# Repository Structure — Everything Claude Code (ECC)

**Version:** 1.8.0  
**Analysis Date:** 2026-03-17  

---

## Top-Level Layout

```
everything-claude-code/
├── agents/                    # 25 specialized AI sub-agents (Markdown + YAML frontmatter)
├── skills/                    # 100+ reusable knowledge modules (SKILL.md files)
├── commands/                  # 52 slash commands (Markdown with description frontmatter)
├── hooks/                     # Lifecycle hook configuration (hooks.json)
├── rules/                     # Always-follow guidelines (language-organized Markdown)
├── mcp-configs/               # 20+ MCP server connection configs (JSON)
├── scripts/                   # Node.js utilities (hooks, install, CLI, orchestration)
├── tests/                     # Test suite (Node.js, 997 tests)
├── contexts/                  # Reusable context files (dev, research, review modes)
├── docs/                      # Architecture decision records and planning docs
├── examples/                  # CLAUDE.md examples for different tech stacks
├── manifests/                 # Install manifests (JSON — components, modules, profiles)
├── schemas/                   # JSON Schema validation for config files
├── plugins/                   # Plugin registration system
├── assets/                    # Static assets
├── everything-claude-code-overview/  # Architectural documentation (reverse-engineered)
├── .claude/                   # Claude Code plugin config (package-manager.json)
├── .claude-plugin/            # Plugin metadata (plugin.json, marketplace.json)
├── .codex/                    # Codex harness config (AGENTS.md, config.toml)
├── .cursor/                   # Cursor harness config (hooks.json)
├── .opencode/                 # OpenCode harness config (opencode.json, index.ts)
├── .agents                    # Multi-harness agent resolver (root-level)
├── AGENTS.md                  # Root-level Codex/multi-harness agent instructions
├── CLAUDE.md                  # Root-level Claude Code project instructions
├── README.md                  # Public documentation
├── install.sh                 # Linux/macOS installer
├── install.ps1                # Windows PowerShell installer
├── package.json               # npm package metadata
└── VERSION                    # Current version string
```

---

## agents/

25 AI sub-agent definitions. Each file is Markdown with YAML frontmatter:

```
agents/
├── architect.md               # System design, scalability decisions (model: opus)
├── planner.md                 # Implementation plans, step breakdowns (model: opus)
├── chief-of-staff.md          # Multi-channel communication triage (model: opus)
├── build-error-resolver.md    # Fix build/TypeScript errors (model: sonnet)
├── code-reviewer.md           # Code quality and security review (model: sonnet)
├── cpp-build-resolver.md      # Fix C++ build errors (model: sonnet)
├── cpp-reviewer.md            # C++ code review (model: sonnet)
├── database-reviewer.md       # PostgreSQL/Supabase schema & query (model: sonnet)
├── doc-updater.md             # Keep docs and codemaps current (model: haiku)
├── docs-lookup.md             # Fetch live library docs via Context7 MCP (model: sonnet)
├── e2e-runner.md              # Generate and run Playwright E2E tests (model: sonnet)
├── go-build-resolver.md       # Fix Go build errors (model: sonnet)
├── go-reviewer.md             # Go code review (model: sonnet)
├── harness-optimizer.md       # Tune agent harness config (model: sonnet)
├── java-build-resolver.md     # Fix Java/Maven/Gradle build errors (model: sonnet)
├── java-reviewer.md           # Java and Spring Boot code review (model: sonnet)
├── kotlin-build-resolver.md   # Fix Kotlin/Gradle build errors (model: sonnet)
├── kotlin-reviewer.md         # Kotlin/Android/KMP code review (model: sonnet)
├── loop-operator.md           # Operate autonomous loops safely (model: sonnet)
├── python-reviewer.md         # Python code review (model: sonnet)
├── refactor-cleaner.md        # Dead code cleanup, deduplication (model: sonnet)
├── rust-build-resolver.md     # Fix Rust build errors (model: sonnet)
├── rust-reviewer.md           # Rust code review (model: sonnet)
├── security-reviewer.md       # OWASP Top 10, secrets, input validation (model: sonnet)
└── tdd-guide.md               # Enforce test-first development (model: sonnet)
```

**File format:**
```yaml
---
name: <identifier>
description: <routing hint — determines when auto-invoked>
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: opus | sonnet | haiku
---
<system prompt body (Markdown)>
```

---

## skills/

100+ skill modules organized as directories containing `SKILL.md`:

```
skills/
├── agent-harness-construction/    # Build and configure agent harnesses
├── agentic-engineering/           # Agentic system design patterns
├── ai-first-engineering/          # AI-first development principles
├── ai-regression-testing/         # Regression testing for AI outputs
├── android-clean-architecture/    # Android Clean Architecture patterns
├── api-design/                    # REST API design standards
├── article-writing/               # Long-form content creation
├── autonomous-loops/              # Autonomous agent loop patterns
├── backend-patterns/              # Node.js/Express backend patterns
├── blueprint/                     # Project blueprint templates
├── bun-runtime/                   # Bun runtime patterns
├── claude-api/                    # Anthropic Claude API usage
├── claude-devfleet/               # Multi-agent fleet orchestration
├── clickhouse-io/                 # ClickHouse analytics patterns
├── coding-standards/              # Universal coding standards
├── compose-multiplatform-patterns/ # Kotlin Compose Multiplatform
├── configure-ecc/                 # ECC configuration guide
├── content-engine/                # Multi-platform content creation
├── content-hash-cache-pattern/    # Cache invalidation patterns
├── continuous-agent-loop/         # Continuous execution patterns
├── continuous-learning/           # Session learning capture (v1)
├── continuous-learning-v2/        # Session learning with hooks (v2)
├── cost-aware-llm-pipeline/       # LLM cost optimization
├── cpp-coding-standards/          # C++ coding conventions
├── cpp-testing/                   # C++ testing patterns
├── crosspost/                     # Multi-platform social posting
├── data-scraper-agent/            # Web data extraction patterns
├── database-migrations/           # Database migration strategies
├── deep-research/                 # Multi-source research workflows
├── deployment-patterns/           # CI/CD deployment strategies
├── django-patterns/               # Django web framework patterns
├── django-security/               # Django security checklist
├── django-tdd/                    # Django test-driven development
├── django-verification/           # Django verification patterns
├── dmux-workflows/                # tmux-based multi-agent workflows
├── docker-patterns/               # Docker containerization patterns
├── documentation-lookup/          # Live docs via Context7 MCP
├── e2e-testing/                   # Playwright E2E testing patterns
├── energy-procurement/            # Energy market business domain
├── enterprise-agent-ops/          # Enterprise agent operations
├── eval-harness/                  # Evaluation framework for agents
├── exa-search/                    # Neural web search via Exa
├── fal-ai-media/                  # AI media generation (image/video/audio)
├── foundation-models-on-device/   # On-device ML patterns
├── frontend-patterns/             # React/Next.js frontend patterns
├── frontend-slides/               # HTML presentation builder
├── golang-patterns/               # Go language patterns
├── golang-testing/                # Go testing strategies
├── inventory-demand-planning/     # Supply chain business domain
├── investor-materials/            # Pitch deck and investor docs
├── investor-outreach/             # Investor communication templates
├── iterative-retrieval/           # RAG and retrieval patterns
├── java-coding-standards/         # Java coding conventions
├── jpa-patterns/                  # Java Persistence API patterns
├── kotlin-coroutines-flows/       # Kotlin async patterns
├── kotlin-exposed-patterns/       # Kotlin Exposed ORM patterns
├── kotlin-ktor-patterns/          # Ktor web framework patterns
├── kotlin-patterns/               # General Kotlin patterns
├── kotlin-testing/                # Kotlin testing strategies
├── laravel-patterns/              # Laravel PHP patterns
├── laravel-security/              # Laravel security checklist
├── laravel-tdd/                   # Laravel test-driven development
├── laravel-verification/          # Laravel verification patterns
├── liquid-glass-design/           # iOS Liquid Glass UI design
├── logistics-exception-management/ # Logistics business domain
├── market-research/               # Competitive analysis workflows
├── mcp-server-patterns/           # MCP server implementation
├── nanoclaw-repl/                 # NanoClaw REPL interface
├── nextjs-turbopack/              # Next.js + Turbopack patterns
├── nutrient-document-processing/  # Document processing domain
├── perl-patterns/                 # Perl language patterns
├── perl-security/                 # Perl security checklist
├── perl-testing/                  # Perl testing strategies
├── plankton-code-quality/         # Code quality metrics
├── postgres-patterns/             # PostgreSQL query patterns
├── production-scheduling/         # Production scheduling domain
├── project-guidelines-example/    # CLAUDE.md example templates
├── prompt-optimizer/              # Prompt engineering optimization
├── python-patterns/               # Python language patterns
├── python-testing/                # Python testing strategies
├── quality-nonconformance/        # QA nonconformance tracking
├── ralphinho-rfc-pipeline/        # RFC writing pipeline
├── regex-vs-llm-structured-text/  # Regex vs LLM comparison
├── returns-reverse-logistics/     # Returns management domain
├── rust-patterns/                 # Rust language patterns
├── rust-testing/                  # Rust testing strategies
├── search-first/                  # Search-before-implement pattern
├── security-review/               # Security audit workflow
├── security-scan/                 # Automated security scanning
├── skill-stocktake/               # Skill inventory and review
├── springboot-patterns/           # Spring Boot patterns
├── springboot-security/           # Spring Boot security
├── springboot-tdd/                # Spring Boot TDD
├── springboot-verification/       # Spring Boot verification
├── strategic-compact/             # Context compaction strategies
├── swift-actor-persistence/       # Swift actor state persistence
├── swift-concurrency-6-2/         # Swift 6.2 concurrency patterns
├── swift-protocol-di-testing/     # Swift protocol DI testing
├── swiftui-patterns/              # SwiftUI development patterns
├── tdd-workflow/                  # Universal TDD workflow
├── team-builder/                  # Team structure and hiring
├── verification-loop/             # Verification and eval loops
├── video-editing/                 # AI-assisted video editing
├── videodb/                       # VideoDB integration
├── visa-doc-translate/            # Document translation domain
└── x-api/                         # X/Twitter API integration
```

**File format:**
```markdown
<!-- skills/<name>/SKILL.md -->
# <Skill Title>

## When to Use
...

## How It Works
...

## Examples
...
```

---

## commands/

52 slash commands invoked as `/command-name`:

```
commands/
├── aside.md               # Save a thought or note aside
├── build-fix.md           # Fix build errors (delegates to build-error-resolver)
├── checkpoint.md          # Save session checkpoint
├── claw.md                # NanoClaw REPL interface
├── code-review.md         # Run code review (delegates to code-reviewer)
├── cpp-build.md           # C++ build and fix
├── cpp-review.md          # C++ code review
├── cpp-test.md            # C++ test runner
├── devfleet.md            # DevFleet multi-agent orchestration
├── docs.md                # Update documentation
├── e2e.md                 # Generate/run E2E tests
├── eval.md                # Run evaluation harness
├── evolve.md              # Evolve/improve a skill
├── go-build.md            # Go build and fix
├── go-review.md           # Go code review
├── go-test.md             # Go test runner
├── gradle-build.md        # Gradle build and fix
├── harness-audit.md       # Audit agent harness configuration
├── instinct-export.md     # Export learned patterns
├── instinct-import.md     # Import patterns from file
├── instinct-status.md     # Show current instincts/patterns
├── kotlin-build.md        # Kotlin build and fix
├── kotlin-review.md       # Kotlin code review
├── kotlin-test.md         # Kotlin test runner
├── learn.md               # Extract patterns from this session
├── learn-eval.md          # Evaluate session for extractable patterns
├── loop-start.md          # Start an autonomous loop
├── loop-status.md         # Check loop progress
├── model-route.md         # Configure model routing rules
├── multi-backend.md       # Multi-backend orchestration
├── multi-execute.md       # Execute tasks across multiple contexts
├── multi-frontend.md      # Multi-frontend orchestration
├── multi-plan.md          # Plan across multiple work streams
├── multi-workflow.md      # Orchestrate multi-step workflows
├── orchestrate.md         # Orchestrate multiple agents
├── plan.md                # Create implementation plan
├── pm2.md                 # PM2 process management
├── projects.md            # List and switch projects
├── promote.md             # Promote skill from session
├── prompt-optimize.md     # Optimize a prompt
├── python-review.md       # Python code review
├── quality-gate.md        # Run quality gate checks
├── refactor-clean.md      # Clean up dead code
├── resume-session.md      # Resume previous session context
├── rust-build.md          # Rust build and fix
├── rust-review.md         # Rust code review
├── rust-test.md           # Rust test runner
├── save-session.md        # Manually save session state
├── sessions.md            # List/manage sessions
├── setup-pm.md            # Set up package manager
├── skill-create.md        # Create a new skill from git history
├── skill-health.md        # Check skill system health
├── tdd.md                 # Run TDD workflow
├── test-coverage.md       # Check test coverage
├── update-codemaps.md     # Update architectural code maps
├── update-docs.md         # Update project documentation
└── verify.md              # Run verification loop
```

---

## hooks/

```
hooks/
├── hooks.json             # Hook configuration (all lifecycle events)
└── README.md              # Hook system documentation
```

**hooks.json** defines hooks for 6 lifecycle events:
- `SessionStart` — run once when session begins
- `PreToolUse` — run before each tool invocation
- `PostToolUse` — run after each tool invocation
- `PreCompact` — run before context compression
- `Stop` — run after each AI response
- `SessionEnd` — run when session ends

---

## scripts/

```
scripts/
├── claw.js                    # NanoClaw REPL entry point
├── doctor.js                  # Diagnose installation issues
├── ecc.js                     # Main CLI entry point (npx ecc)
├── harness-audit.js           # Harness configuration audit
├── install-apply.js           # Apply install plan to filesystem
├── install-plan.js            # Resolve install components
├── list-installed.js          # List installed ECC components
├── orchestrate-codex-worker.sh # Codex orchestration worker
├── orchestrate-worktrees.js   # Git worktree orchestration
├── orchestration-status.js    # Show orchestration progress
├── release.sh                 # Release automation
├── repair.js                  # Repair broken installation
├── session-inspect.js         # Inspect session data
├── sessions-cli.js            # Session management CLI
├── setup-package-manager.js   # Detect and configure package manager
├── skill-create-output.js     # Generate skill from session
├── skills-health.js           # Check skill system health
├── status.js                  # Show ECC installation status
├── sync-ecc-to-codex.sh       # Sync ECC config to Codex format
├── uninstall.js               # Remove ECC installation
└── hooks/                     # Individual hook script implementations
    ├── auto-tmux-dev.js        # Auto-start dev servers
    ├── build-complete.js       # Post-build analysis
    ├── check-console-log.js    # Detect console.log statements
    ├── cost-tracker.js         # Token/cost metrics
    ├── doc-file-warning.js     # Warn about non-standard docs
    ├── evaluate-session.js     # Evaluate patterns in session
    ├── insaits-security-wrapper.js  # AI security monitoring
    ├── observe.sh              # Capture tool use observations
    ├── post-bash-build-complete.js  # Post-build hook
    ├── post-bash-pr-created.js # Post-PR creation hook
    ├── post-edit-console-warn.js    # Console.log warning
    ├── post-edit-format.js     # Auto-format after edits
    ├── post-edit-typecheck.js  # TypeScript check after edits
    ├── pre-bash-git-push-reminder.js # Git push reminder
    ├── pre-bash-tmux-reminder.js    # tmux reminder
    ├── pre-compact.js          # Save state before compact
    ├── quality-gate.js         # Quality gate checks
    ├── run-with-flags.js       # Profile-gated hook runner
    ├── run-with-flags-shell.sh # Shell-based profile runner
    ├── session-end.js          # Persist session state
    ├── session-end-marker.js   # Session end lifecycle marker
    ├── session-start.js        # Load prior context
    └── suggest-compact.js      # Suggest manual compaction
```

---

## rules/

Always-follow guidelines organized by language:

```
rules/
├── README.md
├── common/
│   ├── agents.md          # Agent delegation guidelines
│   ├── git-workflow.md    # Git commit and PR standards
│   ├── patterns.md        # Universal coding patterns
│   ├── security.md        # Security-first rules
│   └── testing.md         # Testing requirements
├── typescript/
├── python/
├── golang/
├── kotlin/
├── swift/
├── php/
└── perl/
```

---

## mcp-configs/

```
mcp-configs/
└── mcp-servers.json       # 20+ MCP server configurations
```

Defines connection parameters for: github, firecrawl, supabase, memory, sequential-thinking, vercel, railway, cloudflare, playwright, exa-web-search, context7, insaits, fal-ai, and more.

---

## tests/

```
tests/
├── run-all.js                  # Test runner entry point
├── codex-config.test.js        # Codex configuration validation
├── opencode-config.test.js     # OpenCode configuration validation
├── lib/
│   ├── utils.test.js           # Utility function tests
│   └── package-manager.test.js # Package manager detection tests
└── hooks/
    └── hooks.test.js           # Hook system tests
```

---

## contexts/

Reusable context modes loaded at session start:

```
contexts/
├── dev.md       # Development mode context (active coding)
├── research.md  # Research mode context (exploration, analysis)
└── review.md    # Review mode context (code/security review)
```

---

## docs/

Architecture and planning documentation:

```
docs/
├── ARCHITECTURE-IMPROVEMENTS.md
├── COMMAND-AGENT-MAP.md
├── ECC-2.0-SESSION-ADAPTER-DISCOVERY.md
├── MEGA-PLAN-REPO-PROMPTS-2026-03-12.md
├── PHASE1-ISSUE-BUNDLE-2026-03-12.md
├── PR-399-REVIEW-2026-03-12.md
├── PR-QUEUE-TRIAGE-2026-03-13.md
├── SELECTIVE-INSTALL-ARCHITECTURE.md
├── SELECTIVE-INSTALL-DESIGN.md
├── SESSION-ADAPTER-CONTRACT.md
├── continuous-learning-v2-spec.md
└── token-optimization.md
```

---

## manifests/

JSON manifests controlling installation:

```
manifests/
├── install-components.json    # Individual installable components
├── install-modules.json       # Named module bundles
└── install-profiles.json      # Preset installation profiles (minimal/standard/full)
```

---

## schemas/

JSON Schema files for config validation:

```
schemas/
├── ecc-install-config.schema.json
├── hooks.schema.json
├── install-components.schema.json
├── install-modules.schema.json
├── install-profiles.schema.json
├── install-state.schema.json
├── package-manager.schema.json
├── plugin.schema.json
└── state-store.schema.json
```

---

## examples/

Reference `CLAUDE.md` files for different tech stacks:

```
examples/
├── CLAUDE.md                  # Generic project template
├── django-api-CLAUDE.md       # Django API project
├── go-microservice-CLAUDE.md  # Go microservice project
├── laravel-api-CLAUDE.md      # Laravel API project
├── rust-api-CLAUDE.md         # Rust API project
├── saas-nextjs-CLAUDE.md      # Next.js SaaS project
├── user-CLAUDE.md             # User-level preferences
└── statusline.json            # Status display configuration
```

---

## Harness-Specific Directories

ECC supports multiple AI agent harnesses via harness-specific config directories:

```
.claude/
└── package-manager.json   # Detected package manager config

.claude-plugin/
├── plugin.json            # Claude Code plugin manifest
├── marketplace.json       # Marketplace listing metadata
└── README.md

.codex/
├── AGENTS.md              # Codex-format agent instructions
└── config.toml            # Codex runtime configuration

.cursor/
└── hooks.json             # Cursor hooks configuration

.opencode/
├── opencode.json          # OpenCode runtime config
├── index.ts               # OpenCode entry point
├── MIGRATION.md           # Migration guide from Claude Code
└── README.md
```

---

## Key Configuration Files

| File | Purpose |
|---|---|
| `CLAUDE.md` | Claude Code project-level instructions |
| `AGENTS.md` | Multi-harness agent instruction file |
| `hooks/hooks.json` | Lifecycle hook configuration |
| `mcp-configs/mcp-servers.json` | MCP server connections |
| `.claude/package-manager.json` | Package manager detection result |
| `.claude-plugin/plugin.json` | Plugin schema and metadata |
| `manifests/install-profiles.json` | Install profile definitions |
| `package.json` | npm package config, scripts, and entry points |
| `VERSION` | Current version string |

---

## Entry Points

| Entry Point | Purpose |
|---|---|
| `install.sh` | Linux/macOS installation |
| `install.ps1` | Windows PowerShell installation |
| `npx ecc` → `scripts/ecc.js` | Primary CLI |
| `hooks/hooks.json` | Hook system registration |
| `CLAUDE.md` | Claude Code project context |
| `AGENTS.md` | Codex/multi-harness context |

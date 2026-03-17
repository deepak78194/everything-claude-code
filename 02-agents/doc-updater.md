# Agent: doc-updater

**Category:** Documentation  
**Model Tier:** `haiku` (fast, cost-efficient for repetitive tasks)  
**Source:** `agents/doc-updater.md`

---

## Purpose

The doc-updater agent keeps **documentation and architectural codemaps synchronized** with the actual codebase. It generates codemaps from AST analysis, refreshes READMEs, and ensures docs reflect the current state of the code.

---

## When to Use

Invoke this agent:
- After adding or changing significant code
- When README is out of date
- When codemaps don't reflect current architecture
- Before a release to ensure docs are accurate
- When onboarding documentation needs updating

---

## Responsibilities

1. **Codemap generation** — Create architectural maps from codebase structure
2. **Documentation updates** — Refresh READMEs and guides
3. **AST analysis** — Use TypeScript compiler API to understand structure
4. **Dependency mapping** — Track imports/exports across modules
5. **Documentation quality** — Ensure docs match code reality

---

## Inputs

- Request to update docs or codemaps
- Access to all files (Read/Write/Edit/Bash/Grep/Glob)

## Outputs

- Updated `docs/CODEMAPS/` with current architecture maps
- Refreshed README files
- Updated API documentation

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read source files for documentation extraction |
| `Write` | Create new documentation files |
| `Edit` | Update existing documentation |
| `Bash` | Run AST analysis and documentation tools |
| `Grep` | Search for function signatures and exports |
| `Glob` | Discover files to document |

---

## Codemap Format

```markdown
# Codemap: <Module Name>

## Purpose
<What this module does>

## Entry Points
- `<function/class>` — <description>

## Public API
| Export | Type | Description |
|---|---|---|
| `functionName` | `(args) => ReturnType` | What it does |

## Dependencies
- Imports from: `<other-module>`
- Imported by: `<consumer-modules>`

## Key Data Flows
1. <step>
2. <step>
```

---

## Update Workflow

### 1. Scan for changes
```bash
git diff --name-only HEAD~1  # Files changed in last commit
```

### 2. Identify documentation impact
- New files → new entries in codemap
- Renamed exports → update all references
- Deleted modules → remove from codemap

### 3. Regenerate affected codemaps
```bash
# TypeScript AST analysis example
npx ts-morph-analyzer src/ --output docs/CODEMAPS/
```

### 4. Validate consistency
```bash
# Check all links in docs are valid
npx markdown-link-check docs/**/*.md
```

---

## Documentation Standards

### What to Document
- Public API functions and their parameters
- Module responsibilities and boundaries
- Data flow between components
- Configuration options with defaults
- Breaking changes in CHANGELOG

### What Not to Document
- Implementation details (let code speak)
- Obvious variable names
- Trivial getters/setters

---

## Platform Adaptation

### GitHub Copilot
Add documentation update instructions to `copilot-instructions.md`. Use GitHub Actions to auto-generate API docs on push:
```yaml
- name: Generate docs
  run: npm run docs:generate
- name: Commit docs
  uses: stefanzweifel/git-auto-commit-action@v4
  with:
    commit_message: "docs: auto-update codemaps"
```

### IntelliJ AI Assistant
Create a "Update Documentation" AI Action. Supplement with IntelliJ's JavaDoc/KDoc generation for Java/Kotlin codebases.

### Monorepo Usage
Generate codemaps per service. Maintain a root-level codemap showing inter-service dependencies.

---

## Dependencies

- Often triggered after: **refactor-cleaner** (docs need updating after code cleanup)
- Often triggered after: **planner** (plan describes architectural changes to document)
- Complements: **docs-lookup** (docs-lookup finds external docs, doc-updater maintains internal docs)

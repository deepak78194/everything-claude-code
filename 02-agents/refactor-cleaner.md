# Agent: refactor-cleaner

**Category:** Quality  
**Model Tier:** `sonnet`  
**Source:** `agents/refactor-cleaner.md`

---

## Purpose

The refactor-cleaner agent identifies and removes **dead code, duplicates, and unused dependencies**. It runs analysis tools to find unused code and safely removes it while ensuring tests continue to pass.

---

## When to Use

Invoke this agent:
- When the codebase has grown organically and accumulated dead code
- Before a major release to reduce bundle/binary size
- When onboarding new engineers (clean codebase is easier to learn)
- After removing a feature (clean up the leftovers)
- When dependency count is high and slowing build times

---

## Responsibilities

1. **Dead code detection** — Find unused functions, classes, exports
2. **Duplicate elimination** — Identify and consolidate repeated logic
3. **Dependency cleanup** — Remove unused packages and imports
4. **Safe refactoring** — Verify tests pass before and after each change

---

## Inputs

- Request to clean up specific files, modules, or the whole codebase
- Access to all files (Read/Write/Edit/Bash/Grep/Glob)
- Test suite to validate safety of changes

## Outputs

- Cleaned source files with dead code removed
- Updated `package.json` with unused dependencies removed
- Summary of what was removed and why

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read files to understand code |
| `Write` | Create replacement files |
| `Edit` | Remove dead code from files |
| `Bash` | Run analysis tools and tests |
| `Grep` | Search for usages before deletion |
| `Glob` | Find files to analyze |

---

## Detection Commands

### JavaScript/TypeScript

```bash
# Find unused exports and imports
npx knip

# Find unused dependencies
npx depcheck

# Find unused TypeScript exports
npx ts-prune

# Find all unused variables (via TypeScript compiler)
npx tsc --noEmit --strict
```

### Python

```bash
# Find unused imports
pip install autoflake
autoflake --check --remove-unused-variables --remove-all-unused-imports -r src/

# Find unused code
pip install vulture
vulture src/
```

### Go

```bash
# Find unused code
go vet ./...
staticcheck ./...
```

---

## Safety Protocol

Before removing any code:

1. **Verify it is unused** — Search all call sites with `grep`
2. **Check for dynamic usage** — Look for string-based invocation, reflection, eval
3. **Check test coverage** — Run tests before and after
4. **Preserve public APIs** — Only remove internal/private code without external consumers

```bash
# Always run tests after cleanup
npm test
# Verify build still passes
npm run build
```

---

## Refactoring Patterns

### Extract Duplicate Logic

```typescript
// BEFORE: duplicate validation in 3 places
function createUser(data) {
  if (!data.email || !data.email.includes('@')) throw new Error('Invalid email');
  // ...
}
function updateUser(data) {
  if (!data.email || !data.email.includes('@')) throw new Error('Invalid email');
  // ...
}

// AFTER: single shared validator
function validateEmail(email: string): void {
  if (!email || !email.includes('@')) throw new Error('Invalid email');
}
```

### Remove Dead Feature Flags

```typescript
// BEFORE: flag never flipped on
if (process.env.FEATURE_OLD_FLOW === 'true') {
  // 200 lines of dead code
} else {
  // actual current code
}

// AFTER: remove dead branch entirely
// actual current code
```

---

## Platform Adaptation

### GitHub Copilot
Use Copilot to assist with manual dead code removal. Add cleanup instructions to PR templates. Use `knip` / `depcheck` in CI to prevent new dead code accumulation.

### IntelliJ AI Assistant
Use IntelliJ's built-in unused code inspections. Supplement with an AI Action for bulk dead code removal across a module.

### Monorepo Usage
Run analysis per service with scoped knip configurations:
```json
// knip.json (per service)
{
  "entry": ["src/index.ts"],
  "project": ["src/**/*.ts"]
}
```

---

## Dependencies

- Often preceded by: **code-reviewer** (review identifies dead code)
- Followed by: **code-reviewer** (review the cleanup)
- May trigger: **build-error-resolver** (if cleanup removes something still referenced)
- Validated by: **tdd-guide** (tests must pass after cleanup)

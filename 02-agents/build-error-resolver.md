# Agent: build-error-resolver

**Category:** Build  
**Model Tier:** `sonnet`  
**Source:** `agents/build-error-resolver.md`

---

## Purpose

The build-error-resolver agent **diagnoses and fixes build failures and type errors** with minimal changes. It focuses exclusively on getting the build green — no refactoring, no architecture changes, no style improvements.

---

## When to Use

Invoke this agent:
- When the build fails
- When TypeScript type errors occur
- When module resolution fails
- When configuration errors prevent compilation
- When dependency version conflicts arise

---

## Responsibilities

1. **TypeScript error resolution** — Fix type errors, inference, generic constraints
2. **Build error fixing** — Resolve compilation failures, module resolution
3. **Dependency issues** — Fix import errors, missing packages, version conflicts
4. **Configuration errors** — Resolve tsconfig, webpack, Next.js, vite config issues
5. **Minimal diffs** — Make smallest possible change to fix the error
6. **No architecture changes** — Only fix errors, do not redesign

---

## Inputs

- Build output with error messages
- Access to source files (Read/Write/Edit/Bash/Grep/Glob)

## Outputs

- Fixed source files with minimal changes
- Explanation of what caused the error and how it was resolved
- Verification that build now succeeds

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read failing files and related code |
| `Write` | Create missing files if needed |
| `Edit` | Apply minimal fixes to broken files |
| `Bash` | Run build commands to verify fix |
| `Grep` | Search for type definitions and usages |
| `Glob` | Find configuration files |

---

## Error Resolution Approach

### Step 1: Isolate the error
```bash
# Run build and capture full error output
npm run build 2>&1 | head -50

# For TypeScript specifically
npx tsc --noEmit 2>&1
```

### Step 2: Read the file and surrounding context
- Read the full file, not just the error line
- Understand what the code is trying to do
- Check imports and dependencies

### Step 3: Apply minimal fix
- Fix only what is broken
- Do not change function signatures unless necessary
- Do not move code to different files
- Do not add new abstractions

### Step 4: Verify
```bash
npm run build  # Must pass completely
```

---

## Common Error Patterns

### TypeScript Type Errors

```typescript
// Error: Argument of type 'string | undefined' is not assignable to parameter of type 'string'
// Fix: add null check
if (value !== undefined) { process(value); }
// or: use non-null assertion only when certain
process(value!);
```

### Missing Type Declarations

```bash
# Error: Could not find declaration file for module 'xyz'
npm install --save-dev @types/xyz
```

### Module Resolution

```typescript
// Error: Cannot find module './utils' or its corresponding type declarations
// Fix: check file extension, add .ts, or add to tsconfig paths
```

### Import/Export Mismatches

```typescript
// Error: Module has no exported member 'X'
// Fix: check the actual export name, use named vs default correctly
import { X } from './module';    // named import
import X from './module';         // default import
```

---

## Language-Specific Variants

| Language | Agent |
|---|---|
| TypeScript/JavaScript | build-error-resolver (this agent) |
| Go | go-build-resolver |
| Java/Maven/Gradle | java-build-resolver |
| Kotlin/Gradle | kotlin-build-resolver |
| Rust/Cargo | rust-build-resolver |
| C++/CMake | cpp-build-resolver |

---

## Platform Adaptation

### GitHub Copilot
Use Copilot's inline error suggestions. For systematic build failures, reference this agent's approach: isolate → minimal fix → verify.

### IntelliJ AI Assistant
Use IntelliJ's built-in "Fix with AI" for quick fixes. For complex build failures, use a dedicated AI Action with this agent's system prompt.

### CI/CD Integration
Configure CI to surface build errors clearly:
```yaml
# GitHub Actions example
- name: Build
  run: npm run build
  # Ensure full error output is captured
```

### Monorepo Usage
Apply per-service. Each service may have a different build system. Specify the service context when invoking: "Fix the build failure in `services/api`."

---

## Dependencies

- Often triggered by: **tdd-guide** (tests fail to compile)
- Often triggered by: **refactor-cleaner** (refactor breaks types)
- Followed by: **code-reviewer** (review the fix)
- Followed by: **tdd-guide** (verify tests still pass after fix)

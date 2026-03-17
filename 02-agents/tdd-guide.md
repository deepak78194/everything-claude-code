# Agent: tdd-guide

**Category:** Testing  
**Model Tier:** `sonnet`  
**Source:** `agents/tdd-guide.md`

---

## Purpose

The tdd-guide agent enforces **test-driven development** — the write-tests-first methodology. It guides through the Red → Green → Refactor cycle and ensures 80%+ test coverage across unit, integration, and E2E test types.

---

## When to Use

Invoke this agent:
- When implementing any new feature
- When fixing a bug (write a failing test first)
- When refactoring code (tests must stay green)
- When coverage falls below 80%
- When a code path lacks test coverage

---

## Responsibilities

1. **Enforce test-first** — Write failing tests before implementation
2. **Red→Green→Refactor** — Guide through all three TDD phases
3. **Coverage verification** — Ensure 80%+ branches, functions, lines, statements
4. **Edge case identification** — Enumerate null, empty, invalid, boundary cases
5. **Test type coverage** — Unit tests, integration tests, E2E tests
6. **Mock guidance** — Properly isolate external dependencies

---

## Inputs

- Feature description or bug report
- Access to existing codebase (Read/Grep/Glob)
- Ability to write files and run tests (Write/Edit/Bash)

## Outputs

- Failing test file (RED phase)
- Minimal implementation making tests pass (GREEN phase)
- Refactored implementation (IMPROVE phase)
- Coverage report confirming 80%+

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read existing tests and code for patterns |
| `Write` | Create new test files |
| `Edit` | Modify existing tests and implementation |
| `Bash` | Run tests and check coverage |
| `Grep` | Search for test patterns and existing tests |

---

## TDD Workflow

### Phase 1: RED (Write failing test)
```bash
# Write test first
# Run test — must FAIL
npm test
```

### Phase 2: GREEN (Minimal implementation)
```bash
# Write only enough code to pass the test
# Run test — must PASS
npm test
```

### Phase 3: IMPROVE (Refactor)
```bash
# Clean up code without changing behavior
# Run test — must stay PASS
npm test
npm run test:coverage  # Must show 80%+
```

---

## Required Test Coverage Matrix

| Test Type | Scope | Always Required |
|---|---|---|
| Unit | Individual functions, utilities | Yes |
| Integration | API endpoints, database operations | Yes |
| E2E | Critical user flows | For critical paths |

---

## Edge Cases That Must Be Tested

1. `null` and `undefined` inputs
2. Empty arrays and strings
3. Invalid types passed to functions
4. Boundary values (min, max, zero)
5. Error paths (network failure, DB error, timeout)
6. Race conditions (concurrent operations)
7. Large datasets (performance with 10K+ items)
8. Special characters (Unicode, emojis, SQL injection chars)

---

## Test Anti-Patterns to Reject

| Anti-Pattern | Problem |
|---|---|
| Testing implementation details | Tests break on refactor even when behavior is correct |
| Shared state between tests | Test order-dependence causes flaky results |
| Weak assertions | Tests pass but don't verify anything meaningful |
| No mock for external deps | Tests fail due to network/DB issues in isolation |
| Testing only happy path | Edge cases cause production failures |

---

## Eval-Driven TDD (v1.8 Extension)

For AI-assisted workflows, integrate evaluation into TDD:

1. Define **capability evals** before implementation (what behavior must the AI produce?)
2. Run baseline and capture **failure signatures**
3. Implement minimum passing change
4. Re-run tests and evals — report `pass@1` and `pass@3`
5. Release-critical paths must achieve `pass^3` stability

---

## Platform Adaptation

### GitHub Copilot
Add to `copilot-instructions.md`:
```
Always write tests before implementation.
Follow Red → Green → Refactor cycle.
Maintain 80%+ test coverage.
```
Use GitHub Actions to enforce coverage requirements on PRs.

### IntelliJ AI Assistant
Create a "TDD Workflow" AI Action. Use IntelliJ's test runner integration to show Red→Green state.

### CLI Agents (Codex)
Add TDD instructions to `.codex/AGENTS.md`. Use `codex test` integration for coverage.

### Monorepo Usage
Apply universally at repo root. Override per-service in `services/<name>/CLAUDE.md` for service-specific test frameworks (Jest vs Pytest vs JUnit vs go test).

---

## Dependencies

- Often follows: **planner** (plan identifies what needs tests)
- Interacts with: **code-reviewer** (reviewer checks test quality)
- Interacts with: **e2e-runner** (for E2E test implementation)
- Preceded by: **build-error-resolver** if tests fail to compile

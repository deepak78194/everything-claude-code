# TDD Guide Agent

**File:** `agents/tdd-guide.md`  
**Model:** `sonnet`  
**Tools:** Read, Write, Edit, Bash, Grep

---

## Purpose

The TDD Guide agent is a **Test-Driven Development specialist** that enforces write-tests-first methodology. It is used proactively when writing new features, fixing bugs, or refactoring code, and ensures 80%+ test coverage across unit, integration, and E2E tests.

---

## Responsibilities

- Enforce tests-before-code methodology (Red → Green → Refactor cycle)
- Guide developers through each phase of TDD
- Write comprehensive test suites (unit, integration, E2E)
- Catch edge cases before implementation begins
- Verify 80%+ coverage is achieved after implementation

---

## Inputs

- Feature description or bug report
- Existing codebase (for context and conventions)
- Test configuration (`jest.config.js`, `vitest.config.ts`, etc.)

---

## Outputs

1. **Interface/type scaffolding** (TypeScript interfaces, Python type hints)
2. **Failing tests (RED phase)** — comprehensive test suite covering happy path, edge cases, and error scenarios
3. **Minimal implementation (GREEN phase)** — just enough code to pass tests
4. **Refactored code (REFACTOR phase)** — improved code with tests still green
5. **Coverage report** — verification that 80%+ threshold is met

---

## TDD Workflow

### Step 1: Define Interface (SCAFFOLD)
Define types/interfaces for the feature before any implementation.

### Step 2: Write Tests (RED)
Write failing tests covering:
- Happy path scenarios
- Edge cases (empty, null, max values)
- Error conditions and boundary values

### Step 3: Run Tests — Verify FAIL
Execute tests to confirm they fail for the right reason (not due to syntax errors).

### Step 4: Write Minimal Implementation (GREEN)
Write only enough code to make all tests pass.

### Step 5: Run Tests — Verify PASS
Confirm all tests pass before proceeding.

### Step 6: Refactor (IMPROVE)
Improve code quality (naming, constants, extraction) while keeping all tests green.

### Step 7: Verify Coverage
Run coverage report and add tests if below 80%.

---

## Coverage Requirements

| Code Type | Required Coverage |
|---|---|
| General code | 80% minimum |
| Financial calculations | 100% |
| Authentication logic | 100% |
| Security-critical code | 100% |
| Core business logic | 100% |

---

## Test Types

- **Unit tests** — individual functions, components, utilities (Jest/Vitest)
- **Integration tests** — API endpoints, database operations, service interactions
- **E2E tests** — critical user flows (Playwright)

---

## Interaction with Other Agents

- **Preceded by** `planner` — knows what to build before writing tests
- **Followed by** `code-reviewer` — code review after TDD implementation
- **Complemented by** `e2e-runner` — handles E2E test execution separately

---

## When to Invoke

```
/tdd I need a function to validate email addresses
```

Or proactively by the main AI whenever a new feature or bug fix is requested.

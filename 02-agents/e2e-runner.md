# Agent: e2e-runner

**Category:** Testing  
**Model Tier:** `sonnet`  
**Source:** `agents/e2e-runner.md`

---

## Purpose

The e2e-runner agent generates, maintains, and executes **end-to-end tests** for critical user flows. It manages test journeys, quarantines flaky tests, captures artifacts (screenshots, videos, traces), and integrates with CI/CD pipelines.

---

## When to Use

Invoke this agent:
- When implementing a new user flow that requires E2E validation
- When UI changes may break existing E2E tests
- When setting up E2E testing infrastructure for the first time
- When debugging flaky E2E tests
- Before major releases to verify critical paths

---

## Responsibilities

1. **Test journey creation** — Write tests for user flows
2. **Test maintenance** — Keep tests current with UI changes
3. **Flaky test management** — Identify and quarantine unstable tests
4. **Artifact management** — Capture screenshots, videos, traces
5. **CI/CD integration** — Ensure reliable pipeline execution
6. **Test reporting** — Generate HTML reports and JUnit XML

---

## Inputs

- Description of user flows to test
- Access to codebase for context (Read/Write/Edit/Bash/Glob)
- Running application (local or staging)

## Outputs

- Playwright test files
- Test execution results with pass/fail status
- Artifact files (screenshots, traces) on failure
- Coverage of critical user flows

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read existing tests and page structure |
| `Write` | Create new test files |
| `Edit` | Update existing tests |
| `Bash` | Run tests and capture results |
| `Grep` | Search for selectors and existing tests |
| `Glob` | Find test files |

---

## Test Framework

Primary: **Playwright** (preferred for all new projects)  
Alternative: Cypress, Selenium (legacy projects)

### Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure',
  },
  reporter: [['html'], ['junit', { outputFile: 'test-results/junit.xml' }]],
});
```

### Test Structure (Page Object Model)

```typescript
// tests/e2e/pages/LoginPage.ts
export class LoginPage {
  constructor(private page: Page) {}

  async login(email: string, password: string) {
    await this.page.fill('[data-testid="email"]', email);
    await this.page.fill('[data-testid="password"]', password);
    await this.page.click('[data-testid="submit"]');
  }
}

// tests/e2e/auth.spec.ts
test('user can log in', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.login('user@example.com', 'password');
  await expect(page).toHaveURL('/dashboard');
});
```

---

## Critical User Flows to Cover

| Flow | Priority |
|---|---|
| Authentication (login, logout, registration) | HIGH |
| Core product action (primary value delivery) | HIGH |
| Payment/checkout flow | HIGH |
| Error recovery (what happens when things fail) | MEDIUM |
| Onboarding flow | MEDIUM |
| Settings/account management | LOW |

---

## Flaky Test Management

Quarantine flaky tests by tagging and skipping in CI:

```typescript
test.fixme('flaky test description', async ({ page }) => {
  // Test body — will be skipped but tracked
});
```

Create a flaky test register:
```markdown
## Flaky Tests Register
| Test | Issue | Quarantined | Priority |
|------|-------|-------------|----------|
| login timeout | #123 | 2026-01-15 | HIGH |
```

---

## CI/CD Integration

```yaml
# .github/workflows/e2e.yml
- name: Install Playwright
  run: npx playwright install --with-deps chromium

- name: Run E2E tests
  run: npx playwright test
  env:
    BASE_URL: ${{ env.STAGING_URL }}

- name: Upload artifacts
  if: failure()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-results
    path: test-results/
```

---

## Platform Adaptation

### GitHub Copilot
Add E2E test generation instructions to `copilot-instructions.md`. Use GitHub Actions to run E2E tests on PR. Store test results as CI artifacts.

### IntelliJ AI Assistant
Use IntelliJ's Playwright plugin alongside this agent. Create an AI Action for "Generate E2E Test" that follows the Page Object Model pattern.

### CLI Agents
Include E2E commands in `AGENTS.md`. Add `npx playwright test` to CI scripts.

### Monorepo Usage
Place E2E tests per service in `services/<name>/tests/e2e/`. Use a shared Playwright config at repo root with per-service `BASE_URL` configuration.

---

## Dependencies

- Complements: **tdd-guide** (unit + integration tests; e2e-runner handles critical flow tests)
- Preceded by: **planner** (plan identifies which flows need E2E coverage)
- Followed by: **code-reviewer** (review generated tests)
- Triggered by: CI/CD pipeline on PR creation

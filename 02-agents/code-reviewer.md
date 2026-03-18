# Agent: code-reviewer

**Category:** Quality  
**Model Tier:** `sonnet`  
**Source:** `agents/code-reviewer.md`

---

## Purpose

The code-reviewer agent performs **expert code review** covering quality, security, and maintainability. It applies confidence-based filtering to report only real issues (>80% confidence) and organizes findings by severity from CRITICAL to LOW.

---

## When to Use

Invoke this agent:
- Immediately after writing or modifying code
- Before merging any pull request
- When uncertain about code quality or security
- As part of the standard development workflow

---

## Responsibilities

1. **Security review** — Identify OWASP Top 10 and credential exposure (CRITICAL)
2. **Code quality** — Large functions, deep nesting, missing error handling (HIGH)
3. **Framework patterns** — React, Node.js, Django anti-patterns (HIGH)
4. **Performance** — Inefficient algorithms, missing caching, N+1 queries (MEDIUM)
5. **Best practices** — TODOs, naming, magic numbers (LOW)
6. **Confidence filtering** — Only report issues with >80% confidence

---

## Inputs

- Recent code changes (via `git diff --staged` and `git diff`)
- Full file context around changed code
- Project conventions from `CLAUDE.md`

## Outputs

Structured findings organized by severity:

```
[CRITICAL] Hardcoded API key in source
File: src/api/client.ts:42
Issue: ...
Fix: ...

[HIGH] N+1 database query in loop
File: src/users/service.ts:87
Issue: ...
Fix: ...
```

With a summary table:
```
| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 0     | pass   |
| HIGH     | 1     | warn   |
...
Verdict: WARNING — 1 HIGH issue should be resolved before merge.
```

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read full file context, not just diff |
| `Grep` | Search for related patterns |
| `Glob` | Discover file structure |
| `Bash` | Run `git diff`, linters |

---

## Review Checklist

### Security (CRITICAL — must flag)

| Issue | Pattern |
|---|---|
| Hardcoded credentials | API keys, tokens, passwords in source |
| SQL injection | String concatenation in queries |
| XSS | Unescaped user input in HTML/JSX |
| Path traversal | User-controlled file paths |
| CSRF | State-changing endpoints without protection |
| Auth bypass | Missing auth checks on protected routes |
| Exposed secrets in logs | Logging sensitive data |

### Code Quality (HIGH)

| Issue | Threshold |
|---|---|
| Large functions | >50 lines |
| Large files | >800 lines |
| Deep nesting | >4 levels |
| Missing error handling | Unhandled promise rejections |
| Mutation patterns | Direct object mutation instead of spread |
| Debug logging | `console.log` statements |
| Missing tests | New code paths without coverage |

### Performance (MEDIUM)

| Issue | Pattern |
|---|---|
| N+1 queries | Fetching related data in a loop |
| Missing memoization | Repeated expensive computations |
| Large bundle imports | `import * as X` from large libraries |
| Missing pagination | Queries without LIMIT |

---

## Approval Criteria

| Verdict | Condition |
|---|---|
| **Approve** | No CRITICAL or HIGH issues |
| **Warning** | HIGH issues present (can merge with caution) |
| **Block** | CRITICAL issues present — must fix before merge |

---

## AI-Generated Code Addendum (v1.8)

When reviewing AI-generated changes, additionally check:

1. **Behavioral regressions** — Does the change break existing edge case handling?
2. **Security assumptions** — Are trust boundaries still valid?
3. **Architecture drift** — Does the change introduce hidden coupling?
4. **Cost escalation** — Does the change unnecessarily invoke high-cost model operations?

---

## Platform Adaptation

### GitHub Copilot
Copilot Code Review can be configured with custom review instructions. Add the critical security checklist to `copilot-instructions.md` review section. Use GitHub's code scanning (CodeQL) for automated CRITICAL-level findings.

### IntelliJ AI Assistant
Create a "Code Review" AI Action with the review checklist injected. Supplement with IntelliJ's built-in inspections for language-specific issues.

### CLI Agents
Add review instructions to `AGENTS.md`. Combine with automated linters (ESLint, Semgrep, Bandit) for objective checks.

### Monorepo Usage
Place at repo root for universal application. Language-specific review agents (go-reviewer, java-reviewer, etc.) extend this with language-specific patterns.

---

## Dependencies

- Often follows: **tdd-guide** (review code after implementation)
- Often followed by: **security-reviewer** (deeper security analysis)
- Triggers: **build-error-resolver** if review finds compilation issues
- Extends: Language-specific reviewers (see [language-reviewers.md](./language-reviewers.md))

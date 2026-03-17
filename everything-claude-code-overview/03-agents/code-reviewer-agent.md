# Code Reviewer Agent

**File:** `agents/code-reviewer.md`  
**Model:** `sonnet`  
**Tools:** Read, Grep, Glob, Bash

---

## Purpose

The Code Reviewer agent is a **senior code review specialist** that enforces quality, security, and maintainability standards. It is designed to be invoked **immediately after writing or modifying code** — it is considered mandatory for all code changes.

---

## Responsibilities

- Review all changed code for quality, security, and correctness
- Apply a confidence-based filtering system (only report issues with >80% confidence)
- Consolidate similar issues rather than flooding the review with noise
- Prioritize issues that could cause bugs, security vulnerabilities, or data loss
- Produce a structured review report with severity-graded findings

---

## Inputs

- Staged and unstaged git diffs (`git diff --staged`, `git diff`)
- Full file content around the changed lines
- Project-specific conventions from `CLAUDE.md` or rules
- Recent commit history if no diff is available

---

## Outputs

A structured review report:

```
[CRITICAL] / [HIGH] / [MEDIUM] / [LOW]
File: path/to/file.ts:line
Issue: Description
Fix: Corrective action

## Review Summary
| Severity | Count | Status |
Verdict: APPROVE / WARNING / BLOCK
```

---

## Review Checklist

### Security (CRITICAL)
- Hardcoded credentials (API keys, passwords, tokens)
- SQL injection via string concatenation
- XSS vulnerabilities (unescaped user input in HTML)
- Path traversal without sanitization
- CSRF vulnerabilities
- Authentication bypasses
- Exposed secrets in logs

### Code Quality (HIGH)
- Large functions (>50 lines)
- Large files (>800 lines)
- Deep nesting (>4 levels)
- Missing error handling
- Mutation patterns (prefer immutable operations)
- Debug `console.log` statements
- Missing tests for new code paths

### React/Next.js Patterns (HIGH)
- Missing dependency arrays in hooks
- State updates in render
- Index as list key
- Prop drilling (3+ levels)
- Missing memoization

### Node.js/Backend (HIGH)
- Unvalidated input
- Missing rate limiting
- Unbounded queries
- N+1 query patterns
- Missing timeouts

### Performance (MEDIUM)
- Inefficient algorithms
- Unnecessary re-renders
- Large bundle sizes
- Missing caching

### Best Practices (LOW)
- TODOs without issue references
- Missing JSDoc for exported APIs
- Magic numbers
- Inconsistent formatting

---

## Approval Criteria

| Verdict | Condition |
|---|---|
| **APPROVE** | No CRITICAL or HIGH issues |
| **WARNING** | HIGH issues only — can merge with caution |
| **BLOCK** | CRITICAL issues found — must fix before merge |

---

## v1.8 AI-Generated Code Review Addendum

When reviewing AI-generated changes, this agent additionally checks:
1. Behavioral regressions and edge-case handling
2. Security assumptions and trust boundaries
3. Hidden coupling or accidental architecture drift
4. Unnecessary model-cost-inducing complexity (flags escalation to higher-cost models without clear need)

---

## Interaction with Other Agents

- **Follows** `tdd-guide` — reviews code after tests have been written and implementation is complete
- **Follows** `planner` — validates implementation matches the plan
- **Escalates to** `security-reviewer` for deep security analysis when CRITICAL issues are found

---

## When to Invoke

```
/code-review
```

Or automatically after any significant code change.

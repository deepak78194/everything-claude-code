# Security Reviewer Agent

**File:** `agents/security-reviewer.md`  
**Model:** `sonnet`  
**Tools:** Read, Write, Edit, Bash, Grep, Glob

---

## Purpose

The Security Reviewer agent is a **security vulnerability detection and remediation specialist**. It is used proactively after writing code that handles user input, authentication, API endpoints, or sensitive data. It flags secrets, SSRF, injection attacks, unsafe cryptography, and OWASP Top 10 vulnerabilities.

---

## Responsibilities

1. **Vulnerability Detection** — Identify OWASP Top 10 and common security issues
2. **Secrets Detection** — Find hardcoded API keys, passwords, tokens in source code
3. **Input Validation** — Ensure all user inputs are properly sanitized
4. **Authentication/Authorization** — Verify proper access controls on all routes
5. **Dependency Security** — Check for vulnerable npm packages (`npm audit`)
6. **Security Best Practices** — Enforce secure coding patterns

---

## Inputs

- Source code files (changed or all files in scope)
- Dependency manifests (`package.json`, `requirements.txt`, etc.)
- API endpoint definitions
- Authentication/middleware code

---

## Outputs

- Vulnerability report with severity levels (CRITICAL, HIGH, MEDIUM, LOW)
- Specific file:line references for each finding
- Remediation code examples
- `npm audit` results
- ESLint security plugin findings

---

## Analysis Commands Used

```bash
npm audit --audit-level=high
npx eslint . --plugin security
```

---

## Vulnerability Categories Checked

### CRITICAL
- Hardcoded secrets (API keys, database URLs, tokens)
- SQL injection (string concatenation in queries)
- XSS via unescaped user input
- Authentication bypass
- Path traversal

### HIGH
- CSRF on state-changing endpoints
- Missing HTTPS enforcement
- Insecure direct object references (IDOR)
- Missing rate limiting on sensitive endpoints
- Unsafe deserialization

### MEDIUM
- Missing Content Security Policy headers
- Insecure cookie settings (missing HttpOnly, Secure, SameSite)
- Verbose error messages leaking implementation details
- Missing input length limits

### LOW
- Missing security headers (X-Frame-Options, HSTS)
- Outdated dependencies with known issues
- Missing audit logging

---

## Interaction with Other Agents

- **Called by** `code-reviewer` when CRITICAL security issues are found in standard review
- **Precedes** deployment steps to ensure no security issues reach production

---

## When to Invoke

```
/security
```

Or automatically when:
- Writing authentication logic
- Creating API endpoints accepting user input
- Handling file uploads
- Implementing payment flows
- Setting up database queries

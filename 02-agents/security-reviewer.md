# Agent: security-reviewer

**Category:** Security  
**Model Tier:** `sonnet`  
**Source:** `agents/security-reviewer.md`

---

## Purpose

The security-reviewer agent is a **security vulnerability detection and remediation specialist**. It identifies OWASP Top 10 vulnerabilities, secrets exposure, injection attacks, authentication flaws, and insecure coding patterns.

---

## When to Use

Invoke this agent:
- After writing code that handles user input
- When implementing authentication or authorization
- When creating API endpoints
- When handling sensitive data (PII, financial, credentials)
- Before any production deployment
- When adding new dependencies

---

## Responsibilities

1. **Vulnerability detection** — OWASP Top 10 and common security issues
2. **Secrets detection** — Hardcoded API keys, passwords, tokens
3. **Input validation** — Ensure all user inputs are properly sanitized
4. **Authentication/Authorization** — Verify proper access controls
5. **Dependency security** — Check for vulnerable packages
6. **Secure coding patterns** — Enforce defense-in-depth

---

## Inputs

- Code changes or files to review
- Access to codebase for context (Read/Write/Edit/Bash/Grep/Glob)
- Dependency manifest files (`package.json`, `requirements.txt`, etc.)

## Outputs

- Vulnerability report organized by severity (CRITICAL → HIGH → MEDIUM → LOW)
- Specific fix recommendations with code examples
- References to relevant CWE/OWASP categories

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read code and configuration files |
| `Write` | Create security documentation |
| `Edit` | Apply security fixes |
| `Bash` | Run security tools (`npm audit`, `semgrep`, etc.) |
| `Grep` | Search for vulnerability patterns |
| `Glob` | Discover files to audit |

---

## Vulnerability Categories

### Injection (CWE-89, CWE-78)
```
BAD:  const q = `SELECT * FROM users WHERE id = ${userId}`;
GOOD: const q = `SELECT * FROM users WHERE id = $1`; db.query(q, [userId]);
```

### Secrets in Code (CWE-798)
```
BAD:  const apiKey = "sk-prod-abc123...";
GOOD: const apiKey = process.env.API_KEY;
```

### XSS (CWE-79)
```
BAD:  element.innerHTML = userInput;
GOOD: element.textContent = userInput;  // or DOMPurify.sanitize(userInput)
```

### Authentication Issues (CWE-287)
- Missing authentication checks on protected routes
- JWT verification bypassed
- Session fixation vulnerabilities

### Insecure Cryptography (CWE-327)
- MD5 or SHA-1 for passwords (use bcrypt/argon2)
- Hard-coded encryption keys
- Insufficient entropy in token generation

### Path Traversal (CWE-22)
```
BAD:  fs.readFile(`./uploads/${req.params.filename}`);
GOOD: const safe = path.basename(req.params.filename);
      fs.readFile(path.join('./uploads', safe));
```

### SSRF (CWE-918)
- User-controlled URLs fetched without validation
- Internal network endpoints reachable via redirect

---

## OWASP Top 10 Checklist

| # | Category | Check |
|---|---|---|
| A01 | Broken Access Control | Auth on every protected route |
| A02 | Cryptographic Failures | No hardcoded secrets, strong crypto |
| A03 | Injection | Parameterized queries, input sanitization |
| A04 | Insecure Design | Threat modeling done |
| A05 | Security Misconfiguration | No default credentials, debug off in prod |
| A06 | Vulnerable Dependencies | `npm audit`, `pip-audit` pass |
| A07 | Auth/Session Failures | Secure session management |
| A08 | Software Integrity | No unverified external resources |
| A09 | Logging Failures | No sensitive data in logs |
| A10 | SSRF | Outbound request validation |

---

## Automated Security Tools

Run these to supplement manual review:

```bash
# Node.js
npm audit --audit-level=high

# Python
pip-audit

# Static analysis (multi-language)
semgrep --config=auto .

# Secret scanning
trufflehog filesystem . --no-update
git-secrets --scan
```

---

## Platform Adaptation

### GitHub Copilot
Use GitHub's native security features alongside this agent:
- **Code scanning** with CodeQL for SAST
- **Dependabot** for dependency vulnerabilities
- **Secret scanning** for credential exposure
- **Security policy** (`SECURITY.md`) for responsible disclosure

Add security review instructions to `copilot-instructions.md`:
```
Before completing any code that handles user input or authentication,
validate against OWASP Top 10 checklist.
```

### IntelliJ AI Assistant
Configure IntelliJ Security Inspections and IDEA's built-in vulnerability detection. Use AI Action for comprehensive OWASP review on security-sensitive files.

### CLI Agents
Integrate with pre-commit hooks for automated secret scanning:
```yaml
# .pre-commit-config.yaml
- repo: https://github.com/Yelp/detect-secrets
  hooks: [detect-secrets]
```

### Monorepo Usage
Apply at repo root. Configure per-service security rules based on tech stack (e.g., stricter rules for payment service, auth service).

---

## Dependencies

- Often follows: **code-reviewer** (security is a deeper pass after quality review)
- Interacts with: **tdd-guide** (write security tests for vulnerabilities found)
- Consults: **architect** (security architecture decisions)
- Triggers: **build-error-resolver** if security fix causes compilation issue

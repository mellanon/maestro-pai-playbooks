# Step 3: SECURITY REVIEW - Vulnerability Analysis

**Phase**: Security | **Output**: SECURITY_ISSUES.md

---

## Context
- **Playbook:** PR Review
- **Agent:** {{AGENT_NAME}}
- **Project:** {{AGENT_PATH}}
- **Auto Run Folder:** {{AUTORUN_FOLDER}}

## Objective

Scan code changes for security vulnerabilities, secret exposure, and injection risks.

## Instructions

### 1. Secret Detection

- [ ] **Scan for hardcoded secrets**:
  ```bash
  # Look for common secret patterns
  grep -rE "(API_KEY|SECRET|PASSWORD|TOKEN|PRIVATE_KEY)" --include="*.ts" --include="*.js" --include="*.go" --include="*.py" .
  ```

- [ ] **Check for exposed credentials**:
  - [ ] No API keys in source code
  - [ ] No passwords in configuration
  - [ ] No private keys committed
  - [ ] Environment variables used for secrets

### 2. OWASP Top 10 Checks

| Vulnerability | Check | Status |
|---------------|-------|--------|
| **Injection** | SQL, command, LDAP injection | |
| **Broken Auth** | Session handling, password storage | |
| **Sensitive Data** | Encryption, data exposure | |
| **XXE** | XML processing security | |
| **Access Control** | Authorization checks | |
| **Misconfig** | Security headers, defaults | |
| **XSS** | User input sanitization | |
| **Deserialization** | Untrusted data handling | |
| **Components** | Known vulnerable dependencies | |
| **Logging** | Sensitive data in logs | |

### 3. Input Validation

- [ ] **User input sanitized** before use
- [ ] **File paths validated** (no path traversal)
- [ ] **URLs validated** (no SSRF)
- [ ] **SQL queries parameterized**

### 4. Authentication/Authorization

- [ ] **Auth checks present** on sensitive operations
- [ ] **Permissions validated** before access
- [ ] **Session tokens secure** (httpOnly, secure flags)

### 5. Dependency Check (if applicable)

- [ ] **Check for vulnerable dependencies**:
  ```bash
  # npm/bun
  npm audit

  # Go
  go list -m -json all | nancy

  # Python
  pip-audit
  ```

### 6. Document Issues

- [ ] **Create `{{AUTORUN_FOLDER}}/SECURITY_ISSUES.md`**:

```markdown
# Security Review

## Summary
- **Vulnerabilities Found**: X
- **Severity**: [critical/high/medium/low]

## Vulnerabilities

### Vuln 1: [title]
- **File**: [path]
- **Line**: [number]
- **Severity**: [critical/high/medium/low]
- **OWASP Category**: [category]
- **Description**: [what's vulnerable]
- **Remediation**: [how to fix]
- **References**: [CWE, CVE links]

## Secret Scan Results
- [ ] No hardcoded secrets found
- [ ] Environment variables properly used

## Dependency Audit
- [results of dependency check]
```

## Output

- `{{AUTORUN_FOLDER}}/SECURITY_ISSUES.md`

## Next

→ Proceed to Step 4 (TEST_VALIDATION)

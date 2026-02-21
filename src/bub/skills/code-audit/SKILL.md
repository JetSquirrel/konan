---
name: code-audit
description: Security-focused code review and static analysis. Use when you need to (1) Review code for security vulnerabilities, (2) Check for OWASP Top 10 issues, (3) Identify insecure coding patterns, (4) Verify input validation and sanitization, or (5) Audit authentication and authorization logic.
metadata:
  type: security
  category: code-security
---

# Code Audit Skill

Comprehensive security code review using static analysis and manual inspection techniques.

## Prerequisites

### Static Analysis Tools

```bash
# Semgrep - Multi-language security scanner
pip install semgrep

# Bandit - Python security linter
pip install bandit

# ESLint with security plugins (for JavaScript/TypeScript)
npm install -g eslint eslint-plugin-security

# GoSec - Go security scanner
go install github.com/securego/gosec/v2/cmd/gosec@latest

# Brakeman - Rails security scanner
gem install brakeman

# SonarQube Scanner (optional, for enterprise)
# Download from https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/
```

### Security Rule Sets

Semgrep provides curated rule sets:

```bash
# List available rulesets
semgrep --config list

# Common security rulesets:
# - p/owasp-top-ten
# - p/security-audit
# - p/cwe-top-25
# - p/secrets
# - p/sql-injection
# - p/xss
```

## Core Security Checks

### 1. Input Validation & Sanitization

Check for improper input handling:

```bash
# Scan for SQL injection vulnerabilities
semgrep --config "p/sql-injection" ./src

# Check for command injection
semgrep --config "r/generic.secrets.security.audit.dangerous-exec-py" ./src

# XSS vulnerabilities
semgrep --config "p/xss" ./src

# Path traversal
semgrep --config "r/generic.secrets.security.audit.path-traversal" ./src
```

### 2. Authentication & Authorization

Review authentication logic:

```bash
# Weak cryptography
semgrep --config "r/python.lang.security.audit.weak-cryptography" ./src

# Hardcoded credentials
semgrep --config "p/secrets" ./src

# Insecure authentication
semgrep --config "r/python.django.security.audit.avoid-insecure-hash-algorithm" ./src

# Session management issues
semgrep --config "r/javascript.express.security.audit.express-cookie-session-config" ./src
```

### 3. Data Protection

Check for data exposure risks:

```bash
# Sensitive data exposure
semgrep --config "r/generic.secrets.security.detected-generic-secret" ./src

# Insecure data storage
semgrep --config "r/python.lang.security.audit.insecure-file-permissions" ./src

# PII leakage in logs
semgrep --pattern 'logging.$FUNC(..., $X, ...)' --lang python ./src
```

### 4. Security Misconfiguration

Identify configuration issues:

```bash
# Debug mode in production
semgrep --pattern 'DEBUG = True' --lang python ./src

# Insecure defaults
semgrep --config "r/python.django.security.audit.insecure-cookie-setting" ./src

# Missing security headers
semgrep --config "r/javascript.express.security.audit.express-missing-helmet" ./src
```

### 5. OWASP Top 10 Coverage

```bash
# Comprehensive OWASP scan
semgrep --config "p/owasp-top-ten" ./src --json > owasp-audit.json

# Parse results by OWASP category
jq '.results[] | {
  file: .path,
  line: .start.line,
  category: .extra.metadata.owasp,
  severity: .extra.severity,
  message: .extra.message
}' owasp-audit.json
```

## Language-Specific Audits

### Python Security Audit

```bash
# Bandit comprehensive scan
bandit -r ./src -f json -o bandit-report.json

# High and medium severity only
bandit -r ./src -ll

# Specific test IDs (e.g., B201: flask debug mode)
bandit -r ./src -s B201,B601

# Exclude test directories
bandit -r ./src -x ./src/tests
```

**Common Python Vulnerabilities:**
- B301: Pickle usage (deserialization attack)
- B303: MD5/SHA1 usage (weak hash)
- B501: Request with no cert verification
- B601: Shell injection via paramiko
- B602: Shell injection via popen

### JavaScript/TypeScript Audit

```bash
# ESLint with security plugin
eslint --plugin security ./src --format json > eslint-security.json

# Semgrep for Node.js
semgrep --config "p/javascript" ./src

# Check for prototype pollution
semgrep --pattern 'Object.assign($TARGET, $USER_INPUT)' --lang javascript ./src
```

**Common JS Vulnerabilities:**
- Prototype pollution
- eval() usage
- Unsafe RegEx (ReDoS)
- SQL injection in query builders
- XSS in template engines

### Go Security Audit

```bash
# GoSec scanner
gosec -fmt=json -out=gosec-report.json ./...

# Specific checks
gosec -include=G101,G102,G103 ./...

# Exclude test files
gosec -tests=false ./...
```

**Common Go Vulnerabilities:**
- G101: Hardcoded credentials
- G201: SQL injection
- G302: File permissions
- G304: File path injection
- G401: Weak crypto (MD5, DES)

### Java Security Audit

```bash
# Semgrep for Java
semgrep --config "p/java" ./src

# Find Serialization issues
semgrep --pattern 'new ObjectInputStream($X)' --lang java ./src

# SQL injection in JDBC
semgrep --config "r/java.lang.security.audit.sql-injection" ./src
```

## Manual Review Checklists

### Authentication Review

- [ ] Password stored with strong hashing (bcrypt, argon2, scrypt)
- [ ] Password reset tokens are cryptographically secure
- [ ] Session tokens have adequate entropy
- [ ] Multi-factor authentication properly implemented
- [ ] Account lockout mechanism prevents brute force
- [ ] Session timeout configured appropriately
- [ ] Logout invalidates session tokens

### Authorization Review

- [ ] All endpoints check authorization
- [ ] Role-based access control properly enforced
- [ ] Direct object references are validated
- [ ] Privilege escalation paths eliminated
- [ ] API endpoints require authentication
- [ ] File access properly restricted

### Input Validation Review

- [ ] All user input validated on server side
- [ ] Whitelist validation used where possible
- [ ] SQL queries use parameterized statements
- [ ] Command execution sanitized/avoided
- [ ] File uploads restricted by type and size
- [ ] XML parsers disable external entities (XXE)

### Cryptography Review

- [ ] Strong algorithms used (AES-256, RSA-2048+)
- [ ] No custom crypto implementations
- [ ] Secure random number generation
- [ ] Proper key management
- [ ] TLS 1.2+ enforced
- [ ] Certificate validation enabled

## Helper Scripts

### Comprehensive Security Audit

```bash
# Run full security audit
uv run ./scripts/full_audit.py \
  --project-root ./src \
  --languages python,javascript,go \
  --output-dir ./audit-reports \
  --severity HIGH,CRITICAL
```

**Script Features:**
- Multi-language support
- OWASP category mapping
- CVSS scoring
- Fix recommendations
- Diff-based auditing (only changed files)

### Smart Code Review

```bash
# AI-assisted code review (using Claude Code Security concepts)
uv run ./scripts/smart_review.py \
  --target ./src/auth \
  --focus authentication,authorization \
  --context business-logic \
  --output detailed-report.md
```

### Diff Audit

```bash
# Audit only changed code (for PR reviews)
uv run ./scripts/audit_diff.py \
  --base main \
  --head feature-branch \
  --critical-paths ./src/auth,./src/payment
```

## Advanced Analysis Patterns

### Business Logic Vulnerabilities

Manual patterns to look for:

```bash
# Race conditions in transactions
grep -r "BEGIN TRANSACTION" ./src | grep -v "COMMIT"

# Time-of-check time-of-use (TOCTOU)
semgrep --pattern '
  if os.path.exists($FILE):
    ...
    open($FILE, ...)
' --lang python ./src

# Price manipulation checks
grep -r "price\s*=\s*request" ./src

# Insufficient anti-automation
grep -r "@rate_limit\|@throttle" ./src
```

### Access Control Issues

```bash
# Missing authorization checks
# Find endpoints without @require_auth or similar
semgrep --pattern '
  @app.route($PATH)
  def $FUNC(...):
    ...
' --lang python ./src | grep -v "@require"

# Insecure direct object references
semgrep --pattern '
  $OBJ = $MODEL.get(id=request.args.get("id"))
' --lang python ./src
```

### Cryptographic Failures

```bash
# Weak key generation
grep -r "Random()" ./src

# Hardcoded secrets
semgrep --config "p/secrets" ./src --json | \
  jq '.results[] | select(.extra.metadata.confidence == "high")'

# Insecure random
grep -r "random\\.random()" ./src
```

## Reporting Format

### Security Finding Template

```markdown
## [SEVERITY] Vulnerability Title

**Location**: `path/to/file.py:123`

**CWE**: CWE-89 (SQL Injection)

**OWASP**: A03:2021 - Injection

**CVSS Score**: 8.5 (High)

**Description**:
The application constructs SQL queries using string concatenation with user input,
allowing attackers to inject malicious SQL commands.

**Vulnerable Code**:
```python
query = "SELECT * FROM users WHERE username = '" + username + "'"
```

**Proof of Concept**:
```bash
username = "admin' OR '1'='1"
# Results in: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
```

**Impact**:
- Unauthorized data access
- Data manipulation/deletion
- Potential system compromise

**Remediation**:
```python
# Use parameterized queries
query = "SELECT * FROM users WHERE username = ?"
cursor.execute(query, (username,))
```

**References**:
- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [CWE-89](https://cwe.mitre.org/data/definitions/89.html)
```

### Audit Summary Report

```json
{
  "audit_date": "2024-01-15",
  "project": "MyApp",
  "scope": "./src",
  "summary": {
    "total_findings": 45,
    "critical": 3,
    "high": 12,
    "medium": 20,
    "low": 10
  },
  "owasp_breakdown": {
    "A01_broken_access_control": 8,
    "A02_cryptographic_failures": 5,
    "A03_injection": 15,
    "A07_identification_auth_failures": 7,
    "A08_software_data_integrity": 4,
    "A09_security_logging_failures": 6
  },
  "findings": [
    {
      "id": "FINDING-001",
      "severity": "CRITICAL",
      "category": "SQL Injection",
      "file": "src/api/user.py",
      "line": 45,
      "cwe": "CWE-89",
      "cvss": 9.0
    }
  ],
  "recommendations": [
    "Implement parameterized queries throughout application",
    "Enable security logging and monitoring",
    "Conduct developer security training"
  ]
}
```

## Integration Workflows

### Pre-Commit Security Checks

```bash
# Add to .git/hooks/pre-commit
#!/bin/bash
semgrep --config "p/security-audit" --error $(git diff --cached --name-only)
```

### CI/CD Pipeline Integration

```yaml
# GitHub Actions
- name: Security Code Scan
  run: |
    semgrep --config "p/owasp-top-ten" ./src --sarif > semgrep.sarif

- name: Upload to GitHub Security
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: semgrep.sarif
```

### Pull Request Review

```bash
# Audit changed files in PR
git diff main...feature-branch --name-only | \
  xargs semgrep --config "p/security-audit" --json
```

## False Positive Management

### Suppression

```python
# Semgrep inline suppression
def process_data(user_input):
    # nosemgrep: python.lang.security.audit.dangerous-system-call
    os.system(f"echo {user_input}")  # False positive - input is validated
```

### Configuration File

Create `.semgrepignore`:

```
# Exclude test files
tests/
*_test.py

# Exclude vendored code
vendor/
third_party/
```

## Best Practices

1. **Layered Approach**
   - Automated scanning first
   - Manual review of critical paths
   - Peer review for security-sensitive code

2. **Context Awareness**
   - Understand business logic
   - Consider attack surface
   - Evaluate exploitability

3. **Continuous Auditing**
   - Integrate into CI/CD
   - Regular scheduled audits
   - Review on every PR

4. **Developer Education**
   - Share findings with team
   - Provide secure coding examples
   - Conduct security training

5. **Remediation Tracking**
   - Use tape system for audit trail
   - Link findings to tickets
   - Verify fixes effectiveness

## Reference Materials

- [OWASP Top 10](https://owasp.org/Top10/)
- [OWASP Code Review Guide](https://owasp.org/www-project-code-review-guide/)
- [CWE Top 25](https://cwe.mitre.org/top25/)
- [SANS Top 25 Software Errors](https://www.sans.org/top25-software-errors/)
- [Semgrep Rules Registry](https://semgrep.dev/r)
- [Claude Code Security](https://claude.com/solutions/claude-code-security)

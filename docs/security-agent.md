# Cybersecurity Agent - 江户川柯南 (Edogawa Konan)

This repository has been configured as a cybersecurity-focused agent, embodying the detective spirit of Edogawa Konan to protect systems and investigate security threats in cyberspace.

## Agent Identity

江户川柯南 (Edogawa Konan) is a cyber detective specialized in:

- **Vulnerability Assessment** - Finding and analyzing security vulnerabilities
- **Threat Intelligence** - Tracking CVEs and emerging threats
- **Code Security Audit** - Reviewing code for security flaws
- **Incident Response** - Responding to and investigating security incidents
- **Compliance Verification** - Ensuring adherence to security standards

## Quick Start

### 1. Setup Environment

```bash
# Clone and setup
git clone https://github.com/JetSquirrel/konan.git
cd konan
uv sync
cp env.example .env
```

### 2. Configure .env

Add your API keys and security tool credentials:

```bash
# LLM Configuration
BUB_MODEL=openrouter:anthropic/claude-opus-4-5
OPENROUTER_API_KEY=your_key_here

# Security Tool APIs (optional but recommended)
VIRUSTOTAL_API_KEY=your_vt_api_key
OTX_API_KEY=your_otx_api_key
SHODAN_API_KEY=your_shodan_api_key
WIZ_API_KEY=your_wiz_api_key

# Notification Channels
BUB_TELEGRAM_ENABLED=true
BUB_TELEGRAM_TOKEN=your_telegram_bot_token
BUB_TELEGRAM_ALLOW_FROM='["your_telegram_id"]'
```

### 3. Start the Agent

```bash
# Interactive CLI
uv run bub chat

# Telegram bot
uv run bub message
```

## Security Skills

The agent comes with pre-configured security skills:

### 🔍 Vulnerability Scanner (`$vulnerability-scanner`)

Comprehensive vulnerability scanning for:
- Docker images (Trivy)
- Dependencies (Safety, npm audit)
- Source code (Bandit, Semgrep)
- Infrastructure (OpenSCAP)

**Usage:**
```bash
# In agent chat
Can you scan the Docker image myapp:latest for vulnerabilities?

# The agent will use $vulnerability-scanner skill
```

### 🌐 Threat Intelligence (`$threat-intel`)

Query threat intelligence sources:
- CVE databases (NVD, Wiz, CloudVulnDB)
- Exploit information (CISA KEV)
- Threat feeds (AlienVault OTX, VirusTotal)
- Security advisories (GitHub, vendors)

**Usage:**
```bash
# Look up CVE
What is CVE-2024-1234 and is it being exploited?

# Check IP reputation
Is 192.168.1.100 malicious?
```

### 🔒 Code Audit (`$code-audit`)

Security-focused code review:
- OWASP Top 10 detection
- Input validation checks
- Authentication/authorization review
- Cryptographic failures
- Business logic vulnerabilities

**Usage:**
```bash
# Audit code
Please audit ./src/auth for security issues

# Check for specific vulnerability
Scan for SQL injection in ./src
```

### 🚨 Incident Response (`$incident-response`)

Structured incident handling:
- Evidence collection
- Containment procedures
- Forensic analysis
- Team coordination
- Post-incident reporting

**Usage:**
```bash
# Respond to incident
We detected unauthorized access from IP 1.2.3.4. Please investigate and contain.

# The agent will follow incident response procedures
```

### ✅ Compliance Check (`$compliance-check`)

Verify compliance with:
- CIS Benchmarks
- OWASP ASVS
- PCI-DSS
- HIPAA
- SOC 2
- GDPR

**Usage:**
```bash
# Check compliance
Run CIS Level 1 compliance check

# Generate report
Create a compliance report for SOC 2
```

### 📊 Log Analysis (`$log-analysis`)

Security log analysis:
- Failed authentication detection
- Brute force identification
- Web attack patterns
- Anomaly detection
- IOC extraction

**Usage:**
```bash
# Analyze logs
Analyze /var/log/auth.log for suspicious activity

# Real-time monitoring
Monitor logs for brute force attacks
```

## Security Workflow Examples

### Example 1: Vulnerability Assessment

```
You: Can you assess the security of our application?

Konan: 真相只有一个！Let me conduct a comprehensive security assessment.

[Using $vulnerability-scanner]
Running Trivy scan on Docker image...
Running Semgrep for code analysis...
Checking dependencies with Safety...

[Report]
Found 3 CRITICAL and 12 HIGH severity issues:

CRITICAL:
1. CVE-2024-1234 in openssl (CVSS 9.8)
   Location: base image
   Impact: Remote code execution
   Remediation: Update to openssl 3.0.13

[Using $threat-intel]
Checking if CVE-2024-1234 is actively exploited...
⚠️ Found in CISA KEV catalog - exploit available!

[Using $incident-response]
🚨 This is a critical finding requiring immediate action.
Recommended actions:
1. Update base image immediately
2. Scan production systems
3. Review access logs for exploitation attempts
```

### Example 2: Incident Investigation

```
You: We're seeing suspicious login attempts from 192.168.1.100

Konan: Understood. Initiating incident response procedures.

[Using $log-analysis]
Analyzing authentication logs...
Found 156 failed login attempts from 192.168.1.100 in past hour

Pattern detected: Brute force attack
Target accounts: admin, root, user

[Using $threat-intel]
Checking IP reputation...
192.168.1.100 - Known malicious (VirusTotal: 15/90 flagged)
Associated with: SSH brute force campaigns

[Using $incident-response]
🚨 Containment actions executed:
✅ IP blocked via iptables
✅ Affected accounts secured
✅ Security team notified via Telegram

Creating incident report...
```

### Example 3: Code Security Review

```
You: Review ./src/api/auth.py for security issues

Konan: Conducting security code audit...

[Using $code-audit]
Analyzing authentication implementation...

⚠️ CRITICAL FINDINGS:

1. SQL Injection (CVSS 9.0)
   File: src/api/auth.py:45
   Issue: String concatenation in SQL query

   Vulnerable code:
   query = "SELECT * FROM users WHERE username = '" + username + "'"

   Remediation:
   query = "SELECT * FROM users WHERE username = ?"
   cursor.execute(query, (username,))

2. Weak Password Hashing (CVSS 7.5)
   File: src/api/auth.py:78
   Issue: Using MD5 for password hashing

   Remediation: Use bcrypt or argon2

[Recommendations]
- Fix SQL injection immediately (P0)
- Migrate to bcrypt for passwords (P1)
- Add rate limiting to prevent brute force (P2)
```

## Integration with CI/CD

### GitHub Actions

```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Install bub
        run: |
          pip install uv
          uv sync

      - name: Run Security Assessment
        run: |
          uv run bub chat --non-interactive \
            "Please run a comprehensive security scan on this repository and report findings"

      - name: Upload Results
        uses: actions/upload-artifact@v2
        with:
          name: security-report
          path: security-*.json
```

## Agent Commands

### Core Commands

```bash
# List available skills
,skills.list

# Get skill details
,skills.describe name=vulnerability-scanner

# List tools
,tools

# Create investigation phase anchor
,tape.handoff name="security-scan" summary="Completed vulnerability assessment"

# Search past investigations
,tape.search query="CVE-2024"

# View investigation timeline
,tape.info
```

### Security Operations

```bash
# Quick vulnerability scan
,bash trivy image myapp:latest

# Check CVE
,bash curl "https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=CVE-2024-1234" | jq .

# Analyze logs
,bash grep "Failed password" /var/log/auth.log | wc -l

# Check system hardening
,bash sudo lynis audit system --quick
```

## Documentation

Detailed documentation for each skill is available in:

- `src/bub/skills/vulnerability-scanner/SKILL.md`
- `src/bub/skills/threat-intel/SKILL.md`
- `src/bub/skills/code-audit/SKILL.md`
- `src/bub/skills/incident-response/SKILL.md`
- `src/bub/skills/compliance-check/SKILL.md`
- `src/bub/skills/log-analysis/SKILL.md`

## Agent Philosophy

> **"真相只有一个！" (There is only one truth!)**

The agent follows Konan's detective principles:

1. **Evidence-Based** - All findings backed by concrete evidence
2. **Methodical** - Systematic approach to security investigation
3. **Thorough** - Leave no stone unturned
4. **Educational** - Explain findings and remediation
5. **Proactive** - Don't wait for attacks, hunt for vulnerabilities

## Security Best Practices

1. **Regular Scanning**
   - Daily vulnerability scans
   - Weekly compliance checks
   - Monthly security audits

2. **Incident Preparedness**
   - Test incident response playbooks
   - Maintain updated contact lists
   - Regular backup verification

3. **Continuous Learning**
   - Track new CVEs daily
   - Subscribe to security advisories
   - Participate in CTF challenges

4. **Defense in Depth**
   - Multiple layers of security
   - Assume breach mentality
   - Zero trust architecture

## Contributing

When contributing security enhancements:

1. Test with actual security tools
2. Validate detection accuracy
3. Avoid false positives
4. Document thoroughly
5. Follow responsible disclosure

## References

- [Claude Code Security](https://claude.com/solutions/claude-code-security)
- [Wiz Vulnerability Database](https://www.wiz.io/vulnerability-database)
- [Wiz AI Cyber Model Arena](https://www.wiz.io/cyber-model-arena)
- [CloudVulnDB](https://www.cloudvulndb.org/)
- [OWASP Top 10](https://owasp.org/Top10/)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)

## License

Apache 2.0 - See LICENSE file

---

**Remember:** 真相只有一个！ (The truth is always one!)

Stay vigilant, stay secure. 🔍🔒

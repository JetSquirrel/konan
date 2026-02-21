---
name: compliance-check
description: Security compliance verification and audit. Use when you need to (1) Verify compliance with security standards (OWASP, CIS, PCI-DSS, HIPAA, SOC2), (2) Generate compliance reports, (3) Audit security configurations, (4) Check policy enforcement, or (5) Prepare for security certifications.
metadata:
  type: security
  category: compliance
---

# Compliance Check Skill

Automated compliance verification against industry security standards and frameworks.

## Prerequisites

### Compliance Tools

```bash
# OpenSCAP - Security compliance scanner
sudo apt-get install libopenscap8 python3-openscap

# Lynis - System auditing tool
git clone https://github.com/CISOfy/lynis
cd lynis && sudo ./lynis audit system

# InSpec - Compliance as Code
curl https://omnitruck.chef.io/install.sh | sudo bash -s -- -P inspec

# Docker Bench for Security
docker run --net host --pid host --userns host --cap-add audit_control \
  -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
  -v /etc:/etc:ro \
  -v /usr/bin/containerd:/usr/bin/containerd:ro \
  -v /usr/bin/runc:/usr/bin/runc:ro \
  -v /usr/lib/systemd:/usr/lib/systemd:ro \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --label docker_bench_security \
  docker/docker-bench-security
```

## Compliance Frameworks

### 1. CIS Benchmarks

Comprehensive security configuration baselines:

```bash
# CIS Ubuntu Benchmark
sudo oscap xccdf eval \
  --profile xccdf_org.ssgproject.content_profile_cis \
  --results cis-ubuntu-results.xml \
  --report cis-ubuntu-report.html \
  /usr/share/xml/scap/ssg/content/ssg-ubuntu2004-ds.xml

# CIS Docker Benchmark
docker run --rm --net host --pid host --userns host --cap-add audit_control \
  -v /etc:/etc:ro \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  docker/docker-bench-security

# CIS Kubernetes Benchmark
git clone https://github.com/aquasecurity/kube-bench
./kube-bench run --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml
```

**Key CIS Controls:**

- **Control 1**: Inventory and Control of Enterprise Assets
- **Control 2**: Inventory and Control of Software Assets
- **Control 3**: Data Protection
- **Control 4**: Secure Configuration of Enterprise Assets
- **Control 5**: Account Management
- **Control 6**: Access Control Management

### 2. OWASP ASVS (Application Security Verification Standard)

Application security requirements:

```bash
# Verify authentication (ASVS V2)
uv run ./scripts/check_owasp_asvs.py \
  --level 2 \
  --category authentication \
  --application ./src

# Verify session management (ASVS V3)
uv run ./scripts/check_owasp_asvs.py \
  --level 2 \
  --category session \
  --application ./src

# Verify access control (ASVS V4)
uv run ./scripts/check_owasp_asvs.py \
  --level 2 \
  --category access_control \
  --application ./src
```

**OWASP ASVS Levels:**

- **Level 1**: Basic security (all applications)
- **Level 2**: Standard security (most applications)
- **Level 3**: High security (critical applications)

### 3. PCI-DSS (Payment Card Industry)

For applications handling payment data:

```bash
# PCI-DSS Requirements Check
uv run ./scripts/check_pci_dss.py \
  --requirement all \
  --scope ./payment-service \
  --output pci-compliance-report.pdf
```

**PCI-DSS Key Requirements:**

| Requirement | Description | Check Command |
|-------------|-------------|---------------|
| 1 | Firewall Configuration | `sudo iptables -L -n -v` |
| 2 | No Default Credentials | `grep -r "default_password" ./` |
| 3 | Protect Stored Data | Check encryption at rest |
| 4 | Encrypt Transmission | Verify TLS/SSL |
| 6 | Secure Development | Code audit with $code-audit |
| 8 | Unique IDs | Check user management |
| 10 | Log Everything | Verify audit logging |

### 4. HIPAA (Healthcare)

For applications handling health information:

```bash
# HIPAA Security Rule Compliance
uv run ./scripts/check_hipaa.py \
  --rule security \
  --safeguards administrative,physical,technical \
  --output hipaa-compliance-report.pdf
```

**HIPAA Safeguards:**

- **Administrative**: Policies, risk analysis, workforce training
- **Physical**: Facility access, workstation security
- **Technical**: Access control, encryption, audit logs

### 5. SOC 2 (Service Organization Control)

Trust Service Criteria:

```bash
# SOC 2 Type II Readiness Assessment
uv run ./scripts/check_soc2.py \
  --criteria security,availability,confidentiality \
  --evidence-collection \
  --output soc2-readiness.pdf
```

**SOC 2 Criteria:**

- **Security**: Protection against unauthorized access
- **Availability**: System availability for operation
- **Processing Integrity**: Complete and accurate processing
- **Confidentiality**: Protection of confidential information
- **Privacy**: PII collection, use, and disposal

### 6. GDPR (Data Privacy)

European data protection regulation:

```bash
# GDPR Compliance Check
uv run ./scripts/check_gdpr.py \
  --check-data-inventory \
  --check-consent-management \
  --check-data-retention \
  --check-breach-procedures \
  --output gdpr-compliance.pdf
```

**GDPR Key Requirements:**

- Right to be forgotten implementation
- Data portability features
- Consent management
- Data breach notification (72 hours)
- Privacy by design

## Automated Compliance Checks

### System Hardening Audit

```bash
# Lynis system audit
sudo lynis audit system \
  --quick \
  --report-file lynis-report.txt

# Parse Lynis results
grep "Hardening index" lynis-report.txt
grep "Suggestions" lynis-report.txt
```

### Security Configuration Assessment

```bash
# File permissions audit
find /etc -type f -perm -002 > world-writable-config.txt

# Check for unnecessary services
systemctl list-unit-files --state=enabled | grep -v "@"

# Verify password policy
grep "^PASS" /etc/login.defs

# Check sudo configuration
sudo visudo -c

# Verify firewall rules
sudo ufw status verbose
```

### Network Security Compliance

```bash
# Check open ports
sudo nmap -sT -O localhost

# Verify TLS configuration
testssl.sh https://yourdomain.com

# Check SSL/TLS certificates
echo | openssl s_client -connect yourdomain.com:443 2>/dev/null | \
  openssl x509 -noout -dates -subject

# DNS security check
dig yourdomain.com DNSSEC
```

### Access Control Verification

```bash
# List privileged users
awk -F: '$3 < 1000 {print $1}' /etc/passwd

# Check password aging
sudo chage -l username

# Review sudo access
grep -v "^#" /etc/sudoers | grep -v "^$"

# Check SSH key authentication
for user in $(cut -f1 -d: /etc/passwd); do
  [ -d /home/$user/.ssh ] && echo "$user has SSH keys"
done
```

## Compliance Reporting

### Generate Compliance Report

```bash
# Comprehensive compliance report
uv run ./scripts/generate_compliance_report.py \
  --frameworks cis,owasp,pci-dss \
  --scope all \
  --include-evidence \
  --format pdf \
  --output compliance-report-$(date +%Y%m%d).pdf
```

**Report Sections:**

1. Executive Summary
2. Compliance Status by Framework
3. Findings and Gaps
4. Evidence of Compliance
5. Remediation Roadmap
6. Risk Assessment

### Continuous Compliance Monitoring

```bash
# Schedule daily compliance checks
crontab -e
# Add:
0 2 * * * /path/to/compliance-check.sh > /var/log/compliance/$(date +\%Y\%m\%d).log 2>&1
```

### Evidence Collection

```bash
# Collect compliance evidence
mkdir -p compliance-evidence/$(date +%Y-%m)

# Configuration files
cp /etc/ssh/sshd_config compliance-evidence/$(date +%Y-%m)/
cp /etc/security/pwquality.conf compliance-evidence/$(date +%Y-%m)/

# Security logs (last 30 days)
journalctl --since "30 days ago" > compliance-evidence/$(date +%Y-%m)/system-logs.txt

# User access records
sudo last -F > compliance-evidence/$(date +%Y-%m)/login-history.txt

# System updates
dpkg -l > compliance-evidence/$(date +%Y-%m)/installed-packages.txt
```

## Compliance Automation with InSpec

### InSpec Profiles

Create InSpec profile for custom checks:

```ruby
# controls/security_baseline.rb
control 'password-policy' do
  impact 1.0
  title 'Ensure strong password policy'
  desc 'Password policy should enforce complexity'

  describe file('/etc/security/pwquality.conf') do
    its('content') { should match /minlen = 14/ }
    its('content') { should match /dcredit = -1/ }
  end
end

control 'ssh-hardening' do
  impact 1.0
  title 'SSH should be hardened'

  describe sshd_config do
    its('PermitRootLogin') { should eq 'no' }
    its('PasswordAuthentication') { should eq 'no' }
    its('Protocol') { should eq '2' }
  end
end
```

Run InSpec:

```bash
inspec exec ./security-baseline --reporter json:compliance-results.json cli
```

## Framework-Specific Checks

### CIS Level 1 Compliance Check

```bash
#!/bin/bash
# CIS Ubuntu 20.04 Level 1 Checks

echo "=== CIS Ubuntu 20.04 Benchmark Level 1 ==="

# 1.1.1.1 Ensure mounting of cramfs filesystems is disabled
if lsmod | grep -q cramfs; then
    echo "FAIL: cramfs is not disabled"
else
    echo "PASS: cramfs is disabled"
fi

# 1.5.1 Ensure permissions on bootloader config are configured
if [ $(stat -c %a /boot/grub/grub.cfg) -eq 400 ]; then
    echo "PASS: grub.cfg permissions correct"
else
    echo "FAIL: grub.cfg permissions incorrect"
fi

# 3.4.1.1 Ensure a Firewall package is installed
if dpkg -l | grep -q ufw; then
    echo "PASS: ufw installed"
else
    echo "FAIL: ufw not installed"
fi

# More checks...
```

### OWASP Top 10 Compliance

```bash
# Check OWASP Top 10 2021 compliance
uv run ./scripts/check_owasp_top10.py \
  --application ./src \
  --output owasp-compliance.json

# Parse results
jq '.[] | select(.compliant == false) | {category: .category, issues: .issues}' owasp-compliance.json
```

### PCI-DSS Requirement 6.5 (Secure Coding)

```bash
# Scan for PCI-DSS 6.5 violations
semgrep --config "r/generic.secrets" ./src  # 6.5.3 Cryptographic storage
semgrep --config "p/sql-injection" ./src     # 6.5.1 Injection flaws
semgrep --config "p/xss" ./src               # 6.5.7 XSS
semgrep --config "p/csrf" ./src              # 6.5.9 CSRF
```

## Gap Analysis & Remediation

### Identify Compliance Gaps

```bash
# Generate gap analysis
uv run ./scripts/gap_analysis.py \
  --current-state ./current-compliance.json \
  --target-framework cis-level2 \
  --output gap-analysis.md
```

**Gap Report Format:**

```markdown
# Compliance Gap Analysis - CIS Level 2

## Summary
- Total Controls: 153
- Compliant: 98 (64%)
- Non-Compliant: 45 (29%)
- Not Applicable: 10 (7%)

## High Priority Gaps
1. **Control 5.2.1**: Ensure password creation requirements
   - Status: Non-Compliant
   - Risk: High
   - Remediation: Configure /etc/security/pwquality.conf
   - Effort: 2 hours

2. **Control 4.1.2**: Ensure auditd is enabled
   - Status: Non-Compliant
   - Risk: High
   - Remediation: systemctl enable auditd
   - Effort: 1 hour
```

### Remediation Tracking

```bash
# Track remediation progress
uv run ./scripts/track_remediation.py \
  --gap-analysis gap-analysis.json \
  --update-status \
  --generate-dashboard
```

## Audit Trail & Documentation

### Compliance Audit Log

```bash
# Record compliance check to tape
,tape.handoff name="compliance-check" summary="CIS Level 1 audit completed - 85% compliant"

# Record remediation action
,tape.handoff name="remediation" summary="Fixed Control 5.2.1 - Updated password policy"
```

### Evidence Management

```bash
# Create evidence package for auditors
uv run ./scripts/package_evidence.py \
  --framework soc2 \
  --period 2024-Q1 \
  --include-logs \
  --include-screenshots \
  --include-policies \
  --output soc2-evidence-2024Q1.zip
```

## Continuous Compliance

### CI/CD Integration

```yaml
# .github/workflows/compliance.yml
name: Compliance Check

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM
  push:
    branches: [main]

jobs:
  compliance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Run OWASP Check
        run: semgrep --config "p/owasp-top-ten" ./src

      - name: Run CIS Docker Benchmark
        run: docker run --rm docker/docker-bench-security

      - name: Upload Results
        uses: actions/upload-artifact@v2
        with:
          name: compliance-report
          path: compliance-report.json
```

### Dashboard & Metrics

```bash
# Generate compliance dashboard
uv run ./scripts/compliance_dashboard.py \
  --metrics compliance-score,gap-trend,remediation-velocity \
  --period 90d \
  --export html \
  --output compliance-dashboard.html
```

**Key Metrics:**

- Compliance Score (% of controls met)
- Gap Trend (improving/declining)
- Time to Remediation (average)
- Control Coverage by Framework

## Best Practices

1. **Continuous Monitoring**
   - Automate compliance checks
   - Track changes over time
   - Set up alerts for drift

2. **Evidence Collection**
   - Maintain audit trail
   - Document all checks
   - Preserve evidence for auditors

3. **Risk-Based Approach**
   - Prioritize critical controls
   - Focus on high-risk areas
   - Accept documented exceptions

4. **Integration**
   - Build into CI/CD pipeline
   - Link to incident response
   - Connect to vulnerability management

5. **Stakeholder Communication**
   - Regular compliance reports
   - Executive dashboards
   - Auditor-friendly evidence

## Compliance Checklist

### Pre-Audit Preparation

- [ ] Run automated compliance scans
- [ ] Collect evidence of controls
- [ ] Document policy exceptions
- [ ] Review access logs
- [ ] Verify backup/recovery
- [ ] Test incident response
- [ ] Prepare control matrix
- [ ] Schedule auditor meetings

### During Audit

- [ ] Provide evidence packages
- [ ] Demonstrate controls
- [ ] Show continuous monitoring
- [ ] Present remediation plans
- [ ] Address auditor questions
- [ ] Document findings

### Post-Audit

- [ ] Review audit findings
- [ ] Create remediation plan
- [ ] Track remediation progress
- [ ] Update policies/procedures
- [ ] Schedule re-assessment
- [ ] Share lessons learned

## Reference Resources

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)
- [PCI-DSS Standards](https://www.pcisecuritystandards.org/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [ISO 27001](https://www.iso.org/isoiec-27001-information-security.html)
- [SOC 2 Trust Criteria](https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/socforserviceorganizations.html)
- [GDPR](https://gdpr.eu/)

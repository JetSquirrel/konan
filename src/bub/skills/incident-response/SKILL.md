---
name: incident-response
description: Security incident response and investigation. Use when you need to (1) Respond to security incidents, (2) Contain and investigate breaches, (3) Perform forensic analysis, (4) Coordinate incident response team, or (5) Document incident timeline and remediation.
metadata:
  type: security
  category: incident-response
---

# Incident Response Skill

Structured approach to security incident handling based on NIST and SANS frameworks.

## Prerequisites

### Tools

```bash
# Log analysis
sudo apt-get install jq ripgrep

# Network analysis
sudo apt-get install tcpdump wireshark-common

# File analysis
sudo apt-get install file strings binutils

# System forensics (optional)
# volatility3 for memory analysis
# autopsy for disk forensics
```

### Access Requirements

- Administrative/root access for system investigation
- Access to security logs and SIEM
- Telegram/Discord configured for team alerts

## Incident Response Phases

### Phase 1: Preparation

**Pre-Incident Setup:**

```bash
# Verify monitoring is active
systemctl status auditd
systemctl status rsyslog

# Check log retention
df -h /var/log
du -sh /var/log/*

# Verify backup status
# System-specific backup check commands

# Test alert mechanisms
uv run ../telegram/scripts/telegram_send.py \
  --chat-id $SECURITY_TEAM_CHAT \
  --message "Test: Incident response system operational"
```

### Phase 2: Detection & Analysis

**Incident Classification:**

```bash
# Quick incident triage
uv run ./scripts/triage_incident.py \
  --type <malware|breach|dos|unauthorized_access|data_leak> \
  --severity <critical|high|medium|low> \
  --affected-systems "server1,server2" \
  --initial-evidence "./evidence.log"
```

**Severity Levels:**

- **Critical (P0)**: Active breach, data exfiltration, ransomware
- **High (P1)**: Confirmed compromise, privilege escalation
- **Medium (P2)**: Suspicious activity, attempted breach
- **Low (P3)**: Security policy violation, minor anomaly

**Evidence Collection:**

```bash
# Collect system logs
journalctl --since "1 hour ago" > incident-journal.log

# Collect auth logs
grep -i "authentication failure" /var/log/auth.log > failed-auth.log

# Collect Apache/Nginx access logs
tail -n 10000 /var/log/nginx/access.log | grep "suspicious-pattern" > access-evidence.log

# Network connections
ss -tunap > active-connections.txt
netstat -plant > listening-ports.txt

# Running processes
ps aux > running-processes.txt
top -b -n 1 > system-status.txt

# File system timeline
find / -mtime -1 -type f > recent-changes.txt
```

### Phase 3: Containment

**Short-term Containment:**

```bash
# Block malicious IP immediately
sudo iptables -A INPUT -s <malicious_ip> -j DROP

# Disable compromised account
sudo usermod -L compromised_user

# Isolate affected system
sudo iptables -A INPUT -j DROP
sudo iptables -A OUTPUT -j DROP
# Keep SSH for investigation
sudo iptables -I INPUT -p tcp --dport 22 -s <admin_ip> -j ACCEPT

# Kill malicious process
sudo kill -9 <pid>

# Block malicious domain in hosts
echo "0.0.0.0 malicious.domain.com" | sudo tee -a /etc/hosts
```

**Alert Security Team:**

```bash
uv run ../telegram/scripts/telegram_send.py \
  --chat-id $SECURITY_TEAM_CHAT \
  --message "🚨 CRITICAL INCIDENT DETECTED

Type: Unauthorized Access
Severity: P0
Affected: production-server-01
Status: Contained

Actions taken:
- IP blocked: 192.168.1.100
- User locked: compromised_user
- Network isolated

Investigating now. Details to follow."
```

### Phase 4: Eradication

**Remove Threat:**

```bash
# Remove malware
sudo rm -f /tmp/malicious_script.sh
sudo rm -f /var/www/.backdoor.php

# Remove malicious cron jobs
sudo crontab -u compromised_user -r
sudo rm -f /etc/cron.d/malicious_job

# Remove unauthorized SSH keys
sudo rm -f /home/compromised_user/.ssh/authorized_keys

# Revert file changes
git checkout -- compromised_file.php

# Update and patch system
sudo apt-get update && sudo apt-get upgrade -y

# Change compromised credentials
uv run ./scripts/rotate_credentials.py \
  --service all \
  --force \
  --notify-team
```

### Phase 5: Recovery

**Restore Operations:**

```bash
# Restore from clean backup
uv run ./scripts/restore_backup.py \
  --backup-id clean-backup-20240115 \
  --verify-integrity \
  --test-before-production

# Verify system integrity
sudo aide --check
sudo debsums -c

# Remove containment rules (carefully)
sudo iptables -F
# Restore proper firewall rules

# Re-enable services
sudo systemctl start nginx
sudo systemctl start application

# Verify functionality
curl -I http://localhost/health
```

### Phase 6: Post-Incident Activity

**Documentation:**

```bash
# Generate incident report
uv run ./scripts/generate_incident_report.py \
  --incident-id INC-2024-001 \
  --timeline ./evidence/timeline.json \
  --impact-analysis \
  --remediation-steps \
  --lessons-learned \
  --output incident-report-INC-2024-001.pdf
```

**Timeline Tracking:**

```bash
# Record to tape system for audit trail
,tape.handoff name="incident-detected" summary="Unauthorized access detected on server-01"
# ... investigation steps ...
,tape.handoff name="incident-contained" summary="Malicious IP blocked, user disabled"
# ... eradication steps ...
,tape.handoff name="incident-resolved" summary="System restored, credentials rotated"
```

## Investigation Playbooks

### Unauthorized Access Investigation

```bash
# 1. Identify entry point
grep "Accepted publickey\|Accepted password" /var/log/auth.log | tail -50

# 2. Track user activity
sudo aureport -au -i --start recent --summary

# 3. Find executed commands
cat /home/$USER/.bash_history

# 4. Check file access
sudo ausearch -f /etc/passwd -i

# 5. Network activity
sudo tcpdump -i eth0 -w capture.pcap 'host <suspicious_ip>'
```

### Malware Investigation

```bash
# 1. File analysis
file suspicious_file
strings suspicious_file | grep -i "http\|IP\|password"

# 2. Hash comparison
sha256sum suspicious_file
# Check against VirusTotal using $threat-intel skill

# 3. Runtime analysis (sandbox recommended)
strace ./suspicious_file 2>&1 | tee strace.log

# 4. Network indicators
lsof -i -P -n | grep suspicious_process
```

### Data Breach Investigation

```bash
# 1. Identify data exfiltration
sudo tcpdump -r capture.pcap -A | grep -i "select\|password\|ssn\|credit"

# 2. Check database access logs
grep "SELECT" /var/log/mysql/mysql.log | grep "<compromised_user>"

# 3. Identify affected records
mysql -e "SELECT * FROM audit_log WHERE timestamp > 'breach_start' AND timestamp < 'breach_end'"

# 4. Estimate impact
uv run ./scripts/assess_breach_impact.py \
  --breach-window "2024-01-15 10:00 to 2024-01-15 12:00" \
  --affected-tables "users,payments" \
  --generate-notification-list
```

## Forensic Analysis

### Memory Dump Analysis

```bash
# Capture memory (requires volatility)
sudo python3 vol.py -f memory.dmp windows.info
sudo python3 vol.py -f memory.dmp windows.pslist
sudo python3 vol.py -f memory.dmp windows.netscan
sudo python3 vol.py -f memory.dmp windows.cmdline
```

### Disk Forensics

```bash
# Create forensic image
sudo dd if=/dev/sda1 of=disk-image.dd bs=4M status=progress

# Mount read-only
sudo mount -o ro,loop disk-image.dd /mnt/forensics

# Timeline analysis
fls -r -m / disk-image.dd > timeline.body
mactime -b timeline.body > timeline.txt
```

### Log Analysis

```bash
# Aggregate suspicious events
cat /var/log/auth.log | grep "Failed password" | \
  awk '{print $1,$2,$3,$9,$11}' | sort | uniq -c | sort -rn | head -20

# Identify brute force attempts
grep "Failed password" /var/log/auth.log | \
  awk '{print $11}' | sort | uniq -c | \
  awk '$1 > 10 {print $2 " - " $1 " attempts"}'

# Find privilege escalation
grep "sudo" /var/log/auth.log | grep -v "session opened"

# Detect lateral movement
grep "sshd.*Accepted" /var/log/auth.log | awk '{print $1,$2,$3,$9,$11}'
```

## Team Coordination

### Initial Alert

```bash
# Notify security team
uv run ../telegram/scripts/telegram_send.py \
  --chat-id $SECURITY_TEAM_CHAT \
  --message "🔴 Security Incident Declared

Incident ID: INC-2024-001
Type: Unauthorized Access
Severity: CRITICAL
Declared By: Conan
Declared At: $(date -u '+%Y-%m-%d %H:%M:%S UTC')

Incident Commander: [Name]
Communication Channel: #incident-2024-001

All hands on deck. Join war room."
```

### Status Updates

```bash
# Regular updates every 30 minutes
uv run ./scripts/send_status_update.py \
  --incident-id INC-2024-001 \
  --status "Investigating - Containment in progress" \
  --next-update 30min \
  --channel telegram
```

### Stakeholder Communication

```bash
# Executive summary (non-technical)
uv run ./scripts/executive_summary.py \
  --incident-id INC-2024-001 \
  --format email \
  --to "leadership@company.com" \
  --subject "Security Incident Update - INC-2024-001"
```

## Compliance & Legal

### Evidence Preservation

```bash
# Create evidence package
mkdir -p evidence/INC-2024-001
cd evidence/INC-2024-001

# Copy with timestamps preserved
cp -p /var/log/auth.log ./auth.log
cp -p /var/log/syslog ./syslog

# Calculate hashes
sha256sum * > evidence-hashes.txt

# Sign evidence package
gpg --sign evidence-hashes.txt

# Archive
tar -czf ../INC-2024-001-evidence.tar.gz .
```

### Chain of Custody

```bash
# Document evidence handling
cat > chain-of-custody.txt <<EOF
Evidence ID: INC-2024-001-001
Description: Auth log from compromised server
Collected By: Edogawa Konan
Collected At: $(date -u)
Hash: $(sha256sum auth.log | awk '{print $1}')
Storage: evidence/INC-2024-001/
EOF
```

## Incident Report Template

```markdown
# Incident Report: INC-2024-001

## Executive Summary
Brief overview of incident for leadership (2-3 sentences)

## Incident Details
- **Incident ID**: INC-2024-001
- **Severity**: Critical
- **Type**: Unauthorized Access
- **Detection Time**: 2024-01-15 14:30 UTC
- **Containment Time**: 2024-01-15 15:00 UTC
- **Resolution Time**: 2024-01-15 18:00 UTC
- **Total Duration**: 3.5 hours

## Timeline
| Time (UTC) | Event | Action Taken |
|------------|-------|--------------|
| 14:30 | Suspicious login detected | Alert triggered |
| 14:35 | Investigation started | Evidence collection |
| 14:50 | Confirmed unauthorized access | IP blocked, user disabled |
| 15:00 | Containment achieved | Network isolated |
| 16:00 | Malware removed | System cleaned |
| 17:00 | System restored | From clean backup |
| 18:00 | Incident closed | Monitoring resumed |

## Impact Assessment
- **Systems Affected**: production-server-01, database-01
- **Data Compromised**: User emails (1,234 records)
- **Business Impact**: Minimal - no service disruption
- **Financial Impact**: TBD

## Root Cause
Weak password policy allowed brute force attack to succeed.

## Remediation Actions
1. ✅ Blocked malicious IP
2. ✅ Rotated all credentials
3. ✅ Restored from backup
4. ✅ Enhanced password policy
5. 🔄 Deploying MFA (in progress)

## Lessons Learned
- Need automated account lockout
- MFA should be mandatory
- Alert thresholds were too high

## Recommendations
1. Implement MFA organization-wide
2. Deploy fail2ban for brute force protection
3. Increase security monitoring budget
4. Conduct security awareness training
```

## Recovery Time Objectives

| Incident Type | RTO | RPO | Priority |
|---------------|-----|-----|----------|
| Malware | 4 hours | 1 hour | P1 |
| Data Breach | 1 hour | N/A | P0 |
| DoS Attack | 30 min | N/A | P0 |
| Unauthorized Access | 2 hours | 1 hour | P1 |
| Configuration Error | 1 hour | N/A | P2 |

## Best Practices

1. **Speed Over Perfection**
   - Contain first, investigate later
   - Document as you go
   - Use standardized playbooks

2. **Communication is Key**
   - Keep team informed
   - Set expectations on timeline
   - Provide regular updates

3. **Preserve Evidence**
   - Never modify original logs
   - Maintain chain of custody
   - Document all actions

4. **Learn and Improve**
   - Conduct post-incident review
   - Update playbooks
   - Share lessons learned

5. **Legal Considerations**
   - Involve legal early for breaches
   - Follow notification requirements
   - Preserve evidence properly

## Reference Resources

- [NIST Incident Response Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)
- [SANS Incident Response Steps](https://www.sans.org/media/score/504-incident-response-cycle.pdf)
- [OWASP Incident Response](https://owasp.org/www-community/Incident_Response_Checklists)

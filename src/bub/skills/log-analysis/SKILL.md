---
name: log-analysis
description: Security log analysis and anomaly detection. Use when you need to (1) Analyze system and application logs for security events, (2) Detect anomalous patterns and suspicious activities, (3) Investigate authentication failures, (4) Track user activities, or (5) Correlate security events across multiple sources.
metadata:
  type: security
  category: log-analysis
---

# Log Analysis Skill

Advanced security log analysis for threat detection and investigation.

## Prerequisites

### Log Analysis Tools

```bash
# Core tools (usually pre-installed)
sudo apt-get install jq ripgrep grep awk

# Advanced log analysis
pip install logparser
pip install pandas numpy

# Log aggregation (optional)
# ElasticSearch, Splunk, or similar SIEM
```

### Log Sources

Common security log locations:

```bash
# System logs
/var/log/syslog          # General system activity
/var/log/auth.log        # Authentication attempts
/var/log/secure          # Security/auth (RHEL)
/var/log/audit/audit.log # Audit subsystem

# Web server logs
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/apache2/access.log
/var/log/apache2/error.log

# Application logs
/var/log/application/*.log

# Container logs
docker logs <container_id>
kubectl logs <pod_name>

# Systemd journal
journalctl --since "1 hour ago"
```

## Core Analysis Techniques

### 1. Authentication Analysis

**Failed Login Attempts:**

```bash
# Count failed SSH attempts by IP
grep "Failed password" /var/log/auth.log | \
  awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

# Failed login with usernames
grep "Failed password for" /var/log/auth.log | \
  awk '{print $(NF-5), $(NF-3)}' | sort | uniq -c | sort -rn

# Successful logins
grep "Accepted password\|Accepted publickey" /var/log/auth.log | \
  awk '{print $1,$2,$3,$9,$(NF-3)}' | tail -20

# Brute force detection (>10 failures from same IP)
grep "Failed password" /var/log/auth.log | \
  awk '{print $(NF-3)}' | sort | uniq -c | awk '$1 > 10'
```

**Privilege Escalation:**

```bash
# sudo usage
grep "sudo" /var/log/auth.log | grep -v "session opened"

# su command usage
grep "su\[" /var/log/auth.log

# Root access
grep "session opened for user root" /var/log/auth.log
```

**Account Activity:**

```bash
# User login history
last -F | head -50

# Failed login attempts per user
lastb -F | awk '{print $1}' | sort | uniq -c | sort -rn

# Current logged-in users
who -a

# Last command executed by user
sudo last -x username
```

### 2. Web Application Logs

**Suspicious Requests:**

```bash
# SQL injection attempts
grep -E "(union.*select|drop.*table|' or '1'='1)" /var/log/nginx/access.log

# XSS attempts
grep -E "(<script>|javascript:|onerror=)" /var/log/nginx/access.log

# Path traversal attempts
grep -E "(\.\.\/|\.\.\\\\)" /var/log/nginx/access.log

# Command injection
grep -E "(;.*cat|;.*ls|`.*`)" /var/log/nginx/access.log
```

**Anomalous Patterns:**

```bash
# Requests to unusual file types
grep -E "\.(bak|old|tmp|swp|sql|zip)$" /var/log/nginx/access.log

# High rate requests from single IP
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Non-standard HTTP methods
grep -v "GET\|POST\|HEAD" /var/log/nginx/access.log

# Error responses (4xx, 5xx)
grep " 40[0-9] \| 50[0-9] " /var/log/nginx/access.log | tail -50
```

**Attack Fingerprinting:**

```bash
# User-Agent analysis (identify bots/scanners)
awk -F\" '{print $6}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Scanner detection
grep -E "(nikto|nmap|masscan|sqlmap|nessus)" /var/log/nginx/access.log

# WordPress attack attempts
grep "wp-admin\|wp-login\|xmlrpc" /var/log/nginx/access.log
```

### 3. Network Activity Analysis

**Connection Monitoring:**

```bash
# Active connections by remote IP
ss -tunap | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn

# Port scan detection (many connections to different ports)
tcpdump -nn -r capture.pcap 'tcp[tcpflags] & (tcp-syn) != 0' | \
  awk '{print $3}' | cut -d. -f1-4 | sort | uniq -c | sort -rn

# DNS query analysis
grep "query" /var/log/syslog | awk '{print $NF}' | sort | uniq -c | sort -rn
```

**Firewall Logs:**

```bash
# Blocked connections
grep "UFW BLOCK" /var/log/syslog | tail -50

# Top blocked IPs
grep "UFW BLOCK" /var/log/syslog | awk '{print $12}' | \
  cut -d= -f2 | sort | uniq -c | sort -rn

# Blocked destination ports
grep "UFW BLOCK" /var/log/syslog | awk '{print $14}' | \
  cut -d= -f2 | sort | uniq -c | sort -rn
```

### 4. System Security Events

**File System Changes:**

```bash
# Recently modified files
find /etc /usr/bin /usr/sbin -type f -mtime -1 -ls

# SUID/SGID files
find / -perm -4000 -o -perm -2000 2>/dev/null

# World-writable files
find /etc -type f -perm -002 2>/dev/null
```

**Process Activity:**

```bash
# Suspicious processes
ps aux | grep -E "(nc|ncat|/tmp/|wget|curl)" | grep -v grep

# Processes listening on unusual ports
ss -tlnp | grep -v ":22\|:80\|:443"

# High CPU/Memory usage (potential DoS or crypto mining)
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10
```

**Audit Logs:**

```bash
# File access audit
sudo ausearch -f /etc/passwd -i

# User command execution
sudo ausearch -ua username -i

# System calls by process
sudo ausearch -c sshd -i
```

### 5. Application Logs

**Error Pattern Analysis:**

```bash
# Error frequency
grep -i "error\|exception\|fail" /var/log/application.log | \
  awk '{print $1,$2}' | uniq -c

# Stack trace extraction
awk '/Exception/,/^$/' /var/log/application.log

# Top errors
grep -i "error" /var/log/application.log | \
  sed 's/[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}.*\[ERROR\]//' | \
  sort | uniq -c | sort -rn | head -20
```

## Advanced Analysis Patterns

### Timeline Analysis

```bash
# Create activity timeline
cat /var/log/auth.log | awk '{print $1,$2,$3}' | sort | uniq -c

# Visualize hourly activity
awk '{print $3}' /var/log/auth.log | cut -d: -f1 | sort | uniq -c

# Correlation across multiple logs
grep "2024-01-15 14:" /var/log/auth.log /var/log/syslog /var/log/nginx/access.log
```

### Anomaly Detection

```bash
# Baseline normal activity
# Calculate average requests per hour
awk '{print $4}' /var/log/nginx/access.log | cut -d: -f2 | sort | uniq -c

# Detect spikes (3x average)
# Compare current hour to baseline

# Unusual time access (3 AM activity)
grep "Jan 15 0[0-3]:" /var/log/auth.log

# Geographic anomalies (requires GeoIP)
# Access from unusual countries
```

### Correlation Analysis

```bash
# Multi-log correlation script
uv run ./scripts/correlate_events.py \
  --logs /var/log/auth.log,/var/log/syslog,/var/log/nginx/access.log \
  --time-window 5min \
  --pivot-field ip_address \
  --output correlation-report.json
```

## Helper Scripts

### Comprehensive Log Analysis

```bash
# Run full log analysis
uv run ./scripts/analyze_logs.py \
  --log-dir /var/log \
  --types auth,web,system \
  --lookback 24h \
  --detect-anomalies \
  --output security-log-analysis.json
```

**Script Features:**
- Failed authentication detection
- Brute force identification
- Anomaly scoring
- Attack pattern matching
- IOC extraction
- Timeline generation

### Security Event Aggregation

```bash
# Aggregate security events
uv run ./scripts/aggregate_security_events.py \
  --sources syslog,auth,nginx \
  --severity HIGH,CRITICAL \
  --group-by source_ip \
  --output aggregated-events.json
```

### Real-time Monitoring

```bash
# Real-time threat detection
uv run ./scripts/monitor_logs.py \
  --logs /var/log/auth.log,/var/log/nginx/access.log \
  --alert-on brute_force,sql_injection,suspicious_ua \
  --notify telegram \
  --threshold high
```

## Log Parsing Examples

### Parse Apache/Nginx Access Log

```bash
# Extract useful fields
awk -F'"' '{
  split($1, a, " ");
  print a[1], a[4], a[6], a[7], $2, $4, $6
}' /var/log/nginx/access.log | head -10

# Convert to JSON
awk -F'"' '{
  split($1, a, " ");
  printf "{\"ip\":\"%s\",\"time\":\"%s\",\"method\":\"%s\",\"path\":\"%s\",\"status\":\"%s\",\"size\":\"%s\",\"ua\":\"%s\"}\n",
  a[1], a[4], $2, $3, a[9], a[10], $6
}' /var/log/nginx/access.log | jq .
```

### Parse Auth Log

```bash
# Extract failed SSH attempts
grep "Failed password" /var/log/auth.log | \
  awk '{
    print "{\"time\":\"" $1 " " $2 " " $3 "\",\"user\":\"" $(NF-5) "\",\"ip\":\"" $(NF-3) "\"}"
  }' | jq .

# Parse sudo usage
grep "sudo:" /var/log/auth.log | \
  awk '{
    print "{\"time\":\"" $1 " " $2 " " $3 "\",\"user\":\"" $6 "\",\"command\":\"" substr($0, index($0,$10)) "\"}"
  }' | jq .
```

### Parse Syslog

```bash
# Extract kernel messages
grep "kernel:" /var/log/syslog | tail -50

# Parse systemd messages
journalctl -u nginx.service --since "1 hour ago" --output json | jq .
```

## Security Event Detection

### Brute Force Attack Detection

```bash
#!/bin/bash
# Detect brute force attacks

THRESHOLD=10
LOG_FILE="/var/log/auth.log"
TIME_WINDOW="1 hour ago"

journalctl --since "$TIME_WINDOW" | grep "Failed password" | \
  awk '{print $(NF-3)}' | sort | uniq -c | \
  while read count ip; do
    if [ $count -gt $THRESHOLD ]; then
      echo "ALERT: Brute force from $ip ($count attempts)"
      # Block IP
      sudo iptables -A INPUT -s $ip -j DROP
      # Notify team
      uv run ../telegram/scripts/telegram_send.py \
        --chat-id $SECURITY_CHAT \
        --message "🚨 Brute force detected from $ip - $count attempts. IP blocked."
    fi
  done
```

### Web Attack Detection

```bash
#!/bin/bash
# Detect web attacks

LOG_FILE="/var/log/nginx/access.log"

# SQL Injection
if grep -q -E "(union.*select|drop.*table)" $LOG_FILE; then
  echo "ALERT: SQL injection attempt detected"
fi

# XSS
if grep -q -E "(<script>|javascript:)" $LOG_FILE; then
  echo "ALERT: XSS attempt detected"
fi

# Directory traversal
if grep -q -E "\.\./|\.\.\\\" $LOG_FILE; then
  echo "ALERT: Directory traversal attempt detected"
fi
```

### Anomaly Detection Script

```python
#!/usr/bin/env python3
# Detect statistical anomalies in logs

import pandas as pd
import numpy as np
from datetime import datetime

# Read log file
logs = pd.read_csv('parsed-logs.csv')

# Convert timestamp
logs['timestamp'] = pd.to_datetime(logs['timestamp'])
logs.set_index('timestamp', inplace=True)

# Hourly request count
hourly = logs.resample('H').size()

# Calculate Z-score
mean = hourly.mean()
std = hourly.std()
z_scores = (hourly - mean) / std

# Detect anomalies (|Z| > 3)
anomalies = hourly[np.abs(z_scores) > 3]

print("Anomalous hours:")
for time, count in anomalies.items():
    print(f"{time}: {count} requests (normal: {mean:.0f})")
```

## Integration with SIEM

### ElasticSearch Integration

```bash
# Send logs to ElasticSearch
tail -f /var/log/auth.log | while read line; do
  curl -X POST "http://localhost:9200/security-logs/_doc/" \
    -H 'Content-Type: application/json' \
    -d "{\"message\":\"$line\",\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%S)\"}"
done

# Query ElasticSearch
curl -X GET "http://localhost:9200/security-logs/_search" \
  -H 'Content-Type: application/json' \
  -d '{"query":{"match":{"message":"Failed password"}}}'
```

### Splunk Integration

```bash
# Forward logs to Splunk
# Configure /opt/splunkforwarder/etc/system/local/inputs.conf
[monitor:///var/log/auth.log]
sourcetype = linux_secure
index = security

[monitor:///var/log/nginx/access.log]
sourcetype = nginx_access
index = web
```

## Reporting

### Daily Security Report

```bash
# Generate daily security summary
uv run ./scripts/daily_security_report.py \
  --date $(date +%Y-%m-%d) \
  --include failed-logins,blocked-ips,errors,anomalies \
  --output daily-report-$(date +%Y%m%d).html
```

**Report Contents:**
- Failed authentication summary
- Blocked IPs and reasons
- Top error messages
- Anomalous activities
- Recommendations

### IOC Extraction

```bash
# Extract indicators of compromise
uv run ./scripts/extract_iocs.py \
  --logs /var/log/*.log \
  --types ip,domain,hash,email \
  --output iocs.json

# Check against threat intel
cat iocs.json | jq -r '.ips[]' | while read ip; do
  # Use $threat-intel skill to check reputation
  echo "Checking $ip..."
done
```

## Best Practices

1. **Centralized Logging**
   - Aggregate logs from all systems
   - Use SIEM for correlation
   - Ensure log integrity (WORM storage)

2. **Log Retention**
   - Keep security logs for 90+ days
   - Archive for compliance (1+ years)
   - Rotate logs to manage disk space

3. **Real-time Monitoring**
   - Set up alerts for critical events
   - Use automated response for known threats
   - Monitor for anomalies

4. **Log Protection**
   - Secure log files (restrict permissions)
   - Forward logs to separate system
   - Use log signing/encryption

5. **Regular Analysis**
   - Daily review of security logs
   - Weekly trend analysis
   - Monthly compliance reports

## Performance Optimization

### For Large Log Files

```bash
# Use grep with -F for fixed strings (faster)
grep -F "Failed password" /var/log/auth.log

# Use ripgrep (much faster than grep)
rg "Failed password" /var/log/auth.log

# Parallel processing
cat large.log | parallel --pipe grep "pattern"

# Use mmap for very large files
python -c "import mmap; ..."
```

### Log Compression

```bash
# Compress old logs
gzip /var/log/auth.log.1

# Search compressed logs
zgrep "pattern" /var/log/auth.log.1.gz

# Compress rotated logs automatically
# Configure in /etc/logrotate.d/
```

## Reference Resources

- [Linux Log Files Guide](https://www.loggly.com/ultimate-guide/linux-logging-basics/)
- [Apache Log Format](https://httpd.apache.org/docs/current/logs.html)
- [Nginx Log Format](https://nginx.org/en/docs/http/ngx_http_log_module.html)
- [Systemd Journal](https://www.freedesktop.org/software/systemd/man/journalctl.html)
- [Auditd Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/chap-system_auditing)

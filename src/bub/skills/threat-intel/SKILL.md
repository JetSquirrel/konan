---
name: threat-intel
description: Threat intelligence gathering and CVE tracking. Use when you need to (1) Look up CVE details and exploit information, (2) Check if vulnerabilities are being actively exploited, (3) Query threat intelligence feeds, (4) Investigate malware indicators, or (5) Track security advisories and bulletins.
metadata:
  type: security
  category: threat-intelligence
---

# Threat Intelligence Skill

Real-time threat intelligence gathering from authoritative sources.

## Prerequisites

### API Keys (Optional but Recommended)

Configure in `.env`:

```bash
# VirusTotal API
VIRUSTOTAL_API_KEY=your_vt_api_key

# AlienVault OTX
OTX_API_KEY=your_otx_api_key

# Shodan (for infrastructure search)
SHODAN_API_KEY=your_shodan_api_key

# GitHub Security Advisory API (public, no key needed)
# Use with gh CLI or direct API calls
```

### Required Tools

```bash
# gh CLI for GitHub Security Advisories
# Already available via $gh skill

# curl/httpie for API queries
# Standard in most environments

# jq for JSON parsing
sudo apt-get install jq  # or brew install jq
```

## Core Capabilities

### 1. CVE Database Queries

#### National Vulnerability Database (NVD)

```bash
# Query specific CVE
curl "https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=CVE-2024-1234" | jq .

# Search CVEs by keyword
curl "https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch=remote+code+execution&resultsPerPage=20" | jq .

# Get recent CVEs
curl "https://services.nvd.nist.gov/rest/json/cves/2.0?pubStartDate=$(date -d '7 days ago' -I)T00:00:00.000&pubEndDate=$(date -I)T23:59:59.999" | jq .

# Get CVEs by CVSS score
curl "https://services.nvd.nist.gov/rest/json/cves/2.0?cvssV3Severity=CRITICAL" | jq .
```

#### Wiz Vulnerability Database

```bash
# Search vulnerabilities
curl "https://www.wiz.io/vulnerability-database/api/search?q=kubernetes" | jq .

# Get CVE details from Wiz
curl "https://vulnerabilities.wiz.io/CVE-2024-1234" | jq .

# Check exploitation status
curl "https://www.wiz.io/vulnerability-database/api/cve/CVE-2024-1234/exploitation" | jq .
```

#### CloudVulnDB

```bash
# Search cloud-specific vulnerabilities
curl "https://www.cloudvulndb.org/api/v1/vulnerabilities/search?cloud=aws" | jq .

# Get vulnerability details
curl "https://www.cloudvulndb.org/api/v1/vulnerabilities/CVE-2024-1234" | jq .
```

### 2. GitHub Security Advisories

```bash
# Search GitHub Security Advisories
gh api graphql -f query='
  query {
    securityVulnerabilities(first: 20, ecosystem: PYTHON, orderBy: {field: UPDATED_AT, direction: DESC}) {
      nodes {
        advisory {
          ghsaId
          summary
          severity
          publishedAt
        }
        vulnerableVersionRange
        package {
          name
        }
      }
    }
  }
' | jq .

# Get specific advisory
gh api /advisories/GHSA-xxxx-yyyy-zzzz

# Search by package
gh api /search/repositories?q=topic:security+advisory+python
```

### 3. Exploit Information

#### Exploit Database Search

```bash
# Search Exploit-DB
curl "https://www.exploit-db.com/search?cve=CVE-2024-1234" | grep -i exploit

# Check if exploit code is available
curl -s "https://www.exploit-db.com/exploits/search?cve=2024-1234" | jq .
```

#### CISA Known Exploited Vulnerabilities

```bash
# Get CISA KEV catalog
curl "https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json" | jq .

# Check if CVE is in KEV catalog
curl -s "https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json" | \
  jq '.vulnerabilities[] | select(.cveID == "CVE-2024-1234")'

# Get recent KEV additions
curl -s "https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json" | \
  jq '.vulnerabilities[] | select(.dateAdded > "'$(date -d '30 days ago' -I)'") | {cve: .cveID, name: .vulnerabilityName, date: .dateAdded}'
```

### 4. Threat Intelligence Feeds

#### AlienVault OTX

```bash
# Get threat pulses
curl -H "X-OTX-API-KEY: $OTX_API_KEY" \
  "https://otx.alienvault.com/api/v1/pulses/subscribed" | jq .

# Search indicators
curl -H "X-OTX-API-KEY: $OTX_API_KEY" \
  "https://otx.alienvault.com/api/v1/indicators/IPv4/8.8.8.8/general" | jq .

# Get CVE-related pulses
curl -H "X-OTX-API-KEY: $OTX_API_KEY" \
  "https://otx.alienvault.com/api/v1/search/pulses?q=CVE-2024-1234" | jq .
```

#### VirusTotal

```bash
# Check file hash
curl -H "x-apikey: $VIRUSTOTAL_API_KEY" \
  "https://www.virustotal.com/api/v3/files/<file_hash>" | jq .

# Check URL reputation
curl -H "x-apikey: $VIRUSTOTAL_API_KEY" \
  "https://www.virustotal.com/api/v3/urls/<url_id>" | jq .

# Check IP address
curl -H "x-apikey: $VIRUSTOTAL_API_KEY" \
  "https://www.virustotal.com/api/v3/ip_addresses/8.8.8.8" | jq .
```

### 5. Security Advisory Tracking

#### Vendor Security Bulletins

```bash
# Red Hat Security Advisories
curl "https://access.redhat.com/labs/securitydataapi/cve/CVE-2024-1234.json" | jq .

# Ubuntu Security Notices
curl "https://ubuntu.com/security/notices.json" | jq .

# Debian Security Tracker
curl "https://security-tracker.debian.org/tracker/CVE-2024-1234" | grep -i description
```

## Helper Scripts

### CVE Lookup Script

```bash
# Comprehensive CVE lookup
uv run ./scripts/lookup_cve.py \
  --cve CVE-2024-1234 \
  --sources nvd,wiz,github \
  --check-exploit \
  --check-kev \
  --output detailed
```

**Script Features:**
- Queries multiple databases simultaneously
- Checks exploitation status
- Verifies if in CISA KEV catalog
- Provides CVSS scoring
- Lists affected products
- Shows patch availability

### Threat Feed Monitor

```bash
# Monitor threat feeds for indicators
uv run ./scripts/monitor_threats.py \
  --sources otx,virustotal \
  --indicators ip,domain,hash \
  --watch-list ./threat-iocs.txt \
  --alert-on-match
```

### Advisory Tracker

```bash
# Track security advisories
uv run ./scripts/track_advisories.py \
  --vendors redhat,ubuntu,debian \
  --severity critical,high \
  --products python,nodejs,docker \
  --since 7d \
  --notify telegram
```

## Investigation Workflows

### CVE Deep Dive

When investigating a specific CVE:

```bash
# 1. Get basic CVE information
curl "https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=CVE-2024-1234" | \
  jq '{
    id: .vulnerabilities[0].cve.id,
    description: .vulnerabilities[0].cve.descriptions[0].value,
    cvss: .vulnerabilities[0].cve.metrics.cvssMetricV31[0].cvssData.baseScore,
    severity: .vulnerabilities[0].cve.metrics.cvssMetricV31[0].cvssData.baseSeverity,
    published: .vulnerabilities[0].cve.published
  }'

# 2. Check exploitation status
curl -s "https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json" | \
  jq --arg cve "CVE-2024-1234" '.vulnerabilities[] | select(.cveID == $cve)'

# 3. Search for exploit code
curl -s "https://www.exploit-db.com/search?cve=CVE-2024-1234"

# 4. Check vendor advisories
gh api /advisories?cve_id=CVE-2024-1234

# 5. Get remediation guidance
curl "https://www.wiz.io/vulnerability-database/CVE-2024-1234" | jq '.remediation'
```

### Threat Actor Investigation

Tracking threat actor activities:

```bash
# 1. Search OTX for actor indicators
curl -H "X-OTX-API-KEY: $OTX_API_KEY" \
  "https://otx.alienvault.com/api/v1/search/pulses?q=APT28" | \
  jq '.results[] | {name: .name, tags: .tags, indicators: .indicator_count}'

# 2. Get associated IoCs
curl -H "X-OTX-API-KEY: $OTX_API_KEY" \
  "https://otx.alienvault.com/api/v1/pulses/<pulse_id>/indicators" | \
  jq '.[] | {type: .type, indicator: .indicator}'

# 3. Check IoCs against VirusTotal
# (iterate through IoCs from step 2)
```

### Emerging Threat Monitoring

Track new vulnerabilities and threats:

```bash
# Get CVEs from last 24 hours
curl -s "https://services.nvd.nist.gov/rest/json/cves/2.0?pubStartDate=$(date -d '1 day ago' -I)T00:00:00.000" | \
  jq '.vulnerabilities[] | {
    cve: .cve.id,
    severity: .cve.metrics.cvssMetricV31[0].cvssData.baseSeverity,
    score: .cve.metrics.cvssMetricV31[0].cvssData.baseScore,
    description: .cve.descriptions[0].value
  } | select(.severity == "CRITICAL" or .severity == "HIGH")'

# Check new KEV additions
curl -s "https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json" | \
  jq --arg date "$(date -d '7 days ago' -I)" \
  '.vulnerabilities[] | select(.dateAdded > $date) | {cve: .cveID, added: .dateAdded, name: .vulnerabilityName}'
```

## Reporting Formats

### CVE Summary Report

```json
{
  "cve_id": "CVE-2024-1234",
  "cvss_score": 9.8,
  "severity": "CRITICAL",
  "description": "Remote code execution in...",
  "affected_products": ["product-1", "product-2"],
  "exploit_available": true,
  "in_kev_catalog": true,
  "patch_available": true,
  "references": [
    "https://nvd.nist.gov/vuln/detail/CVE-2024-1234",
    "https://github.com/advisories/GHSA-xxxx"
  ],
  "recommendations": [
    "Update to version X.Y.Z immediately",
    "Apply temporary mitigation: ..."
  ]
}
```

### Threat Intelligence Brief

```markdown
# Threat Intelligence Brief - [Date]

## New Critical Vulnerabilities (24h)
- CVE-2024-1234 (CVSS 9.8): RCE in Component X
- CVE-2024-5678 (CVSS 9.1): Auth bypass in Product Y

## Active Exploitation
- CVE-2024-1111: Added to CISA KEV catalog
- CVE-2024-2222: Exploit code published on GitHub

## Threat Actor Activity
- APT28: New campaign targeting infrastructure
- Ransomware Group X: Updated TTPs observed

## Recommended Actions
1. Patch CVE-2024-1234 immediately
2. Monitor for CVE-2024-1111 exploitation attempts
3. Review indicators for APT28 activity
```

## API Rate Limiting

### NVD API
- **Without API key**: 5 requests per 30 seconds
- **With API key**: 50 requests per 30 seconds
- Request key: https://nvd.nist.gov/developers/request-an-api-key

### VirusTotal
- **Free tier**: 500 requests/day, 4 requests/minute
- **Premium**: Higher limits

### GitHub
- **Authenticated**: 5000 requests/hour
- Use `gh` CLI for automatic auth

## Failure Handling

### API Errors

```bash
# Handle rate limiting
if [ $? -eq 429 ]; then
  echo "Rate limited, waiting 60 seconds..."
  sleep 60
  # retry
fi

# Handle API unavailability
curl --max-time 10 --retry 3 --retry-delay 5 "https://api.example.com/endpoint"
```

### Data Validation

```bash
# Validate CVE format
if [[ $CVE_ID =~ ^CVE-[0-9]{4}-[0-9]+$ ]]; then
  # valid CVE ID
else
  echo "Invalid CVE ID format"
fi

# Validate JSON response
if ! jq empty <<< "$response" 2>/dev/null; then
  echo "Invalid JSON response"
fi
```

## Integration with Other Skills

### With Vulnerability Scanner

```bash
# Scan finds CVE -> Use threat-intel to get details
trivy image myapp:latest --format json | \
  jq '.Results[].Vulnerabilities[].VulnerabilityID' | \
  xargs -I {} uv run ./scripts/lookup_cve.py --cve {}
```

### With Incident Response

```bash
# IoC detected -> Check reputation
echo "suspicious.domain.com" | \
  xargs -I {} curl -H "x-apikey: $VIRUSTOTAL_API_KEY" \
  "https://www.virustotal.com/api/v3/domains/{}"
```

## Best Practices

1. **Automate Monitoring**
   - Set up daily CVE feed checks
   - Monitor CISA KEV catalog updates
   - Track vendor security bulletins

2. **Prioritize Intelligence**
   - Focus on actively exploited CVEs
   - Track vulnerabilities in your tech stack
   - Monitor threat actors relevant to your industry

3. **Maintain Context**
   - Document threat intelligence findings in tape
   - Link CVEs to affected systems
   - Track remediation timeline

4. **Verify Information**
   - Cross-reference multiple sources
   - Validate exploit claims
   - Confirm patch effectiveness

5. **Share Intelligence**
   - Alert team via Telegram/Discord for critical findings
   - Document in handoff summaries
   - Maintain threat intelligence database

## Reference Resources

- [NVD API Documentation](https://nvd.nist.gov/developers)
- [CISA KEV Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [Wiz Vulnerability Database](https://www.wiz.io/vulnerability-database)
- [GitHub Security Advisories](https://github.com/advisories)
- [AlienVault OTX](https://otx.alienvault.com/)
- [VirusTotal](https://www.virustotal.com/)
- [Exploit Database](https://www.exploit-db.com/)
- [Wiz AI Cyber Model Arena](https://www.wiz.io/cyber-model-arena)

# Konan - Cybersecurity Agent 🔍

[![Release](https://img.shields.io/github/v/release/psiace/bub)](https://github.com/psiace/bub/releases)
[![Build status](https://img.shields.io/github/actions/workflow/status/psiace/bub/main.yml?branch=main)](https://github.com/psiace/bub/actions/workflows/main.yml?query=branch%3Amain)
[![Commit activity](https://img.shields.io/github/commit-activity/m/psiace/bub)](https://github.com/psiace/bub/graphs/commit-activity)
[![License](https://img.shields.io/github/license/psiace/bub)](LICENSE)

> 真相只有一个！ (There is only one truth!)

**江户川柯南 (Edogawa Konan)** - A cybersecurity-focused AI agent built on Bub/Republic framework.

This agent specializes in vulnerability assessment, threat intelligence, code auditing, incident response, and compliance verification. Inspired by the detective Konan, it systematically investigates security threats and protects systems in cyberspace.

## Key Features

🔍 **Vulnerability Scanner** - Comprehensive scanning for containers, code, and dependencies
🌐 **Threat Intelligence** - Real-time CVE tracking and threat feed analysis
🔒 **Code Audit** - Security-focused code review with OWASP Top 10 detection
🚨 **Incident Response** - Structured incident handling and forensic analysis
✅ **Compliance Check** - Verification against CIS, OWASP, PCI-DSS, SOC 2, HIPAA
📊 **Log Analysis** - Security event detection and anomaly identification

## About Bub

Bub is a coding agent CLI built on `republic`.
It is designed for real engineering workflows where execution must be predictable, inspectable, and recoverable.

## Four Things To Know

1. Command boundary is strict: only lines starting with `,` are treated as commands.
2. The same routing model is applied to both user input and assistant output.
3. Successful commands return directly; failed commands fall back to the model with structured context.
4. Session context is append-only tape with explicit `anchor/handoff` transitions.

## Quick Start

### Security Agent Setup

```bash
git clone https://github.com/JetSquirrel/konan.git
cd konan
uv sync
cp env.example .env
```

Configure `.env` with security tools:

```bash
# Core Configuration
BUB_MODEL=openrouter:anthropic/claude-opus-4-5
OPENROUTER_API_KEY=your_key_here

# Security Tool APIs (optional)
VIRUSTOTAL_API_KEY=your_vt_api_key
OTX_API_KEY=your_otx_api_key
SHODAN_API_KEY=your_shodan_api_key
WIZ_API_KEY=your_wiz_api_key

# Notifications
BUB_TELEGRAM_ENABLED=true
BUB_TELEGRAM_TOKEN=your_telegram_bot_token
BUB_TELEGRAM_ALLOW_FROM='["your_telegram_id"]'
```

Start the security agent:

```bash
# Interactive CLI
uv run bub chat

# Telegram bot
uv run bub message
```

## Security Skills

The agent includes six specialized security skills:

### 🔍 Vulnerability Scanner
Scan containers, code, and dependencies for CVEs using Trivy, Bandit, Semgrep.

```bash
# In agent
Can you scan Docker image myapp:latest for vulnerabilities?
```

### 🌐 Threat Intelligence
Query CVE databases, check exploitation status, and track security advisories.

```bash
# In agent
What is CVE-2024-1234 and is it being actively exploited?
```

### 🔒 Code Audit
Security-focused code review checking for OWASP Top 10 and common vulnerabilities.

```bash
# In agent
Please audit ./src/auth for security issues
```

### 🚨 Incident Response
Structured incident handling with containment, investigation, and remediation.

```bash
# In agent
We detected unauthorized access from IP 1.2.3.4. Please investigate and contain.
```

### ✅ Compliance Check
Verify compliance with CIS, OWASP, PCI-DSS, SOC 2, HIPAA, GDPR.

```bash
# In agent
Run CIS Level 1 compliance check on this system
```

### 📊 Log Analysis
Analyze security logs for threats, anomalies, and suspicious patterns.

```bash
# In agent
Analyze /var/log/auth.log for suspicious activity
```

## Original Bub Quick Start

```bash
git clone https://github.com/psiace/bub.git
cd bub
uv sync
cp env.example .env
```

Minimal `.env`:

```bash
BUB_MODEL=openrouter:qwen/qwen3-coder-next
OPENROUTER_API_KEY=your_key_here
```

Start interactive CLI:

```bash
uv run bub
```

## Interaction Rules

- `hello`: natural language routed to model.
- `,help`: internal command.
- `,git status`: shell command.
- `, ls -la`: shell command (space after comma is optional).

Common commands:

```text
,help
,tools
,tool.describe name=fs.read
,skills.list
,skills.describe name=friendly-python
,handoff name=phase-1 summary="bootstrap done"
,anchors
,tape.info
,tape.search query=error
,tape.reset archive=true
,quit
```

## Telegram (Optional)

```bash
BUB_TELEGRAM_ENABLED=true
BUB_TELEGRAM_TOKEN=123456:token
BUB_TELEGRAM_ALLOW_FROM='["123456789","your_username"]'
uv run bub message
```

## Discord (Optional)

```bash
BUB_DISCORD_ENABLED=true
BUB_DISCORD_TOKEN=discord_bot_token
BUB_DISCORD_ALLOW_FROM='["123456789012345678","your_discord_name"]'
BUB_DISCORD_ALLOW_CHANNELS='["123456789012345678"]'
uv run bub message
```

## Documentation

### Security Agent Documentation
- `docs/security-agent.md`: Comprehensive security agent guide
- `SOUL.md`: Agent personality and values
- `AGENTS.md`: Operational instructions and protocols

### Bub Framework Documentation
- `docs/index.md`: getting started and usage overview
- `docs/deployment.md`: local + Docker deployment playbook
- `docs/features.md`: key capabilities and why they matter
- `docs/cli.md`: interactive CLI workflow and troubleshooting
- `docs/architecture.md`: agent loop, tape, anchor, and tool/skill design
- `docs/telegram.md`: Telegram integration and operations
- `docs/discord.md`: Discord integration and operations

### Security Skills Documentation
Each skill has detailed documentation in its `SKILL.md` file:
- `src/bub/skills/vulnerability-scanner/SKILL.md`
- `src/bub/skills/threat-intel/SKILL.md`
- `src/bub/skills/code-audit/SKILL.md`
- `src/bub/skills/incident-response/SKILL.md`
- `src/bub/skills/compliance-check/SKILL.md`
- `src/bub/skills/log-analysis/SKILL.md`

## Development

```bash
uv run ruff check .
uv run mypy
uv run pytest -q
just docs-test
```

## License

[Apache 2.0](./LICENSE)

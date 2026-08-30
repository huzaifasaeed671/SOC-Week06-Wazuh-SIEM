# Wazuh SIEM Deployment, Endpoint Integration & Automated Threat Intelligence with n8n + Gemini AI

> ITSimplera Solutions — Internship Program, Week 06
> **Intern:** Huzaifa Saeed | **Intern ID:** SOCB01-3523
> **Task Type:** Setup · Deployment · Automation | **Difficulty:** Advanced

An end-to-end SOC automation lab: a fully deployed **Wazuh SIEM** stack monitoring Linux and Windows endpoints, wired into an **n8n** workflow automation layer that sends triggered alerts to **Google Gemini AI** for real-time severity scoring, IOC extraction, MITRE ATT&CK mapping, and response recommendations — delivered to a SOC Slack channel.

```
Wazuh Alert → n8n Webhook → Gemini AI Analysis → Enriched Output → Slack/Email Notification
```

## Highlights

- ✅ Deployed Wazuh Manager, Indexer, and Dashboard (v4.14.7) on Ubuntu Server 24.04 LTS with TLS and firewall hardening
- ✅ Enrolled two endpoints — Ubuntu 22.04 LTS and Windows 11 Pro — with active Wazuh agents
- ✅ Configured File Integrity Monitoring (FIM), Vulnerability Detection, and Security Configuration Assessment (SCA)
- ✅ Simulated **17 test security events** across brute-force SSH, unauthorized file modification, privilege escalation, and malware (EICAR) — **100% detection rate**
- ✅ Deployed n8n via Docker with bidirectional Wazuh REST API + webhook integration
- ✅ Integrated Gemini 2.5 Flash for automated alert triage, IOC extraction, and MITRE mapping
- ✅ Reduced mean detection-to-response time from **~12 minutes (manual)** to **~45 seconds (automated)**
- ✅ Reduced manual triage effort by an estimated **73%**

## Architecture

```
┌─────────────────────────────┐
│   WAZUH SERVER (SIEM Core)  │   wazuh-soc-01 · 192.168.56.10
│  Manager · Indexer ·        │   Ubuntu 24.04 · 4 vCPU / 8GB
│  Dashboard · n8n (Docker)   │
└───────────┬──────────────────┘
            │ enrollment (1514/1515)
   ┌────────┴────────┐
   ▼                 ▼
LINUX ENDPOINT    WINDOWS ENDPOINT
ubuntu-web-01     win-ws-01
192.168.56.21     192.168.56.22
FIM · Vuln ·      FIM · Vuln ·
SCA               SCA
   │                 │
   └────────┬────────┘
            │ alerts (level ≥ 7) → webhook
            ▼
   n8n WORKFLOW AUTOMATION (:5678)
   Webhook → Parse → Filter → Gemini AI → Format → Slack → Archive
            │
            ▼
   GOOGLE GEMINI AI (Cloud)
   Gemini 2.5 Flash — severity scoring, IOC extraction, MITRE mapping
```

All systems run in a closed VMware Workstation 17 Pro lab network on `192.168.56.0/24`.

## Stack

| Component | Version | Role |
|---|---|---|
| Wazuh Manager / Indexer / Dashboard | 4.14.7 (OpenSearch 2.x) | SIEM core |
| Wazuh Agent | 4.14.5 | Linux & Windows endpoint telemetry |
| n8n | 2.36.8 (Docker) | Workflow automation / orchestration |
| Google Gemini AI | 2.5 Flash (AI Studio) | Alert enrichment & threat intel |
| Ubuntu Server | 24.04 LTS | SIEM host |
| Windows 11 Pro | 23H2 | Monitored endpoint |

## Repository Contents

```
.
├── docs/                  # Full technical report (PDF) and diagrams
├── configs/               # ossec.conf integration blocks, n8n docker-compose
├── n8n-workflows/         # Exported n8n workflow JSON
├── test-scripts/          # Attack simulation scripts (T01–T17)
└── README.md
```

> Adjust the tree above to match your actual repo layout before publishing.

## Pipeline Walkthrough (example: EICAR malware test)

| Stage | Component | Latency |
|---|---|---|
| Detection | Wazuh Agent (syscheck) | 0 ms |
| Alert Generation | Wazuh Manager | 200 ms |
| Webhook Forwarding | Wazuh Integratord | 250 ms |
| Parse / Filter | n8n (Code + IF nodes) | 170 ms |
| AI Analysis | Gemini 2.5 Flash | 480 ms |
| Formatting | n8n (Code node) | 50 ms |
| SOC Notification | Slack API | 170 ms |
| Archival | Local file write | 80 ms |
| **Total** | | **~1.4 s** |

## Setup Overview

### 1. Wazuh (All-in-One)
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

### 2. Agent enrollment
```bash
# Linux
sudo WAZUH_MANAGER='192.168.56.10' WAZUH_AGENT_NAME='ubuntu-web-01' dpkg -i wazuh-agent_4.14.5-1_amd64.deb

# Windows (elevated PowerShell)
msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER='192.168.56.10' WAZUH_AGENT_NAME='win-ws-01'
```

### 3. n8n via Docker
```bash
docker run -d --name n8n --restart unless-stopped -p 5678:5678 \
  -e GENERIC_TIMEZONE="Asia/Karachi" -e TZ="Asia/Karachi" \
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true -e N8N_RUNNERS_ENABLED=true \
  -v n8n_data:/home/node/.n8n n8nio/n8n
```

### 4. Wazuh → n8n webhook integration
Add to `ossec.conf`:
```xml
<integration>
  <name>custom-n8n</name>
  <hook_url>http://192.168.56.10:5678/webhook/wazuh-alerts</hook_url>
  <level>7</level>
  <group>syscheck,authentication_failures,windows,ids,web</group>
  <alert_format>json</alert_format>
</integration>
```

### 5. Gemini AI
Configure an n8n HTTP Request node against the Google AI Studio API (Gemini 2.5 Flash, temperature 0.3) with a prompt that returns structured JSON: severity assessment, IOCs, MITRE mapping, and prioritized response actions.

## Results

| Metric | Manual | Automated |
|---|---|---|
| Mean detection-to-response time | ~12 min | ~45 sec |
| Alert analysis time per event | ~8 min | ~1.4 sec |
| MITRE ATT&CK mapping rate | ~60% | 100% |
| IOC extraction completeness | ~45% | ~95% |
| False positive rate | ~15% | ~5% |
| Coverage | Business hours only | 24/7 |

17/17 simulated events (SSH brute force, file tampering, privilege escalation, PowerShell obfuscation, Defender tampering, EICAR drops) were detected with **0 false positives** and **100% MITRE ATT&CK coverage**.

## Limitations

- Gemini free tier is capped at 60 requests/minute
- No automated response (SOAR) yet — pipeline stops at notification
- n8n and Wazuh manager currently share one VM (no HA)
- Alert data leaves the network to Google's API — consider a self-hosted LLM (Ollama + Llama 3) for regulated environments

## Roadmap

- [ ] Replace self-signed TLS certs with a trusted CA
- [ ] Cluster Wazuh for high availability
- [ ] Automate backups of configs, workflows, and indexer snapshots
- [ ] Add SOAR-style automated response (firewall block, EDR isolation, ticketing)
- [ ] Enrich alerts with MISP / VirusTotal / AbuseIPDB
- [ ] Evaluate a self-hosted LLM alternative to Gemini

## References

- [Wazuh Documentation](https://documentation.wazuh.com/current/index.html)
- [n8n Documentation](https://docs.n8n.io/)
- [Google Gemini API Docs](https://ai.google.dev/gemini-api/docs)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)

---

*This project was built in an isolated lab environment for training and educational purposes as part of the ITSimplera Solutions internship program.*

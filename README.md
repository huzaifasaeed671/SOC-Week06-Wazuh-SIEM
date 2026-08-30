# SOC Week 06 — Wazuh SIEM Deployment & Automated Threat Intelligence

## Overview

This project documents the implementation of **Wazuh SIEM**, Windows endpoint integration, File Integrity Monitoring (FIM), security event monitoring, and the planned automation workflow using **n8n + Gemini AI**.

## Objectives

- Deploy Wazuh Manager, Indexer, and Dashboard
- Integrate a Windows 11 endpoint using Wazuh Agent
- Configure and test File Integrity Monitoring (FIM)
- Configure Windows log collection
- Configure and verify Vulnerability Detection
- Generate and analyze security alerts
- Automate alert processing using n8n
- Use Gemini AI for alert analysis and threat-intelligence enrichment
- Evaluate the impact of automation on SOC triage

## Environment

| Component | Details |
|---|---|
| SIEM | Wazuh 4.14.6 |
| Server | Wazuh OVA |
| Virtualization | VirtualBox |
| Endpoint | Windows 11 |
| Automation | n8n |
| AI Analysis | Google Gemini AI |

## Implementation

### 1. Wazuh Deployment

The Wazuh 4.14.6 OVA was deployed in VirtualBox.

The following services were successfully verified:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

The Wazuh Dashboard was accessed successfully through the server IP.

### 2. Windows Endpoint Integration

A Windows 11 host was selected as the monitored endpoint.

The Wazuh Agent was installed and configured to communicate with the Wazuh Manager.

Connectivity was tested on:

- TCP 1514 — Wazuh agent communication
- TCP 1515 — Agent enrollment

The Windows endpoint was successfully enrolled and appeared as **Active** in the Wazuh Dashboard.

### 3. File Integrity Monitoring

A dedicated test directory was created:

```text
C:\Wazuh-FIM-Test

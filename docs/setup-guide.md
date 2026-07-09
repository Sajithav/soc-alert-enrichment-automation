# AI-Powered SOC Alert Enrichment & Automation
## Setup Guide

Disclaimer: This project was developed in a controlled lab environment for cybersecurity learning and demonstration purposes.
All identifiers (IP addresses, usernames, hostnames, credentials, and URLs) have been anonymized or replaced with sample values before publication.

This guide explains how to deploy the SOC alert enrichment workflow using Splunk, n8n, OpenAI, and Slack.

---

## Architecture

```
Kali Linux VM
      │
      ▼
Windows 10 VM
(Windows Event Logs)
      │
      ▼
Splunk Universal Forwarder
      │
      ▼
Splunk Enterprise (Ubuntu VM)
      │
Scheduled Alert
      │
Webhook
      ▼
n8n (Ubuntu VM)
      │
OpenAI API
      │
      ▼
Slack SOC Channel
```

---

# Lab Environment

| Component | Description |
|-----------|-------------|
| Hypervisor | VMware Workstation |
| SIEM | Splunk Enterprise |
| Log Forwarder | Splunk Universal Forwarder |
| Automation | n8n |
| AI | OpenAI GPT |
| Notification | Slack |
| Operating System | Ubuntu Server |
| Endpoint | Windows 10 |
| Attacker Machine | Kali Linux |

---

# Prerequisites

- VMware Workstation
- Ubuntu Server
- Windows 10
- Kali Linux
- Splunk Enterprise
- Splunk Universal Forwarder
- n8n-Webhook
- n8n
- OpenAI API Key
- Slack Workspace
- Slack auth token

---

# Step 1 - Configure Windows Log Forwarding

Install Splunk Universal Forwarder on the Windows 10 VM.

Configure `inputs.conf`:

```ini
[WinEventLog://Application]
disabled = 0
index = automationsoc
current_only = 0

[WinEventLog://System]
disabled = 0
index = automationsoc
current_only = 0

[WinEventLog://Security]
disabled = 0
index = automationsoc
current_only = 0
```

Restart the Universal Forwarder.

---

# Step 2 - Configure Splunk

Create the index:

```
automationsoc
```

Restart the Splunk Universal Forwarder service.

Verify that the following Windows Event Logs are being forwarded to the `automationsoc` index:

- Application
- System
- Security

---

# Step 3 - Create Detection Rule

Create a Scheduled Search using the following SPL:

```spl
index="automationsoc" EventCode="4625"
| stats count as failed_attempts values(user) as attacker values(ComputerName) as affected_device values(EventCode) as eventcode by src_ip
| where failed_attempts > 10
| table src_ip attacker affected_device eventcode
```

Trigger Condition:

- Failed login attempts > 10
- Same source IP

---

# Step 4 - Configure Alert Action

Configure the alert to execute a Webhook Action.

Destination:

```
n8n Production Webhook URL
```

---

# Step 5 - Configure n8n

Workflow:

```
Webhook
      ↓
OpenAI Chat Model
      ↓
Slack
```

The workflow:

- Receives alert payload
- Sends prompt to OpenAI
- Maps alert to MITRE ATT&CK
- Generates investigation guidance
- Sends enriched alert to Slack

---

# Step 6 - Configure OpenAI

The OpenAI model generates:

- MITRE ATT&CK Technique
- Severity Assessment
- IOC Summary
- Investigation Steps
- Containment
- Remediation

---

# Step 7 - Configure Slack

Create an Incoming Webhook using Slack auth token

Configure the Slack node inside n8n.

Alerts are delivered to the SOC channel.

---

# Test the Workflow

Generate multiple failed login attempts from the Kali Linux VM.

Example attack:

- SMB Authentication
- Windows Failed Logon (Event ID 4625)

Expected workflow:

Kali Linux

↓

Windows Event Log

↓

Splunk Alert

↓

n8n Webhook

↓

OpenAI Enrichment

↓

Slack Notification

---

# Project Structure

```
soc-alert-enrichment-automation/
│
├── architecture/
├── docs/
├── n8n/
├── screenshots/
├── splunk/
├── README.md
```

---

# Security Notice

This repository does **not** include:

- API Keys
- Slack Webhook URLs
- n8n Credentials
- Splunk Credentials
- Internal Hostnames
- Production URLs

All IP addresses, usernames, hostnames, and alert data have been anonymized for demonstration purposes.

---

# MITRE ATT&CK

Example Mapping

Technique:

- T1078 – Valid Accounts

Possible Tactics

- Initial Access
- Persistence

---

# Author

**Sajith AV**

Project:
AI-Powered SOC Alert Enrichment & Automation

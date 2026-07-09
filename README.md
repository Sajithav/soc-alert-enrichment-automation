# AI-Powered SOC Alert Enrichment & Automation

This project demonstrates a SOC automation workflow using Splunk, n8n, OpenAI API, Slack, and MITRE ATT&CK.

Disclaimer: This project was developed in a controlled lab environment for cybersecurity learning and demonstration purposes. All identifiers (IP addresses, usernames, hostnames, credentials, and URLs) have been anonymized or replaced with sample values before publication.

## Overview

A Splunk alert detects suspicious activity from Windows security logs. The alert triggers an n8n webhook, which sends the alert details to OpenAI for enrichment. OpenAI maps the activity to MITRE ATT&CK techniques and provides investigation and response steps. The final enriched alert is sent to a Slack SOC channel.

## Tech Stack

- Splunk SIEM
- n8n
- OpenAI API
- Slack Webhooks
- Windows Event Logs
- MITRE ATT&CK

## Workflow

1. Windows logs are forwarded to Splunk.
2. Splunk alert triggers when suspicious activity is detected.
3. Splunk sends alert data to n8n webhook.
4. n8n sends the alert to OpenAI for enrichment.
5. OpenAI returns MITRE ATT&CK mapping and investigation steps.
6. n8n sends the final message to Slack.

## Sample Detection Query

```spl
index="automationsoc" EventCode="4625"
| stats count as failed_attempts values(user) as attacker values(ComputerName) as affected_device values(EventCode) as eventcode by src_ip
| where failed_attempts > 10
|table src_ip,attacker,affected_device,eventcode

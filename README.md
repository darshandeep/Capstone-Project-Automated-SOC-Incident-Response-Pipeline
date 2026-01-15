# Capstone-Project-Automated-SOC-Incident-Response-Pipeline
Capstone: Automated SOC Incident Response Pipeline 🛡️ Automated detection-to-triage using Wazuh, Shuffle SOAR, and TheHive. Orchestrated threat intel via VirusTotal API to enrich IOCs instantly. Reduced MTTR from 10m to &lt;1s, eliminating manual Tier 1 triage for credential dumping (T1003).

Capstone Project: Automated SOC Incident Response Pipeline
Table of Contents
Lab Overview

Infrastructure & Architecture

Endpoint Monitoring (Sysmon)

SIEM Deployment (Wazuh)

The Attack: Mimikatz Execution

SOAR Orchestration (Shuffle)

Threat Intelligence Enrichment (VirusTotal)

Incident Management (TheHive)

Summary & MTTR Metrics

1. Lab Overview
Objective: To design and implement a fully automated Incident Response pipeline that detects credential dumping (Mimikatz), enriches the alert with threat intelligence, and generates a case in a management platform.

The Problem: Manual triage of endpoint alerts is time-consuming, leading to high Mean Time to Respond (MTTR) and "Alert Fatigue".

The Solution: An automated workflow that reduces triage time from minutes to under a second.

2. Infrastructure & Architecture
The environment is split into two primary zones:

Local Infrastructure (On-Premises): Windows 10 Endpoint (Victim) and Wazuh Manager (Ubuntu).

Cloud Services: Shuffle.io (SOAR), VirusTotal (Threat Intel), and TheHive (Incident Management).

3. Endpoint Monitoring (Sysmon)
Objective: Capture deep-level Windows process events to detect malicious binary execution.

Installation: Sysmon64 was installed on the Windows 10 target using a custom configuration file (sysmonconfig.xml).

![1 - sysmon downloaded](image_url) ![2 - verified sysmon installation](image_url)

Verification: Verified the service status via PowerShell and the Windows Services manager.

Logging: Confirmed that Sysmon is successfully writing Event ID 1 (Process Creation) logs to the local Event Viewer.

![12 - Log path](image_url)

4. SIEM Deployment (Wazuh)
Objective: Centralize log collection and trigger alerts based on specific adversary techniques.

Manager Setup: Deployed the Wazuh Manager on an Ubuntu 24.04 LTS instance.

![3 - Wazhu installation](image_url) ![4 - Wazhu Dashboard](image_url)

Agent Deployment: Installed the Wazuh Agent on the Windows victim and established a secure connection to the Manager.

![9 - wazhu agent running](image_url) ![10 - Wazhu Dashboard Showing Active device](image_url)

Rule Creation: Developed a custom detection rule (ID: 100002) to trigger when the mimikatz.exe keyword is detected in Sysmon logs.

![11 - sysmon configured for wazhu](image_url) ![13 - sysmon loging into wazhu](image_url) ![14 - Log every event](image_url)

5. The Attack: Mimikatz Execution
Objective: Simulate a credential dumping attack to validate the detection pipeline.

Execution: Mimikatz was executed in a PowerShell environment to simulate an attacker attempting to dump LSASS memory.

![15 - mimikatz name deception](image_url) ![16 - Executed deception](image_url)

Detection: Wazuh immediately captured the process execution, triggering the "Mimikatz Usage Detected" alert with a Level 15 severity.

6. SOAR Orchestration (Shuffle)
Objective: Automate the data flow between the SIEM and external intelligence tools.

Integration: Configured a Webhook in Wazuh to send JSON alert payloads to Shuffle.

![17 - shuffler integration](image_url)

Workflow: Created a Shuffle workflow to ingest the alert, parse the JSON, and extract the SHA256 file hash of the malicious binary.

7. Threat Intelligence Enrichment (VirusTotal)
Objective: Automatically validate the reputation of detected file hashes.

API Integration: Integrated the VirusTotal API into the Shuffle workflow.

Analysis: The extracted hash was queried against VirusTotal, returning a status 200 (Success) and confirming the file as a known malicious binary.

![18 - virusTotal integrated](image_url)

8. Incident Management (TheHive)
Objective: Centralize all enriched alert data into a single case for analyst review.

![5 - cassandra running](image_url) ![6 - Elastic search running](image_url) ![7 - TheHive running](image_url) ![8 - TheHive Login page](image_url)

Case Creation: Shuffle automatically sent the enriched data to TheHive via API.

![19 - TheHive - deception detected](image_url)

Alert Details: The generated ticket included the source host, the specific Wazuh rule triggered, the malicious hash, and its VirusTotal reputation.

![20 - TheHive- Further information](image_url)

Status: The case was created with a "New" status, ready for immediate investigation.

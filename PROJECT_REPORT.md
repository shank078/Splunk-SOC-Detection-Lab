# 📘 Splunk SOC Detection Lab — Full Technical Project Report

**Author:** Shankar Baral  
**Date:** November 2025  
**Project Type:** SIEM Detection Engineering & Threat Hunting Lab  
**Environment:** Microsoft Azure (Australia East) | Windows Server 2022 | Splunk Enterprise 10.4.0

---

## 📑 Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Objectives](#2-objectives)
3. [Lab Architecture](#3-lab-architecture)
4. [Phase 1 — Infrastructure Deployment](#4-phase-1--infrastructure-deployment)
5. [Phase 2 — Splunk Enterprise Installation](#5-phase-2--splunk-enterprise-installation)
6. [Phase 3 — Data Ingestion Pipeline](#6-phase-3--data-ingestion-pipeline)
7. [Phase 4 — Detection Engineering with SPL](#7-phase-4--detection-engineering-with-spl)
8. [Phase 5 — Attack Correlation & Lifecycle Analysis](#8-phase-5--attack-correlation--lifecycle-analysis)
9. [Phase 6 — Incident Response Action](#9-phase-6--incident-response-action)
10. [Findings & Threat Intelligence](#10-findings--threat-intelligence)
11. [MITRE ATT&CK Mapping](#11-mitre-attck-mapping)
12. [Lessons Learned](#12-lessons-learned)
13. [Future Roadmap](#13-future-roadmap)

---

## 1. Executive Summary

A standalone Splunk Enterprise SIEM lab on Microsoft Azure, run as a full SOC detection workflow. I exposed a Windows Server 2022 host to the internet over RDP (port 3389) so it would attract real brute-force traffic — live data to build and test detections against, rather than a synthetic dataset.

Within about 24 hours, Splunk ingested 28,963+ Windows Security events. My SPL queries pulled 34 failed-login attempts (EventID 4625) from 4 distinct attacker IPs across 3 countries, correlated them to the resulting lockout (EventID 4740) on the `SPLUNKVM` admin account, and rebuilt the attack on a single timeline.

The lab culminated in a live **incident response action** — emergency unlock of the locked admin account via Azure RunCommand — demonstrating practical out-of-band recovery techniques essential to enterprise SOC playbooks.

---

## 2. Objectives

| # | Objective | Outcome |
|---|-----------|---------|
| 1 | Deploy a cloud-hosted Windows Server SIEM lab | ✅ Achieved (Azure VM) |
| 2 | Install & configure Splunk Enterprise standalone | ✅ Achieved (v10.4.0) |
| 3 | Ingest Windows Security Event logs in real-time | ✅ 28,963+ events |
| 4 | Develop SPL detection queries for brute-force attacks | ✅ 3 queries authored |
| 5 | Correlate multi-event attack lifecycle (4625 → 4740) | ✅ 18 correlated events |
| 6 | Perform live incident response on locked account | ✅ Out-of-band unlock |

---

## 3. Lab Architecture

### 3.1 Topology

```
                      ┌─────────────────────────┐
                      │   INTERNET (Attackers)  │
                      └────────────┬────────────┘
                                   │ RDP / 3389
                                   │
            ┌──────────────────────▼──────────────────────┐
            │   AZURE CLOUD — AUSTRALIA EAST              │
            │                                             │
            │   ┌────────────────────────────────────┐    │
            │   │ Resource Group: SOC-Lab-RG         │    │
            │   │                                    │    │
            │   │ ┌────────────────────────────────┐ │    │
            │   │ │ VM: SPLUNK-SRV-01              │ │    │
            │   │ │ OS: Windows Server 2022        │ │    │
            │   │ │ Public IP: 20.92.x.x           │ │    │
            │   │ │                                │ │    │
            │   │ │ ┌──────────────┐  ┌──────────┐ │ │    │
            │   │ │ │ Win Security │─▶│  Splunk  │ │ │    │
            │   │ │ │  Event Logs  │  │Enterprise│ │ │    │
            │   │ │ └──────────────┘  │ :8000    │ │ │    │
            │   │ │                   └────┬─────┘ │ │    │
            │   │ │                        │       │ │    │
            │   │ │                        ▼       │ │    │
            │   │ │                  SPL Searches  │ │    │
            │   │ └────────────────────────────────┘ │    │
            │   └────────────────────────────────────┘    │
            └─────────────────────────────────────────────┘
```

### 3.2 Component Inventory

| Component | Specification |
|-----------|---------------|
| Cloud Provider | Microsoft Azure |
| Region | Australia East |
| VM Name | SPLUNK-SRV-01 |
| OS | Windows Server 2022 Datacenter |
| VM Size | Standard B2s (2 vCPU, 4 GiB RAM) |
| Splunk Version | Enterprise 10.4.0 (Standalone) |
| Web Port | 8000 (localhost) |
| Management Port | 8089 |
| Index | main (default) |
| Source Type | WinEventLog:Security |

---

## 4. Phase 1 — Infrastructure Deployment

A Windows Server 2022 VM was provisioned in Azure under a dedicated resource group. The deployment was completed via the Azure Portal with a public IP assigned to enable RDP access. Network Security Group (NSG) rules were configured to allow inbound TCP/3389 from the internet — deliberately creating an exposed surface to attract real-world attack traffic.

**Deployment Result:** ✅ VM successfully provisioned and online.

> *Reference: `screenshots/01_azure_vm_deployment_success.png`*

RDP access was then established from the local workstation to the cloud VM, granting administrative control over the host for Splunk installation.

> *Reference: `screenshots/02_rdp_server_manager_access.png`*

---

## 5. Phase 2 — Splunk Enterprise Installation

Splunk Enterprise v10.4.0 (64-bit MSI installer) was downloaded directly from splunk.com onto the Windows Server. Installation proceeded with default configurations, with the `admin` user credentials set during the setup wizard.

> *Reference: `screenshots/03_splunk_enterprise_installation.png`*

After installation, the Splunk service auto-started and the Web UI became accessible at `http://localhost:8000`. Login was performed using the admin credentials configured during setup.

> *Reference: `screenshots/04_splunk_web_dashboard_login.png`*

---

## 6. Phase 3 — Data Ingestion Pipeline

### 6.1 Configuration Approach

Splunk was configured to ingest **local Windows Event Logs** using the native `WinEventLog` input modular. This was achieved via:

**Splunk Web → Settings → Data Inputs → Local Event Log Collection → New Local Event Log Collection**

The `Security` log channel was selected to capture all authentication-related events.

> *Reference: `screenshots/05_winventlog_data_input_configured.png`*

### 6.2 Equivalent CLI Configuration (`inputs.conf`)

```ini
[WinEventLog://Security]
disabled = 0
start_from = oldest
current_only = 0
checkpointInterval = 5
index = main
sourcetype = WinEventLog:Security
```

Full configuration file is available at [`configs/inputs.conf`](configs/inputs.conf).

### 6.3 Ingestion Verification

After approximately 18–24 hours of exposure, the index reached **28,963+ events**, confirming a healthy data pipeline.

```spl
index=main sourcetype="WinEventLog:Security" | stats count
```

> *Reference: `screenshots/06_winventlog_ingestion_verification.png`*

---

## 7. Phase 4 — Detection Engineering with SPL

### 7.1 Baseline Verification

Before attack data arrived, the brute-force detection query returned zero results — confirming the detection logic itself was sound and the absence of results was due to a genuinely quiet baseline (not query errors).

> *Reference: `screenshots/07_baseline_detection_no_attacks.png`*

### 7.2 Primary Detection — Failed Logins (Event Code 4625)

**SPL Query:**
```spl
index=main sourcetype="WinEventLog:Security" EventCode=4625
| stats count by Source_Network_Address, Account_Name, ComputerName
| sort - count
| rename count as "Failed_Login_Attempts"
```

**Query Logic Breakdown:**

| Component | Purpose |
|-----------|---------|
| `index=main` | Scope the search to the Security events index (matches inputs.conf) |
| `sourcetype="WinEventLog:Security"` | Restrict to Security log only |
| `EventCode=4625` | Filter to failed-logon events only |
| `stats count by ...` | Group failures by attacker IP, target account, host |
| `sort - count` | Order by most aggressive attacker first |
| `rename` | Human-readable column name |

**Detection Outcome:** 4 distinct attacker IPs surfaced — see Section 10.

> *Reference: `screenshots/08_bruteforce_detection_top_attackers.png`*

Full SPL: [`spl-queries/01_bruteforce_detection.spl`](spl-queries/01_bruteforce_detection.spl)

---

## 8. Phase 5 — Attack Correlation & Lifecycle Analysis

A single 4625 is background noise. Correlating the failed logins (4625) with the lockout they caused (4740) turns it into a confirmed incident with a clear cause and effect.

### 8.1 Correlation Query — Brute-Force → Lockout

**SPL Query:**
```spl
index=main sourcetype="WinEventLog:Security" (EventCode=4625 OR EventCode=4740)
| eval Status=if(EventCode==4625, "Failed Login", "Account Locked")
| table _time, Status, Account_Name, Source_Network_Address, ComputerName
| sort - _time
```

**Logic:**
- Pulls **both** failed login (4625) AND account lockout (4740) events.
- Uses `eval` to translate cryptic Event Codes into human-readable status labels.
- Outputs a unified timeline showing the cause-and-effect chain.

### 8.2 Outcome

The correlation revealed the precise moment the `SPLUNKVM` administrator account was **locked** as a direct result of the preceding flood of failed login attempts — confirming that the attackers' brute-force activity successfully triggered Windows' built-in lockout policy, creating a **self-inflicted Denial of Service** scenario.

> *Reference: `screenshots/09_attack_lifecycle_correlation_timeline.png`*

Full SPL: [`spl-queries/02_attack_lifecycle_correlation.spl`](spl-queries/02_attack_lifecycle_correlation.spl)

---

## 9. Phase 6 — Incident Response Action

### 9.1 The Problem

Once the admin account was locked, **standard RDP login was impossible** — the legitimate administrator could no longer access the host through normal channels.

### 9.2 The Solution — Out-of-Band Recovery via Azure RunCommand

Azure's **VM RunCommand** feature was used to execute PowerShell directly on the host *without* requiring an interactive login. This is a critical SOC playbook capability — equivalent to using a "console" or "out-of-band management interface" in on-premise environments.

```powershell
net user SPLUNKVM <NEW_SECURE_PASSWORD>
```

The command executed successfully, resetting the password and unlocking the account.

> *Reference: `screenshots/10_incident_response_account_unlock.png`*

### 9.3 Outcome

Administrative access was restored without rebuilding the VM or losing any Splunk data. This validated the importance of maintaining out-of-band management channels in every cloud SOC architecture.

---

## 10. Findings & Threat Intelligence

### 10.1 Attacker Inventory

| Source IP | Geolocation (est.) | Failed Attempts | Targeted Accounts |
|-----------|---------------------|------------------|---------------------|
| `163.47.70.77` | India (est.) | 16 | `SPLUNKVM`, admin email |
| `190.2.155.233` | Panama (est.) | 12 | `SPLUNKVM` |
| `195.242.214.36` | Europe (est.) | 4 | `SPLUNKVM` |
| `86.254.114.246` | France (est.) | 2 | `Test` |

*Counts are per source IP and sum to 34 failed-login (4625) events. Some attempts hit a blank/unknown username (pre-auth), so the count against named accounts is lower.*

### 10.2 Key Observations

- **Geographic Diversity:** Attacks originated from 4 different countries, indicating either a **distributed botnet** or **independent opportunistic attackers** scanning the public IPv4 space.
- **Username Targeting:** Attackers heavily targeted `SPLUNKVM` (the actual admin username), suggesting **OSINT reconnaissance** preceded the attacks.
- **Speed of Discovery:** The host was discovered and attacked within hours of deployment — confirming that **any unprotected RDP port on the public internet WILL be attacked**, often within minutes.

---

## 11. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique | Evidence |
|--------|--------------|-----------|----------|
| **Initial Access** | T1078 | Valid Accounts | Attackers attempted known usernames |
| **Credential Access** | T1110 | Brute Force | Multiple failed logins per IP |
| **Credential Access** | T1110.001 | Password Guessing | Targeting specific accounts |
| **Discovery** | T1018 | Remote System Discovery | Initial port scan / RDP probing |
| **Impact** | T1531 | Account Access Removal | Caused admin lockout (self-DoS) |

---

## 12. Lessons Learned

1. **Public RDP gets found fast.** Even in a lab, an exposed RDP port drew attacks within hours. In a real environment this belongs behind Azure Bastion, a VPN, or Just-in-Time (JIT) access.
2. **Account lockout policies are double-edged swords.** They block attackers but also create self-inflicted DoS. SOC playbooks must include out-of-band recovery.
3. **Correlation > Detection.** Identifying a 4625 alone is "noise" — correlating it with 4740 turns noise into a **confirmed incident**.
4. **SIEM is only as good as the data feeding it.** A misconfigured `inputs.conf` would have rendered the entire detection chain useless.
5. **Threat actors are opportunistic.** No one targeted me specifically — they scanned ranges and found an open door. Defence-in-depth at every layer is non-negotiable.

---

## 13. Future Roadmap

| Enhancement | Description | Priority |
|-------------|-------------|----------|
| Threat Intel Lookup | Integrate AbuseIPDB / VirusTotal via Splunk Lookups | High |
| Real-time Alerting | Convert SPL searches into Scheduled Alerts | High |
| SOAR Integration | Auto-block IPs via Azure NSG using webhook actions | Medium |
| Custom Dashboard | Build XML dashboard for live SOC overview | Medium |
| Distributed Splunk | Migrate to Indexer + Search Head + Universal Forwarder | Low |
| MITRE Visualization | Map detections directly to ATT&CK matrix dashboard | Medium |
| Sigma Rule Conversion | Convert SPL queries to Sigma rules for portability | Low |

---

**— End of Report —**

> 📂 Return to [README.md](README.md) | View [SPL Queries](spl-queries/) | View [Screenshots](screenshots/)

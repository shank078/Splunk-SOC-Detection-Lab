# 🛡️ Splunk SOC Detection Lab — Brute-Force Attack Correlation

> Enterprise-grade Security Operations Center (SOC) lab demonstrating end-to-end detection engineering using Splunk Enterprise SIEM, Windows Security Event logs, and custom SPL queries to identify and correlate brute-force attacks against a cloud-hosted Windows Server.

![Splunk](https://img.shields.io/badge/Splunk-000000?style=for-the-badge&logo=splunk&logoColor=white)
![Azure](https://img.shields.io/badge/Microsoft_Azure-0089D6?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Windows Server](https://img.shields.io/badge/Windows_Server_2022-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![SIEM](https://img.shields.io/badge/SIEM-Detection_Engineering-red?style=for-the-badge)
![Blue Team](https://img.shields.io/badge/Blue_Team-SOC_Analyst-blue?style=for-the-badge)

---

## 📌 Project Overview

This lab simulates a real-world SOC analyst workflow — from deploying SIEM infrastructure to detecting active brute-force attacks against an internet-exposed Windows Server. The project demonstrates:

- 🏗️ Standalone Splunk Enterprise SIEM deployment on Azure
- 📡 Real-time ingestion of Windows Security Event logs
- 🔍 Custom SPL (Search Processing Language) queries for threat detection
- 🔗 Multi-event correlation (Failed Login → Account Lockout lifecycle)
- 🌍 Geographic threat actor identification across multiple international IPs
- 🚨 Live incident response (emergency account unlock via Azure RunCommand)

---

## 🎯 Key Achievements

| Metric | Result |
|--------|--------|
| Events Ingested | 28,963+ Windows Security Events |
| Attack Detection | 17+ unique failed login attempts identified |
| Threat Actors Isolated | 4 distinct malicious IPs across 3 countries |
| Correlation Built | Failed Login (4625) ↔ Account Lockout (4740) |
| Incident Response | Successful emergency unlock via Azure CLI |

---

## 🚀 Phase 1: Infrastructure Deployment

### Azure VM Provisioning
A Windows Server 2022 VM was deployed in **Azure (Australia East)** as the lab environment.

![Azure VM Deployment](screenshots/01_azure_vm_deployment_success.png)

### Remote Access & Server Manager
RDP access was established to `SPLUNK-SRV-01` for hands-on configuration.

![RDP Access](screenshots/02_rdp_server_manager_access.png)

---

## ⚙️ Phase 2: Splunk Enterprise Setup

### Installation
Splunk Enterprise v10.4.0 was downloaded and installed natively on the Windows Server.

![Splunk Installation](screenshots/03_splunk_enterprise_installation.png)

### Web Console Access
The Splunk Web UI was successfully launched on `localhost:8000` with administrator credentials.

![Splunk Web Dashboard](screenshots/04_splunk_web_dashboard_login.png)

---

## 📡 Phase 3: Data Ingestion

### Windows Event Log Input
Configured Splunk to ingest **local Windows Security Event logs** in real-time using the native `WinEventLog:Security` source type.

![Data Input Configured](screenshots/05_winventlog_data_input_configured.png)

### Ingestion Verification
After enabling RDP-exposed traffic and waiting for attackers to discover the host, **28,963+ events** were ingested.

![Ingestion Verified](screenshots/06_winventlog_ingestion_verification.png)

---

## 🕵️ Phase 4: Detection Engineering

### Baseline (Pre-Attack State)
Initial detection logic returned **zero results**, confirming no brute-force activity before exposure.

![Baseline No Attacks](screenshots/07_baseline_detection_no_attacks.png)

### 🎯 Brute-Force Detection (Event Code 4625)

**SPL Query:**
```spl
index=* sourcetype="WinEventLog:Security" EventCode=4625
| stats count by Source_Network_Address, Account_Name, ComputerName
| sort - count
| rename count as "Failed_Login_Attempts"
```

**Result:** Successfully identified **17 failed login attempts** from 4 distinct attacker IPs targeting multiple account names (including `SPLUNKVM`, `Test`, and admin emails).

![Top Attackers Detected](screenshots/08_bruteforce_detection_top_attackers.png)

| Source IP | Country (est.) | Failed Attempts | Targeted Accounts |
|-----------|----------------|------------------|-------------------|
| `163.47.70.77` | 🇮🇳 India | 16 | SPLUNKVM, admin email |
| `190.2.155.233` | 🇵🇦 Panama | 12 | SPLUNKVM |
| `195.242.214.36` | 🇪🇺 Europe | 4 | SPLUNKVM |
| `86.254.114.246` | 🇫🇷 France | 2 | Test |

---

## 🔗 Phase 5: Incident Correlation

### Brute-Force → Account Lockout Lifecycle (4625 + 4740)

**SPL Query:**
```spl
index=* sourcetype="WinEventLog:Security" (EventCode=4625 OR EventCode=4740)
| eval Status=if(EventCode==4625, "Failed Login", "Account Locked")
| table _time, Status, Account_Name, Source_Network_Address, ComputerName
| sort - _time
```

**Result:** Built a clear chronological narrative showing **18 correlated events**, including the exact moment the SPLUNK-SRV-01$ account was locked due to repeated failures.

![Attack Lifecycle Timeline](screenshots/09_attack_lifecycle_correlation_timeline.png)

---

## 🚨 Phase 6: Incident Response

When the attack succeeded in locking out the legitimate admin account, an **out-of-band recovery** was performed using **Azure RunCommand** to remotely execute a PowerShell unlock — bypassing the locked login screen.

```powershell
net user SPLUNKVM [REDACTED_NEW_PASSWORD]
```

![Account Unlock](screenshots/10_incident_response_account_unlock.png)

> 💡 **Lesson Learned:** Account lockouts caused by attackers create a self-inflicted denial-of-service. Out-of-band recovery channels (Azure RunCommand, Bastion, Serial Console) are essential for SOC playbooks.

---

## 🧠 Skills Demonstrated

- ✅ **SIEM Engineering** — Standalone Splunk deployment & configuration
- ✅ **Detection Engineering** — Custom SPL development for threat hunting
- ✅ **Log Analysis** — Windows Security Event log parsing (4625, 4740, 4688)
- ✅ **Threat Correlation** — Multi-event lifecycle reconstruction
- ✅ **Cloud Security** — Azure VM hardening & emergency recovery
- ✅ **Incident Response** — Out-of-band remediation playbooks

---

## 📂 Repository Contents

| File / Folder | Description |
|---------------|-------------|
| [`PROJECT_REPORT.md`](PROJECT_REPORT.md) | Full technical project report |
| [`spl-queries/`](spl-queries/) | All SPL detection queries with comments |
| [`configs/`](configs/) | Splunk `inputs.conf` and related configs |
| [`docs/`](docs/) | Architecture notes, methodology, lessons learned |
| [`screenshots/`](screenshots/) | All visual proof of work |

---

## 🔮 Future Enhancements

- [ ] Integrate **AbuseIPDB / VirusTotal** threat intel feeds via Splunk lookups
- [ ] Build dynamic XML-based **SOC dashboard** for real-time monitoring
- [ ] Configure **Splunk Alert Actions** → auto-push malicious IPs to Azure NSG deny list
- [ ] Migrate to **distributed Splunk architecture** (Indexer + Search Head + UF)
- [ ] Add **MITRE ATT&CK mapping** (T1110 Brute Force, T1078 Valid Accounts)

---

## 👤 About the Author

### **Shankar Baral**
**Junior Cyber Security Analyst & IT Support Specialist**  
🎓 Master of Cyber Security (GPA: 4.92) • 🇦🇺 Australian Permanent Resident • 📍 Canberra, ACT

---

I am an **IT Support Specialist and Junior Cyber Security Analyst** based in Canberra, ACT. I hold a **Master of Information Technology (Cyber Security)** from **Charles Sturt University**, graduating with a highly distinguished **4.92 GPA**.

Currently, I operate at **Extratech**, where I manage high-volume enterprise escalations, administer **Microsoft Intune and Entra ID** identity lifecycles, and monitor perimeter defenses. I specialize in bridging daily IT operations with proactive security measures, focusing on **Azure/M365 administration** and aligning environments with **ASD Essential Eight** frameworks.

Beyond enterprise operations, I actively design and deploy cloud-native security pipelines. My hands-on engineering experience includes:

- 🌐 Deploying **global threat intelligence dashboards**
- 💻 Configuring **zero-touch Windows Autopilot** environments
- 🛡️ Building **Microsoft Sentinel SIEM architectures** capable of intercepting and visualizing thousands of live brute-force attacks

I am currently advancing my SOC capabilities by actively pursuing the **Blue Team Level 1 (BTL1)** and **Microsoft SC-200** certifications.

---

### 🔗 Connect With Me

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/shankarbaral1/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/shank078)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:shankarbaral1@gmail.com)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-212C42?style=for-the-badge&logo=tryhackme&logoColor=red)](https://tryhackme.com/)

---

⭐ **If you found this project helpful or inspiring, please consider giving it a star!** ⭐
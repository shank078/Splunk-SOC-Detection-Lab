{\rtf1\ansi\ansicpg1252\cocoartf2870
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;\f1\fnil\fcharset0 AppleColorEmoji;\f2\fnil\fcharset0 LucidaGrande;
\f3\fnil\fcharset128 HiraginoSans-W3;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\paperw11900\paperh16840\margl1440\margr1440\vieww37860\viewh21260\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 # 
\f1 \uc0\u55357 \u57057 \u65039 
\f0  Splunk SOC Detection Lab \'97 Brute-Force Attack Correlation\
\
> **Enterprise-grade Security Operations Center (SOC) lab** demonstrating end-to-end detection engineering using **Splunk Enterprise SIEM**, Windows Security Event logs, and custom SPL queries to identify and correlate brute-force attacks against a cloud-hosted Windows Server.\
\
![Splunk](https://img.shields.io/badge/Splunk-Enterprise-000000?style=for-the-badge&logo=splunk&logoColor=white)\
![Azure](https://img.shields.io/badge/Microsoft_Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)\
![Windows Server](https://img.shields.io/badge/Windows_Server_2022-0078D6?style=for-the-badge&logo=windows&logoColor=white)\
![SIEM](https://img.shields.io/badge/SIEM-Detection_Engineering-red?style=for-the-badge)\
![Blue Team](https://img.shields.io/badge/Blue_Team-Threat_Hunting-1f6feb?style=for-the-badge)\
\
---\
\
## 
\f1 \uc0\u55357 \u56524 
\f0  Project Overview\
\
This lab simulates a **real-world SOC analyst workflow** \'97 from deploying SIEM infrastructure to detecting active brute-force attacks against an internet-exposed Windows Server. The project demonstrates:\
\
- 
\f1 \uc0\u55356 \u57303 \u65039 
\f0  Standalone Splunk Enterprise SIEM deployment on Azure\
- 
\f1 \uc0\u55357 \u56545 
\f0  Real-time ingestion of Windows Security Event logs\
- 
\f1 \uc0\u55357 \u56589 
\f0  Custom **SPL (Search Processing Language)** queries for threat detection\
- 
\f1 \uc0\u55357 \u56599 
\f0  Multi-event correlation (Failed Login 
\f2 \uc0\u8594 
\f0  Account Lockout lifecycle)\
- 
\f1 \uc0\u55356 \u57101 
\f0  Geographic threat actor identification across multiple international IPs\
- 
\f1 \uc0\u55357 \u57000 
\f0  Live incident response (emergency account unlock via Azure RunCommand)\
\
---\
\
## 
\f1 \uc0\u55356 \u57263 
\f0  Key Achievements\
\
| Metric | Result |\
|--------|--------|\
| **Events Ingested** | 28,963+ Windows Security Events |\
| **Attack Detection** | 17+ unique failed login attempts identified |\
| **Threat Actors Isolated** | 4 distinct malicious IPs across 3 countries |\
| **Correlation Built** | Failed Login (4625) 
\f3 \uc0\u8596 
\f0  Account Lockout (4740) |\
| **Incident Response** | Successful emergency unlock via Azure CLI |\
\
---\
\
## 
\f1 \uc0\u55356 \u57303 \u65039 
\f0  Architecture\
\
```\
\uc0\u9484 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9488 \
\uc0\u9474                     AZURE CLOUD (Australia East)             \u9474 \
\uc0\u9474                                                              \u9474 \
\uc0\u9474    \u9484 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9488       \u9474 \
\uc0\u9474    \u9474    Windows Server 2022 VM (SPLUNK-SRV-01)         \u9474       \u9474 \
\uc0\u9474    \u9474    \u9484 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9488   \u9484 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9488    \u9474       \u9474 \
\uc0\u9474    \u9474    \u9474   Windows Security  \u9474 \u9472 
\f2 \uc0\u9654 
\f0 \uc0\u9474  Splunk Enterprise\u9474    \u9474       \u9474 \
\uc0\u9474    \u9474    \u9474   Event Logs        \u9474   \u9474    (Standalone)   \u9474    \u9474       \u9474 \
\uc0\u9474    \u9474    \u9474   (4625, 4740, 4688)\u9474   \u9474    Port 8000      \u9474    \u9474       \u9474 \
\uc0\u9474    \u9474    \u9492 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9496   \u9492 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 
\f3 \'84\'a6
\f0 \uc0\u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9496    \u9474       \u9474 \
\uc0\u9474    \u9474             
\f3 \'81\'a3
\f0                         \uc0\u9474              \u9474       \u9474 \
\uc0\u9474    \u9474             \u9474                         
\f2 \uc0\u9660 
\f0              \uc0\u9474       \u9474 \
\uc0\u9474    \u9474    Public RDP (3389)         SPL Search Head      \u9474       \u9474 \
\uc0\u9474    \u9492 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 
\f3 \'84\'a6
\f0 \uc0\u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9496       \u9474 \
\uc0\u9492 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 
\f3 \'84\'a9
\f0 \uc0\u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9496 \
               \uc0\u9474 \
               \uc0\u9474   Brute-Force Attacks\
               \uc0\u9474 \
       \uc0\u9484 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 
\f3 \'84\'a8
\f0 \uc0\u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 
\f3 \'84\'a6
\f0 \uc0\u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 
\f3 \'84\'a6
\f0 \uc0\u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9472 \u9488 \
       \uc0\u9474                 \u9474            \u9474               \u9474 \
   163.47.70.77   190.2.155.233  195.242.214.36  86.254.114.246\
```\
\
---\
\
## 
\f1 \uc0\u55357 \u56960 
\f0  Phase 1: Infrastructure Deployment\
\
### Azure VM Provisioning\
A Windows Server 2022 VM was deployed in **Azure (Australia East)** as the lab environment.\
\
![Azure VM Deployment](screenshots/01_azure_vm_deployment_success.png)\
\
### Remote Access & Server Manager\
RDP access was established to `SPLUNK-SRV-01` for hands-on configuration.\
\
![RDP Access](screenshots/02_rdp_server_manager_access.png)\
\
---\
\
## 
\f1 \uc0\u9881 \u65039 
\f0  Phase 2: Splunk Enterprise Setup\
\
### Installation\
Splunk Enterprise v10.4.0 was downloaded and installed natively on the Windows Server.\
\
![Splunk Installation](screenshots/03_splunk_enterprise_installation.png)\
\
### Web Console Access\
The Splunk Web UI was successfully launched on `localhost:8000` with administrator credentials.\
\
![Splunk Web Dashboard](screenshots/04_splunk_web_dashboard_login.png)\
\
---\
\
## 
\f1 \uc0\u55357 \u56545 
\f0  Phase 3: Data Ingestion\
\
### Windows Event Log Input\
Configured Splunk to ingest **local Windows Security Event logs** in real-time using the native `WinEventLog:Security` source type.\
\
![Data Input Configured](screenshots/05_winventlog_data_input_configured.png)\
\
### Ingestion Verification\
After enabling RDP-exposed traffic and waiting for attackers to discover the host, **28,963+ events** were ingested.\
\
![Ingestion Verified](screenshots/06_winventlog_ingestion_verification.png)\
\
---\
\
## 
\f1 \uc0\u55357 \u56693 \u65039 
\f0  Phase 4: Detection Engineering\
\
### Baseline (Pre-Attack State)\
Initial detection logic returned **zero results**, confirming no brute-force activity before exposure.\
\
![Baseline No Attacks](screenshots/07_baseline_detection_no_attacks.png)\
\
### 
\f1 \uc0\u55356 \u57263 
\f0  Brute-Force Detection (Event Code 4625)\
\
**SPL Query:**\
```spl\
index=* sourcetype="WinEventLog:Security" EventCode=4625 \
| stats count by Source_Network_Address, Account_Name, ComputerName \
| sort - count \
| rename count as "Failed_Login_Attempts"\
```\
\
**Result:** Successfully identified **17 failed login attempts** from 4 distinct attacker IPs targeting multiple account names (including `SPLUNKVM`, `Test`, and admin emails).\
\
![Top Attackers Detected](screenshots/08_bruteforce_detection_top_attackers.png)\
\
| Source IP | Country (est.) | Failed Attempts | Targeted Accounts |\
|-----------|----------------|------------------|-------------------|\
| `163.47.70.77` | 
\f1 \uc0\u55356 \u56814 \u55356 \u56819 
\f0  India | 16 | SPLUNKVM, admin email |\
| `190.2.155.233` | 
\f1 \uc0\u55356 \u56821 \u55356 \u56806 
\f0  Panama | 12 | SPLUNKVM |\
| `195.242.214.36` | 
\f1 \uc0\u55356 \u56810 \u55356 \u56826 
\f0  Europe | 4 | SPLUNKVM |\
| `86.254.114.246` | 
\f1 \uc0\u55356 \u56811 \u55356 \u56823 
\f0  France | 2 | Test |\
\
---\
\
## 
\f1 \uc0\u55357 \u56599 
\f0  Phase 5: Incident Correlation\
\
### Brute-Force 
\f2 \uc0\u8594 
\f0  Account Lockout Lifecycle (4625 + 4740)\
\
**SPL Query:**\
```spl\
index=* sourcetype="WinEventLog:Security" (EventCode=4625 OR EventCode=4740) \
| eval Status=if(EventCode==4625, "Failed Login", "Account Locked") \
| table _time, Status, Account_Name, Source_Network_Address, ComputerName \
| sort - _time\
```\
\
**Result:** Built a clear chronological narrative showing **18 correlated events**, including the exact moment the SPLUNK-SRV-01$ account was locked due to repeated failures.\
\
![Attack Lifecycle Timeline](screenshots/09_attack_lifecycle_correlation_timeline.png)\
\
---\
\
## 
\f1 \uc0\u55357 \u57000 
\f0  Phase 6: Incident Response\
\
When the attack succeeded in locking out the legitimate admin account, an **out-of-band recovery** was performed using **Azure RunCommand** to remotely execute a PowerShell unlock \'97 bypassing the locked login screen.\
\
```powershell\
net user SPLUNKVM [REDACTED_NEW_PASSWORD]\
```\
\
![Account Unlock](screenshots/10_incident_response_account_unlock.png)\
\
> 
\f1 \uc0\u55357 \u56481 
\f0  **Lesson Learned:** Account lockouts caused by attackers create a self-inflicted denial-of-service. Out-of-band recovery channels (Azure RunCommand, Bastion, Serial Console) are essential for SOC playbooks.\
\
---\
\
## 
\f1 \uc0\u55358 \u56800 
\f0  Skills Demonstrated\
\
- 
\f1 \uc0\u9989 
\f0  **SIEM Engineering** \'97 Standalone Splunk deployment & configuration\
- 
\f1 \uc0\u9989 
\f0  **Detection Engineering** \'97 Custom SPL development for threat hunting\
- 
\f1 \uc0\u9989 
\f0  **Log Analysis** \'97 Windows Security Event log parsing (4625, 4740, 4688)\
- 
\f1 \uc0\u9989 
\f0  **Threat Correlation** \'97 Multi-event lifecycle reconstruction\
- 
\f1 \uc0\u9989 
\f0  **Cloud Security** \'97 Azure VM hardening & emergency recovery\
- 
\f1 \uc0\u9989 
\f0  **Incident Response** \'97 Out-of-band remediation playbooks\
\
---\
\
## 
\f1 \uc0\u55357 \u56538 
\f0  Repository Contents\
\
| File / Folder | Description |\
|---------------|-------------|\
| [`PROJECT_REPORT.md`](PROJECT_REPORT.md) | Full technical project report |\
| [`spl-queries/`](spl-queries/) | All SPL detection queries with comments |\
| [`configs/`](configs/) | Splunk `inputs.conf` and related configs |\
| [`docs/`](docs/) | Architecture notes, methodology, lessons learned |\
| [`screenshots/`](screenshots/) | All visual proof of work |\
\
---\
\
## 
\f1 \uc0\u55357 \u56622 
\f0  Future Enhancements\
\
- [ ] Integrate **AbuseIPDB / VirusTotal** threat intel feeds via Splunk lookups\
- [ ] Build dynamic XML-based **SOC dashboard** for real-time monitoring\
- [ ] Configure **Splunk Alert Actions** 
\f2 \uc0\u8594 
\f0  auto-push malicious IPs to Azure NSG deny list\
- [ ] Migrate to **distributed Splunk architecture** (Indexer + Search Head + UF)\
- [ ] Add **MITRE ATT&CK mapping** (T1110 Brute Force, T1078 Valid Accounts)\
\
---\
\
## \uc0\u55357 \u56420  About the Author\
\
### **Shankar Baral**\
**Junior Cyber Security Analyst & IT Support Specialist**  \
\uc0\u55356 \u57235  Master of Cyber Security (GPA: 4.92) \'95 \u55356 \u56806 \u55356 \u56826  Australian Permanent Resident \'95 \u55357 \u56525  Canberra, ACT\
\
---\
\
I am an **IT Support Specialist and Junior Cyber Security Analyst** based in Canberra, ACT. I hold a **Master of Information Technology (Cyber Security)** from **Charles Sturt University**, graduating with a highly distinguished **4.92 GPA**.\
\
Currently, I operate at **Extratech**, where I manage high-volume enterprise escalations, administer **Microsoft Intune and Entra ID** identity lifecycles, and monitor perimeter defenses. I specialize in bridging daily IT operations with proactive security measures, focusing on **Azure/M365 administration** and aligning environments with **ASD Essential Eight** frameworks.\
\
Beyond enterprise operations, I actively design and deploy cloud-native security pipelines. My hands-on engineering experience includes:\
\
- \uc0\u55356 \u57104  Deploying **global threat intelligence dashboards**\
- \uc0\u55357 \u56507  Configuring **zero-touch Windows Autopilot** environments\
- \uc0\u55357 \u57057 \u65039  Building **Microsoft Sentinel SIEM architectures** capable of intercepting and visualizing thousands of live brute-force attacks\
\
I am currently advancing my SOC capabilities by actively pursuing the **Blue Team Level 1 (BTL1)** and **Microsoft SC-200** certifications.\
\
---\
\
### \uc0\u55357 \u56599  Connect With Me\
\
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/shankarbaral1/)\
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/shank078)\
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:shankarbaral1@gmail.com)\
[![TryHackMe](https://img.shields.io/badge/TryHackMe-212C42?style=for-the-badge&logo=tryhackme&logoColor=red)](https://tryhackme.com/)}
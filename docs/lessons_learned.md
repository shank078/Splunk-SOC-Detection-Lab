# 💡 Lessons Learned — Splunk SOC Detection Lab

**Author:** Shankar Baral  
**Project:** Splunk SOC Detection Lab — Brute-Force Attack Correlation

---

## 🎓 Executive Reflection

This lab was designed not just to "build something that works," but to **internalize the mindset of a SOC analyst**. The lessons below are the most valuable takeaways — practical insights gained from running a live, internet-exposed SIEM under active attack.

---

## 🔥 Top 10 Lessons Learned

### 1. **Exposure Equals Compromise**
The single most important takeaway: **any port exposed to the public internet WILL be discovered and attacked**, often within hours. There is no "obscure" anymore — automated scanners map the entire IPv4 space daily. In production, RDP must NEVER be directly internet-facing.

### 2. **Account Lockout Policies Are a Double-Edged Sword**
Windows' default lockout policy successfully blocked the attackers — but it also locked out the legitimate administrator. This **self-inflicted Denial of Service** is a documented attack pattern. Every SOC playbook must include **out-of-band recovery procedures**.

### 3. **Correlation > Single-Event Detection**
A standalone Event 4625 is low-confidence noise. Correlating it with Event 4740 transforms it into a **confirmed incident with a clear cause-and-effect chain**. Junior analysts focus on detection; senior analysts focus on **narrative**.

### 4. **SIEM Is Only as Good as Its Inputs**
The first version of `inputs.conf` had `current_only = 1` — meaning Splunk *missed* the historical attack data accumulated before installation. Fixing this with `current_only = 0` and `start_from = oldest` saved the lab. **Always validate your data pipeline before trusting your detections.**

### 5. **Threat Actors Are Opportunistic, Not Targeted**
None of the attackers knew who I was. They scanned IP ranges, found an open RDP port, and tried common credentials. This is the **default behavior of the internet** — defending against it is non-negotiable for every host, every day.

### 6. **Splunk's `WinEventLog` Sourcetype Is Powerful but Verbose**
Field extraction (`Source_Network_Address`, `Account_Name`) required careful trial-and-error. Real-world deployments should use **CIM (Common Information Model)** compliant data models to standardize fields across multiple log sources.

### 7. **Documentation Is a Detection Skill**
Writing this report forced me to *re-examine* every query, every event code, and every decision. Documenting the "why" exposed gaps in my own understanding — and is the single biggest differentiator between a junior and senior SOC analyst.

### 8. **MITRE ATT&CK Mapping Adds Context**
Without ATT&CK, "brute force" is just a buzzword. With ATT&CK (T1110.001), it's a documented technique with known mitigations, detections, and threat-actor associations. **Always map detections to a framework.**

### 9. **Cloud SOC Requires Cloud-Native Tools**
The Azure RunCommand feature saved the lab. In a pure on-premise mindset, I would have had to physically/remotely reboot the server and use Safe Mode. **Cloud SOC engineers must master cloud-native admin tools (RunCommand, Bastion, Serial Console).**

### 10. **Real Data > Synthetic Data**
This lab attracted authentic, organic attacks within hours — generating data that no synthetic attack simulation could replicate. **Internet-exposed honeypot labs (with proper isolation) are some of the highest-value learning environments a SOC analyst can build.**

---

## 🧠 Skills Reinforced

| Domain | Specific Skills Gained |
|--------|-----------------------|
| **SIEM Engineering** | Standalone Splunk deployment, `inputs.conf` tuning, index management |
| **Detection Engineering** | SPL query development, baseline validation, correlation logic |
| **Threat Hunting** | Pivot-based investigation (IP → user → host), timeline reconstruction |
| **Cloud Security** | Azure VM hardening, NSG rules, RunCommand, out-of-band recovery |
| **Documentation** | Technical writing, executive summary, MITRE mapping |
| **Mindset** | Adversary thinking, blast-radius reasoning, fail-safe design |

---

## ❌ Mistakes Made (And Fixed)

| Mistake | Impact | Resolution |
|---------|--------|------------|
| Initial `current_only = 1` config | Lost historical events | Changed to `0` + `start_from = oldest` |
| Did not change default RDP port | Faster discovery by scanners | (Intentionally left — this was the lab) |
| Used weak admin password | Allowed attempts to succeed-ish | Replaced with complex 20-char password |
| No alerting initially configured | Manual searches required | Future enhancement: Scheduled Alerts |

---

## 🚀 What I Would Do Differently Next Time

1. **Deploy a Splunk Universal Forwarder** instead of native install — closer to real enterprise architecture.
2. **Enable Sysmon** before exposure — would have captured rich process/command-line telemetry (Event ID 1, 3, 11).
3. **Configure Threat Intel Lookups (AbuseIPDB)** upfront — would have auto-tagged malicious IPs at search time.
4. **Build the SOC Dashboard before exposure** — would have visualized the attack live as it happened.
5. **Use Azure JIT-VM Access** for legitimate admin RDP — eliminating the lockout-DoS risk entirely.

---

## 🎯 Real-World Transferable Value

The skills practiced in this lab map **directly** to enterprise SOC roles:

- ✅ Junior SOC Analyst — Tier 1 alert triage (failed login detection)
- ✅ SOC Analyst — Multi-event correlation, IR playbook execution
- ✅ Detection Engineer — Custom SPL development, false-positive tuning
- ✅ Cloud Security Engineer — Azure-native recovery procedures
- ✅ Threat Hunter — Pivot-based hunting using indexed log data

---

> 📂 Return to [README.md](../README.md) | [Methodology](methodology.md)
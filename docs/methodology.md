# 🧭 Methodology — Splunk SOC Detection Lab

**Author:** Shankar Baral  
**Project:** Splunk SOC Detection Lab — Brute-Force Attack Correlation

---

## 🎯 Approach

This lab followed a **structured, phase-based detection engineering methodology**, mirroring how real-world SOC teams build and validate new detection content. Each phase was deliberately scoped to produce a single, verifiable outcome before moving to the next.

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   PHASE 1    │─▶│   PHASE 2    │─▶│   PHASE 3    │─▶│   PHASE 4    │
│  Provision   │  │   Install    │  │   Ingest     │  │   Detect     │
│  Infra       │  │   Splunk     │  │   Logs       │  │   Threats    │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
                                                              │
                          ┌───────────────────────────────────┘
                          ▼
                  ┌──────────────┐  ┌──────────────┐
                  │   PHASE 5    │─▶│   PHASE 6    │
                  │  Correlate   │  │   Respond    │
                  │  Lifecycle   │  │   to Inc.    │
                  └──────────────┘  └──────────────┘
```

---

## 📐 Detection Engineering Framework

Each SPL detection was developed using a **5-step framework**:

### Step 1 — Define the Use Case
> *"What attacker behavior am I trying to detect?"*

Example: *Detect external IPs attempting password-guessing attacks against Windows accounts.*

### Step 2 — Identify the Data Source
> *"Where would evidence of this behavior appear?"*

→ `WinEventLog:Security`, specifically Event Code 4625.

### Step 3 — Build the Logic
> *"What SPL pattern isolates the signal from the noise?"*

→ `stats count by Source_IP, Account` filtered by EventCode and sorted descending.

### Step 4 — Validate Against Baseline
> *"Does the query produce zero results when no attack is occurring?"*

→ Confirmed by running detection during pre-attack window — returned empty result set.

### Step 5 — Validate Against True Positive
> *"Does the query fire when the attack is occurring?"*

→ Validated post-attack — query returned the 34 failed-login (4625) events from the 4 attacker IPs.

---

## 🔗 Correlation Methodology

Correlation was built using Splunk's `eval` and `OR` operators to fuse multiple event types into a single timeline view. The approach was inspired by the **MITRE ATT&CK Kill Chain model**:

| Stage | ATT&CK Tactic | Event Code | Detection |
|-------|--------------|------------|-----------|
| Discovery | TA0007 | (Network scan — inferred) | NSG flow logs (not in scope) |
| Credential Access | TA0006 | 4625 | Brute-force detection |
| Impact | TA0040 | 4740 | Account lockout (self-DoS) |

---

## 🔬 Validation Standards

Every detection query was tested against **three states**:

| State | Expected Result | Outcome |
|-------|----------------|---------|
| Pre-Attack (Baseline) | Zero results | ✅ Confirmed |
| During Attack (Live) | Multiple high-confidence hits | ✅ Confirmed |
| Post-Incident (Hindsight) | Reproducible from indexed data | ✅ Confirmed |

---

## 🛡️ Threat Modeling Assumptions

The lab assumed an **opportunistic external attacker** profile, characterized by:

- No prior knowledge of the target
- Automated tooling (e.g., Hydra, NLBrute, Metasploit)
- Internet-wide IPv4 scanning
- Common username dictionaries (`admin`, `administrator`, `test`, etc.)

These assumptions matched the observed attack patterns precisely — confirming the lab's value as a realistic detection sandbox.

---

## 📊 Success Metrics

| Metric | Target | Achieved |
|--------|--------|----------|
| Events ingested | > 10,000 | ✅ 28,963 |
| Unique attacker IPs detected | ≥ 3 | ✅ 4 |
| Multi-event correlation built | 1 | ✅ 1 (4625 + 4740) |
| MITRE techniques mapped | ≥ 3 | ✅ 5 |
| Out-of-band recovery validated | Yes | ✅ Yes |

---

> 📂 Return to [README.md](../README.md) | [Lessons Learned](lessons_learned.md)
# ☁️ Cloud Threat Detection Lab (AWS → Splunk)

**End-to-end SOC lab that ingests AWS logs into Splunk, simulates real cloud attacks, and detects them with MITRE ATT&CK–mapped SPL rules.**

This project builds the full detection loop a SOC analyst works in: 
- **log ingestion → attack simulation → detection engineering → investigation → response.**
- AWS telemetry (CloudTrail, VPC Flow Logs, GuardDuty, S3 access logs) flows into Splunk, attacks are generated with Stratus Red Team, and each is caught by a custom detection mapped to MITRE ATT&CK and surfaced on a SOC-style dashboard.

---

## 🧱 Architecture

```
   ATTACK SIDE                      LOG SIDE                      SPLUNK SIDE
   ----------                       --------                      -----------

  Stratus Red Team
  (simulated attacks)
        |
        v
  ┌──────────────┐   activity   ┌──────────────────┐
  │ crown-jewels │────────────► │ CloudTrail       │
  │ (target)     │              │ VPC Flow Logs    │
  │ fake secrets │              │ GuardDuty        │
  └──────────────┘              └──────────────────┘
                                        │ write logs
                                        v
                                ┌──────────────────┐
                                │  S3 buckets      │
                                └──────────────────┘
                                        │ ObjectCreated
                                        v
                                ┌──────────────────┐
                                │  SNS topic       │
                                └──────────────────┘
                                        │
                                        v
                                ┌──────────────────┐   pull    ┌──────────┐
                                │  SQS queue       │ ◄──────── │  Splunk  │
                                └──────────────────┘           │  Add-on  │
                                                                └──────────┘
                                                                     │
                                                                     v
                                                        Detections → Dashboard
                                                        → Incident reports
```

**Flow in one line:** 
- attack the target → AWS log services record it to S3 → S3 notifies SNS → SNS fans out to SQS → the Splunk Add-on for AWS pulls from SQS → detections fire → investigate and respond.

---

## 🛠️ Stack

AWS (CloudTrail, VPC Flow Logs, GuardDuty, S3, SNS, SQS, IAM) · Splunk Enterprise · Splunk Add-on for AWS · Stratus Red Team · MITRE ATT&CK · SPL

---

## 📂 Repository layout

| Path | What's in it |
|---|---|
| `detections/` | SPL detection rules, one file per technique |
| `incident-reports/` | Per-attack investigation write-ups |
| `iam-policies/` | Least-privilege IAM and resource policies used in the build |
| `docs/` | Setup guide, MITRE coverage table, architecture notes |

---

## 🔎 Detections

| Detection | MITRE technique | Log source |
|---|---|---|
| Backdoor IAM admin user created | T1136.003 – Create Account: Cloud Account | CloudTrail |
| CloudTrail logging stopped | T1562.001 – Impair Defenses: Disable Logging | CloudTrail |
| *(more as the lab grows)* | | |

See [`docs/mitre-coverage.md`](docs/mitre-coverage.md) for the full mapping.

---

## 🧪 How attacks are generated

Attacks are simulated with [Stratus Red Team](https://github.com/DataDog/stratus-red-team) against an **isolated lab AWS account**. Each attack follows the same loop:

```bash
stratus detonate <attack-id>   # perform the attack
# ... confirm detection fires in Splunk ...
stratus cleanup <attack-id>    # tear down what was created
```

---

## ⚠️ Safety & cost notes

- Runs in a **dedicated lab AWS account**, isolated from anything real.
- A **billing alarm / budget** is set to catch unexpected spend.
- All "sensitive" data in the target bucket is **fake** (placeholder values only).
- **No real credentials are committed** — see [`.gitignore`](.gitignore). Any access key shown in screenshots belongs to a short-lived Stratus resource and is destroyed at cleanup.

---

## 👤 Author

**Niharika Umrani** — Cybersecurity professional (SOC / detection engineering)
CompTIA Security+
[LinkedIn](https://linkedin.com/in/niharikaumrani) · [GitHub](https://github.com/niha-v)

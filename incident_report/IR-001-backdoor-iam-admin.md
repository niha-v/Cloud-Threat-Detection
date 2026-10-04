# Incident Report IR-001: Backdoor IAM Admin User

| Field | Value |
|---|---|
| **Incident ID** | IR-001 |
| **Date detected** | 2026-10-02 |
| **Severity** | High |
| **MITRE technique** | T1136.003 – Create Account: Cloud Account |
| **Status** | Resolved (lab) |

---
<img src = "https://github.com/niha-v/Cloud-Threat-Detection/blob/main/Image/02.png" width = 1000>




## Summary

A new IAM user (`malicious-iam-user`) was created and granted full
`AdministratorAccess`, followed immediately by the creation of a programmatic access
key. The three actions occurred within the same second from a single source, consistent
with an automated persistence technique establishing a backdoor identity.

## Timeline (UTC)

| Time | Event | Detail |
|---|---|---|
| 23:00:06 | DescribeAccountAttributes | Recon prior to action |
| 23:00:07 | CreateUser | `malicious-iam-user` |
| 23:00:07 | AttachUserPolicy | `arn:aws:iam::aws:policy/AdministratorAccess` |
| 23:00:07 | CreateAccessKey | key `AKIA****` (redacted) issued to the new user |

## Indicators

- **IAM user:** `malicious-iam-user`
- **Source IP:** `44.208.162.31`
- **User agent:** `stratus-red-team_*` (simulation marker)

## Detection

Caught in Splunk via CloudTrail ingestion:

```spl
index=main eventName=CreateUser OR eventName=AttachUserPolicy OR eventName=CreateAccessKey
| table _time, eventName, userName, sourceIPAddress, userAgent
| sort _time
```

## Impact

Lab environment, no production impact. In a real environment this would represent a
full-admin backdoor capable of persisting through credential rotation on the original
foothold.

## Response actions

1. Identified the three-event sequence from a single actor.
2. Deleted the malicious IAM user and revoked its access key.
3. Confirmed teardown via `stratus cleanup aws.persistence.iam-create-admin-user`.

## Lessons / follow-up

- Converted the search into a scheduled alert ("Backdoor IAM Admin User Created").
- Hardened the rule to key on `AdministratorAccess` being attached rather than on the
  tool's user agent, so it generalizes beyond the simulation.

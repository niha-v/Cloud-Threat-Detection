# Incident Report IR-002: CloudTrail Logging Stopped

| Field | Value |
|---|---|
| **Incident ID** | IR-002 |
| **Date detected** | 2026-10-08 |
| **Severity** | Critical |
| **MITRE technique** | T1562.008 – Impair Defenses: Disable or Modify Cloud Logs *(verify on attack.mitre.org)* |
| **Status** | Resolved (lab) |

---

## Summary

An actor stopped logging on an AWS CloudTrail trail with the `StopLogging` API call. Stopping a
trail is a classic defense-evasion move: it blinds the SOC so later actions go unrecorded. The
activity was generated on purpose with Stratus Red Team (`aws.defense-evasion.cloudtrail-stop`) in
an isolated lab account and detected in Splunk from CloudTrail logs shipped through S3, SNS and SQS.

## Timeline

Times are as displayed in Splunk (local time). Replace with the UTC `eventTime` before publishing.

| Time | Event | Detail |
|---|---|---|
| 2026-10-08 11:34:40 | `StopLogging` | Actor: account root. Source IP `54.158.139.159`. User agent `stratus-red-team_*` |
| 2026-10-08 11:43:02 | `StopLogging` | Second detonation, same actor and source IP |
| 2026-10-08 11:43:24 | `DeleteTrail` | Tear-down of the throwaway trail by Stratus cleanup (Terraform user agent) |

## Indicators

- **Event names:** `StopLogging`, `DeleteTrail`
- **Actor:** account root (lab only)
- **Source IP:** `54.158.139.159` (CloudShell)
- **User agent:** `stratus-red-team_*` (simulation marker); Terraform user agent on cleanup
- **Trail affected:** throwaway trail `stratus-red-team-ct-stop-<id>-trail` (confirm the exact name
  from `requestParameters.name`). The main lab trail kept logging, which is why the events were captured.

## Detection

See `detections/02-cloudtrail-logging-stopped.md`.

```spl
index=main eventName=StopLogging
| table _time, eventName, userIdentity.arn, requestParameters.name, sourceIPAddress, userAgent
| sort _time
```

The dashboard's "Logging tampering" tile turned red and the MITRE table listed
`T1562.008 Disable or Modify Cloud Logs` with a count of 2.

## Analysis

- `StopLogging` is recorded *before* logging stops, so it is a last-gasp signal. In a real incident,
  assume activity after it on that trail is unmonitored.
- The actor was root with MFA-authenticated console credentials. Real attackers rarely use root,
  so root use is itself worth alerting on.
- The user agent identifies the simulation. A real attacker would not use a tool-named user agent,
  so the durable signal is the behavior (a trail being stopped), not the tool name.

## Impact

None in the lab. In production, stopping CloudTrail removes audit visibility for that trail and could
hide follow-on actions such as data theft or privilege escalation.

## Response actions

1. Re-enable logging on the affected trail: `aws cloudtrail start-logging --name <trail>`.
2. Treat the gap between `StopLogging` and re-enable as a blind spot; pivot to GuardDuty and
   VPC Flow Logs to reconstruct activity.
3. Identify and contain the actor: review the session, rotate or revoke credentials.
4. Clean up the simulation: `stratus cleanup aws.defense-evasion.cloudtrail-stop`.

## Lessons / follow-up

- **Tuning:** the `DeleteTrail` from Stratus cleanup appeared as a high-risk event (a lab-caused false
  positive). Exclude the throwaway trail with `NOT requestParameters.name="stratus-red-team-*"`
  rather than excluding Terraform globally. See `docs/tuning-notes.md`.
- **Broaden the rule** to include `DeleteTrail`, `UpdateTrail` and `PutEventSelectors`.
- **Alert:** every 5 minutes, trigger on results > 0, severity Critical.
- **Hardening:** enable CloudTrail log file validation, deliver logs to a bucket the attacker's
  identity cannot modify, and alert on any root-account activity.

## Evidence

- `docs/images/` – Splunk search showing the `StopLogging` events
- `docs/images/` – dashboard with the red "Logging tampering" tile

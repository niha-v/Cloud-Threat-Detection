# Incident Report IR-005: S3 Bucket Policy Backdoor

| Field | Value |
|---|---|
| **Incident ID** | IR-005 |
| **Date detected** | 2026-10-11 (TODO: confirm) |
| **Severity** | High |
| **MITRE technique** | T1537 – Transfer Data to Cloud Account *(verify)* |
| **Status** | Resolved (lab) |

## Summary

A bucket policy was changed to grant an outside principal access to the bucket, letting an attacker read data later without the original credentials. The activity was simulated with Stratus Red Team (`aws.exfiltration.s3-backdoor-bucket-policy`) in an isolated lab account and detected in Splunk from CloudTrail logs.

## Timeline (UTC)

| Time | Event | Detail |
|---|---|---|
| TODO | `PutBucketPolicy` | TODO: copy `_time`, actor ARN and source IP from the Splunk result |

## Indicators

- **Event:** `PutBucketPolicy`
- **Actor:** TODO
- **Source IP:** TODO
- **User agent:** `stratus-red-team_*` (simulation marker)

## Detection

See [detection 05](../detections/05-s3-bucket-policy-backdoor.md).

## Impact

Lab only. In a real account, the bucket's data would be readable from outside the organization.

## Response actions

1. Review the new policy and remove outside-access statements.
2. Check data events for reads by the new principal.
3. Investigate the actor that changed the policy.
4. `stratus cleanup aws.exfiltration.s3-backdoor-bucket-policy`

## Lessons

- Policy changes are rare and high-signal; allow-list known admin roles instead of silencing the event.
- Preventive fix: S3 Block Public Access and an SCP restricting bucket policy changes.

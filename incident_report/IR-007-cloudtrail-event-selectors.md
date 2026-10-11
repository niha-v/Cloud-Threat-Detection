# Incident Report IR-007: CloudTrail Event Selectors Changed

| Field | Value |
|---|---|
| **Incident ID** | IR-007 |
| **Date detected** | 2026-10-11 (TODO: confirm) |
| **Severity** | High |
| **MITRE technique** | T1562.008 – Impair Defenses: Disable or Modify Cloud Logs *(verify)* |
| **Status** | Resolved (lab) |

## Summary

A trail's event selectors were changed, which can stop CloudTrail from recording activity (including S3 data events) without calling StopLogging. The activity was simulated with Stratus Red Team (`aws.defense-evasion.cloudtrail-event-selectors`) in an isolated lab account and detected in Splunk from CloudTrail logs.

## Timeline (UTC)

| Time | Event | Detail |
|---|---|---|
| TODO | `PutEventSelectors` | TODO: copy `_time`, actor ARN and source IP from the Splunk result |

## Indicators

- **Event:** `PutEventSelectors`
- **Actor:** TODO
- **Source IP:** TODO
- **User agent:** `stratus-red-team_*` (simulation marker)

## Detection

See [detection 07](../detections/07-cloudtrail-event-selectors.md).

## Impact

Lab only. In a real account, later actions could go unrecorded.

## Response actions

1. Restore the original event selectors.
2. Treat the period since the change as partially blind.
3. Investigate the actor.
4. `stratus cleanup aws.defense-evasion.cloudtrail-event-selectors`

## Lessons

- A second way to blind a trail; pair with detection 02 for full coverage.
- Preventive fix: restrict `cloudtrail:PutEventSelectors` and monitor trail configuration with AWS Config.

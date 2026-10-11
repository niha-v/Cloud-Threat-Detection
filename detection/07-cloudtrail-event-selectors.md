# Detection 07: CloudTrail Event Selectors Changed

**MITRE ATT&CK:** T1562.008 – Impair Defenses: Disable or Modify Cloud Logs *(verify on attack.mitre.org)*
**Log source:** AWS CloudTrail (management events)
**Severity:** High
**Report:** [IR-007](../incident-reports/IR-007-cloudtrail-event-selectors.md)

## What it detects

Changes to a trail's event selectors, which control what the trail records. An attacker can narrow them so activity (including S3 data events like the exfiltration in detection 03) is no longer logged, without ever calling `StopLogging`.

## Attack simulated

```bash
stratus detonate aws.defense-evasion.cloudtrail-event-selectors
```

## Detection

```spl
index=main eventName=PutEventSelectors
| table _time, userIdentity.arn, requestParameters.trailName, requestParameters.eventSelectors, sourceIPAddress, userAgent
| sort _time
```

## Tuning

Selector changes are rare. Allow-list the admin role or pipeline that manages trails.

## Alert

Every 5 minutes, trigger when results > 0, severity High.

## Response

1. Restore the original event selectors.
2. Treat the window since the change as partially blind.
3. Investigate the actor.
4. `stratus cleanup aws.defense-evasion.cloudtrail-event-selectors`

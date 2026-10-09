# Detection: CloudTrail Logging Stopped

**MITRE ATT&CK:** T1562.008 – Impair Defenses: Disable or Modify Cloud Logs *(verify on attack.mitre.org)*
**Log source:** AWS CloudTrail (management events)
**Severity:** Critical
**Status:** Built and validated (see [IR-002](../incident-reports/IR-002-cloudtrail-logging-stopped.md))

---

## What it detects

An attacker disabling CloudTrail to blind the SOC before carrying out further actions. The
`StopLogging` event is itself logged just before logging stops, so it is a high-value "last gasp"
signal. If you see it, assume activity after it on that trail is unmonitored.

## Attack simulated

```bash
stratus detonate aws.defense-evasion.cloudtrail-stop
```

Stratus creates its own throwaway trail and stops that one, so the main lab trail keeps logging and
records the call.

Observed CloudTrail events:
- `StopLogging` on the throwaway trail (twice, from two detonations)
- `DeleteTrail` during `stratus cleanup` (tear-down; see tuning below)

---

## Detection

```spl
index=main eventName=StopLogging
| table _time, eventName, userIdentity.arn, requestParameters.name, sourceIPAddress, userAgent
| sort _time
```

Broader version covering other ways to blind a trail:

```spl
index=main (eventName=StopLogging OR eventName=DeleteTrail OR eventName=UpdateTrail OR eventName=PutEventSelectors)
    NOT requestParameters.name="stratus-red-team-*"
| table _time, eventName, userIdentity.arn, requestParameters.name, sourceIPAddress, userAgent
| sort _time
```

## Tuning

`stratus cleanup` deletes the throwaway trail with a Terraform user agent, which the broad rule
flags as `DeleteTrail`. The `NOT requestParameters.name="stratus-red-team-*"` clause removes that
lab-only noise. Keep the exclusion scoped to the trail name; never exclude Terraform globally, since
an attacker could use it too.

Note: the exclusion also hides `StopLogging` on the Stratus trail, so use the simple search above
(no exclusion) when you run the attack and need to see it fire.

---

## Alert configuration

- Schedule: every 5 minutes (or real-time)
- Trigger: number of results > 0
- Severity: Critical

## Response

1. Re-enable logging on the trail immediately (`aws cloudtrail start-logging --name <trail>`).
2. Treat the window after `StopLogging` as a blind spot; pivot to GuardDuty and VPC Flow Logs.
3. Investigate the actor that issued `StopLogging`.
4. `stratus cleanup aws.defense-evasion.cloudtrail-stop`

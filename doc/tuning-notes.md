# Detection Tuning Notes

## Case 1: Cleanup flagged as an attack (`DeleteTrail`)

**Observation:** The dashboard's "Recent high-risk events" table showed a `DeleteTrail` event
at 11:43 local time. The user agent was Terraform (`HashiCorp-terraform-exec`), which is what
Stratus Red Team uses internally to tear down its throwaway trail during `cleanup`.

**Problem:** The rule treated lab cleanup as an attacker deleting a trail (false positive).

**Tuning options:**
- Exclude events whose `requestParameters.name` starts with `stratus-red-team-`.
- Or exclude the Terraform user agent for that trail name only (never exclude Terraform globally,
  since an attacker could use it too).

```spl
index=main eventName=DeleteTrail NOT requestParameters.name="stratus-red-team-*"
```

**Takeaway:** Scope exclusions as narrowly as possible. A broad exclusion creates a blind spot.

## Case 2: Everything runs as root

All simulated activity appears as `arn:aws:iam::<account>:root` because the lab was driven from a
root-authenticated CloudShell. Real attackers rarely use root, and root use is itself suspicious:

```spl
index=main sourcetype=aws:cloudtrail userIdentity.type=Root
| stats count by eventName, sourceIPAddress
```

**Follow-up:** Create a dedicated admin IAM user for daily use and alert on any root activity.

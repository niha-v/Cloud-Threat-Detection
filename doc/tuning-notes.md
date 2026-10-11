# Detection Tuning Notes

## Case 1: Cleanup flagged as an attack (`DeleteTrail`)

**Observation:** The "Recent high-risk events" table showed a `DeleteTrail` event with a Terraform user agent. That is what Stratus Red Team uses to tear down its throwaway trail during `cleanup`.

**Problem:** The rule treated lab cleanup as an attacker deleting a trail (a false positive).

**Fix:** exclude the throwaway trail by name:

```spl
index=main eventName=DeleteTrail NOT requestParameters.name="stratus-red-team-*"
```

Keep exclusions as narrow as possible. Excluding Terraform globally would create a blind spot an attacker could use.

## Case 2: Everything runs as root

All simulated activity appears as the account root user because the lab was driven from a root-authenticated CloudShell. Real attackers rarely use root, and root use is itself suspicious:

```spl
index=main sourcetype=aws:cloudtrail userIdentity.type=Root
| stats count by eventName, sourceIPAddress
```

**Follow-up:** use a dedicated admin IAM user for daily work and alert on any root activity.

## Case 3: Logs about logs

S3 data events enabled on every bucket made Splunk's own reads of the log bucket generate new events. Scoping data events to the target bucket removed the loop and protected the daily ingest quota.

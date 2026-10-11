# Detection 05: S3 Bucket Policy Backdoor

**MITRE ATT&CK:** T1537 – Transfer Data to Cloud Account *(verify on attack.mitre.org)*
**Log source:** AWS CloudTrail (management events)
**Severity:** High
**Report:** [IR-005](../incident-reports/IR-005-s3-bucket-policy-backdoor.md)

## What it detects

A bucket policy changed to grant an outside principal access. Attackers use this to read data later without needing the original credentials.

## Attack simulated

```bash
stratus detonate aws.exfiltration.s3-backdoor-bucket-policy
```

## Detection

```spl
index=main eventName=PutBucketPolicy
| table _time, userIdentity.arn, requestParameters.bucketName, sourceIPAddress, userAgent
| sort _time
```

Then open the event and inspect the policy document for `Principal` values that are `*` or an account other than your own.

## Tuning

Expected policy changes should come from known automation or admin roles. Allow-list those ARNs rather than silencing the event.

## Alert

Every 5 minutes, trigger when results > 0, severity High.

## Response

1. Review the new policy; remove statements granting outside access.
2. Check access logs and data events for reads by the new principal.
3. Investigate the actor that changed the policy.
4. `stratus cleanup aws.exfiltration.s3-backdoor-bucket-policy`

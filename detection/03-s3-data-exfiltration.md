# Detection: S3 Bulk Object Download (Data Exfiltration)

**MITRE ATT&CK:** T1530 – Data from Cloud Storage *(verify on attack.mitre.org)*
**Log source:** AWS CloudTrail S3 data events (scoped to the target bucket only)
**Severity:** High

---

## What it detects

One identity reading several objects from a sensitive bucket in a short window. In the lab, the
`crown-jewels-67` bucket holds fake "sensitive" files; reading them in bulk simulates an attacker
copying data out of cloud storage.

## Attack simulated

```bash
aws s3 cp s3://crown-jewels-67/ ./stolen/ --recursive
```

Observed CloudTrail event: `GetObject` (`eventCategory: Data`) from `aws-cli` running in
CloudShell (`command#s3.cp` in the user agent), 2026-10-08 17:59:08 UTC.

## Detection

```spl
index=main sourcetype=aws:cloudtrail eventName=GetObject requestParameters.bucketName="crown-jewels-67"
| stats count, dc(requestParameters.key) as files, values(requestParameters.key) as keys
        by userIdentity.arn, sourceIPAddress
| where files>=3
```

## Alert configuration

- Schedule: every 5 minutes, searching the last 15 minutes
- Trigger: number of results > 0
- Severity: High

## Why data events matter

Management events (CreateUser, StopLogging, etc.) are logged by default. Reading an object is a
*data event* and is only recorded if enabled. Without it, bulk downloads leave no trace in CloudTrail.
Data events are scoped to the target bucket only, to avoid logging the log buckets themselves
(which creates a feedback loop and burns ingest quota).

## Response

1. Identify the actor and source IP; check whether the activity was expected.
2. Review what was read (`keys` column) and assess sensitivity.
3. Revoke or rotate the credentials used.
4. Review bucket policy and access logs for other readers.

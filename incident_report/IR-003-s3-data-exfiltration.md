# Incident Report IR-003: S3 Data Exfiltration (Simulated)

| Field | Value |
|---|---|
| **Incident ID** | IR-003 |
| **Date detected** | 2026-10-08 |
| **Severity** | High |
| **MITRE technique** | T1530 – Data from Cloud Storage *(verify)* |
| **Status** | Resolved (lab) |

## Summary

Objects in the `crown-jewels-67` bucket (fake sensitive data) were bulk-downloaded using the AWS CLI
from a CloudShell session. The reads were captured as CloudTrail S3 data events and ingested into Splunk.

## Timeline (UTC)

| Time | Event | Detail |
|---|---|---|
| 17:59:08 | GetObject | `crown-jewels-67`, via `aws s3 cp` (CloudShell), source IP `54.198.243.215` |
| ~18:00 | CloudTrail log file written | `...20261008T1800Z...json.gz` to the log bucket |
| ~18:05+ | Indexed in Splunk | `sourcetype=aws:cloudtrail` |

*Confirm the number of files read from the `files` column of the detection search and add it here.*

## Indicators

- **Bucket:** `crown-jewels-67`
- **Source IP:** `54.198.243.215` (CloudShell)
- **User agent:** `aws-cli/2.x ... exec-env/CloudShell ... command#s3.cp`
- **Actor:** account root (lab only; see lessons)

## Detection

See `detections/03-s3-data-exfiltration.md`.

## Impact

Lab only, with fake data. In a real environment this would be loss of confidential data.

## Lessons / follow-up

- S3 data events had to be enabled for the target bucket before any read was visible.
- Activity ran as **root**, which is itself a high-signal finding. A follow-up detection should alert on
  `userIdentity.type=Root` usage, and daily work should move to a separate admin IAM user.
- A burst threshold (`files>=3`) reduces noise from single reads.

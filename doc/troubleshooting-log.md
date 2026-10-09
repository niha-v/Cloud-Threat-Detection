# Troubleshooting Log (replaces the "gotchas" section of setup-guide.md)

Real problems hit while building the lab, and how each was fixed.

| # | Symptom | Root cause | Fix |
|---|---|---|---|
| 1 | Splunk warning: "does not have a valid SNS Signature" | Buckets notified SQS directly; the Add-on expects S3 → SNS → SQS | Inserted an SNS topic between S3 and SQS |
| 2 | "Unknown Error" saving S3 event notification | SNS topic was created as **FIFO**; S3 only publishes to Standard topics | Recreated the topic as Standard |
| 3 | Every CloudTrail file indexed as one giant event; `eventName` searches empty | Input used the S3 Access Logs decoder (sourcetype locked at creation) | Created a **CloudTrail** input (`aws:cloudtrail`) and disabled the old one |
| 4 | `AccessDenied` on `sqs:ListQueues` | List actions can't be resource-scoped; policy pinned it to one queue ARN | Moved `ListQueues` to its own statement with `Resource: "*"` |
| 5 | `AccessDenied` on `s3:GetBucketLocation` | Add-on checks bucket region before reading | Added `s3:GetBucketLocation` to the read policy |
| 6 | Attack events not in Splunk right away | CloudTrail delivers log files in batches (10–20 min) | Waited; searched by bare keyword to confirm arrival |
| 7 | Noisy ingest, logs-about-logs | S3 data events enabled for all buckets | Scoped data events to the target bucket only |
| 8 | `spath ... as` syntax error | Wrong syntax | `spath output=<name> path=<field>` |

**Lessons:** SQS delivers each message to one reader (don't poll the queue in the console while Splunk
is pulling); least-privilege policies need wildcard resources for some list actions; always validate
parsing (`table _time, eventName, ...`) before building detections on top.

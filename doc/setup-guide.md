# Setup Guide

How the lab was built, in the order it was done.

## 1. Safety first
- Dedicated lab AWS account (isolated from anything real).
- Billing budget + alarm set with email alerts.

## 2. AWS log sources
- **CloudTrail** – management events (Read + Write) + S3 data events on the target bucket.
- **VPC Flow Logs** – published to S3.
- **GuardDuty** – enabled.

## 3. Buckets
- `threat-detection-lab` – central log destination.
- `aws-cloudtrail-logs-<account>-<id>` – auto-created by CloudTrail.
- `crown-jewels-<id>` – **target** bucket holding fake "sensitive" files to be attacked.

## 4. Pipeline: S3 → SNS → SQS → Splunk
The Splunk Add-on for AWS (SQS-Based S3 input) expects SNS-wrapped notifications, so:

1. Create a **Standard** SNS topic `s3-log-notifications` (not FIFO — S3 can't publish to FIFO).
2. Subscribe the **Standard** SQS queue `splunk-aws-notifications` to the topic.
3. SQS access policy: allow both `s3.amazonaws.com` and `sns.amazonaws.com` to `SendMessage`.
4. SNS topic access policy: allow `s3.amazonaws.com` to `Publish`.
5. On each log bucket: Event notification (`ObjectCreated:*`) → destination **SNS topic**.

See `iam-policies/` for the exact JSON.

## 5. Splunk ingestion
- Install **Splunk Add-on for AWS**.
- Add the `splunk-reader` IAM credential (read-only to the log buckets + queue).
- Create an **SQS-Based S3** input pointed at `splunk-aws-notifications` in `us-east-1`.
- Confirm data: `index=main | stats count by sourcetype`.

> **Gotcha encountered:** the input's sourcetype auto-set to `aws:s3:accesslogs` and was
> locked. The CloudTrail JSON still auto-extracts (`eventName`, `sourceIPAddress`, etc.),
> so detections work regardless of the label.

## 6. Attack → detect loop
```bash
stratus detonate <attack-id>
# confirm detection in Splunk
stratus cleanup <attack-id>
```

## Troubleshooting log (things that actually went wrong)
- **"Invalid SNS Signature" warning in Splunk** → buckets were pointed straight at SQS;
  the Add-on expects S3 → SNS → SQS. Fixed by inserting the SNS topic.
- **"Unknown Error" saving S3 event notification** → SNS topic had been created as FIFO.
  S3 can only publish to a Standard topic. Recreated as Standard.
- **Attack events not appearing** → earlier CloudTrail file simply hadn't been pulled yet;
  management events deliver in waves (can take 10–20 min).

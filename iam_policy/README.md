# IAM & Resource Policies

All account IDs and bucket names below are **placeholders** — replace with your own.
Never commit real access keys.

## `splunk-reader` IAM policy (read-only, least privilege)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadLogBuckets",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::<LOG_BUCKET>",
        "arn:aws:s3:::<LOG_BUCKET>/*",
        "arn:aws:s3:::<CLOUDTRAIL_BUCKET>",
        "arn:aws:s3:::<CLOUDTRAIL_BUCKET>/*"
      ]
    },
    {
      "Sid": "ReadSQS",
      "Effect": "Allow",
      "Action": [
        "sqs:GetQueueAttributes", "sqs:GetQueueUrl", "sqs:ListQueues",
        "sqs:ReceiveMessage", "sqs:DeleteMessage"
      ],
      "Resource": "arn:aws:sqs:us-east-1:<ACCOUNT_ID>:splunk-aws-notifications"
    }
  ]
}
```

## SQS queue access policy (allow S3 + SNS to send)

```json
{
  "Version": "2012-10-17",
  "Id": "AllowS3AndSNS",
  "Statement": [
    {
      "Sid": "AllowS3Notifications",
      "Effect": "Allow",
      "Principal": { "Service": "s3.amazonaws.com" },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:us-east-1:<ACCOUNT_ID>:splunk-aws-notifications",
      "Condition": { "StringEquals": { "aws:SourceAccount": "<ACCOUNT_ID>" } }
    },
    {
      "Sid": "AllowSNSPublish",
      "Effect": "Allow",
      "Principal": { "Service": "sns.amazonaws.com" },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:us-east-1:<ACCOUNT_ID>:splunk-aws-notifications",
      "Condition": {
        "ArnEquals": { "aws:SourceArn": "arn:aws:sns:us-east-1:<ACCOUNT_ID>:s3-log-notifications" }
      }
    }
  ]
}
```

## SNS topic access policy (allow S3 to publish)

```json
{
  "Version": "2012-10-17",
  "Id": "AllowS3Publish",
  "Statement": [
    {
      "Sid": "AllowS3ToPublish",
      "Effect": "Allow",
      "Principal": { "Service": "s3.amazonaws.com" },
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:us-east-1:<ACCOUNT_ID>:s3-log-notifications",
      "Condition": { "StringEquals": { "aws:SourceAccount": "<ACCOUNT_ID>" } }
    }
  ]
}
```

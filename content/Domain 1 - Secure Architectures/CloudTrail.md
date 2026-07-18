# AWS CloudTrail

## Overview
Records API calls and events made in your AWS account. Provides governance, compliance, and audit for your account.

## Event Types

### Management Events
- Operations on resources (create, modify, delete)
- Read events: DescribeInstances, ListBuckets
- Write events: CreateBucket, TerminateInstances
- Enabled by default

### Data Events
- Object-level operations (S3 GetObject, PutObject, Lambda Invoke)
- High volume — not enabled by default
- Additional cost

### Insight Events
- Detect unusual activity (burst of API calls, error rates)
- Analyzes management events for anomalies
- Creates baseline, then alerts on deviations

## Trail Configuration
- **Single-region trail** — logs events in one region
- **Multi-region trail** — logs events from all regions (recommended)
- **Organization trail** — all accounts in AWS Organizations

## Log Delivery
- Delivered to S3 bucket (with SSE-S3 or SSE-KMS encryption)
- Optional: CloudWatch Logs for real-time monitoring
- Optional: SNS notifications for delivery
- Log file integrity validation available

## Exam Tips
- CloudTrail is enabled by default (90-day event history)
- For long-term storage, create a trail → S3
- Use with CloudWatch Logs for real-time alerting
- Data events are high-volume and cost extra
- CloudTrail Insights for detecting unusual API activity
- Not real-time (typically 15 minutes delay)

---

Related: [[AWS Config]] | [[GuardDuty]] | [[Cross-Cutting/CloudWatch]]
#aws #cloudtrail #audit #security #domain1

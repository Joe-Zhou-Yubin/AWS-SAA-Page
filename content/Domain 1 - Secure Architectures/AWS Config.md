# AWS Config

## Overview
Records and evaluates resource configurations over time. Helps with compliance auditing, security analysis, and change tracking.

## Key Features
- Records configuration changes for AWS resources
- Evaluates against rules (compliance check)
- Configuration timeline for any resource
- No prevention — only detection and remediation
- Regional service (can aggregate across regions/accounts)

## Config Rules
- **AWS Managed Rules** — pre-built (75+ rules)
- **Custom Rules** — Lambda functions you define
- Evaluated on configuration change or periodic schedule

### Example Rules
- `restricted-ssh` — no security groups allowing 0.0.0.0/0 on port 22
- `s3-bucket-public-read-prohibited` — no public S3 buckets
- `ec2-instance-no-public-ip` — no public IPs on EC2
- `rds-storage-encrypted` — RDS encryption enabled

## Remediation
- Automatic remediation via SSM Automation Documents
- Manual remediation with notifications (SNS)
- EventBridge integration for custom workflows

## Config vs CloudTrail

| Feature | AWS Config | CloudTrail |
|---------|-----------|------------|
| Focus | Resource configuration state | API activity (who did what) |
| Question answered | "Is this resource compliant?" | "Who changed this resource?" |
| Evaluation | Rules-based compliance | Audit trail |

## Exam Tips
- Config = resource compliance and configuration history
- Does NOT prevent changes (use SCPs or IAM for prevention)
- Requires an S3 bucket for storing configuration snapshots
- Use Config + CloudTrail together for full audit picture
- Aggregator for multi-account/multi-region view

---

Related: [[CloudTrail]] | [[Security Hub]] | [[Cross-Cutting/Systems Manager]]
#aws #config #compliance #security #domain1

# Amazon GuardDuty

## Overview
Intelligent threat detection service using ML, anomaly detection, and threat intelligence to identify threats in your AWS environment.

## Data Sources
- VPC Flow Logs
- DNS Logs
- CloudTrail Events (management & data events)
- S3 Data Events
- EKS Audit Logs
- RDS Login Activity
- Lambda Network Activity

## Finding Types
- **Reconnaissance** — unusual API calls, port scanning
- **Instance compromise** — cryptocurrency mining, C&C communication
- **Account compromise** — unusual API calls from anomalous locations
- **Bucket compromise** — suspicious S3 access patterns

## Key Features
- One-click enable, no agents required
- 30-day free trial
- Can be managed across accounts (via Organizations)
- Findings → EventBridge → SNS/Lambda for automated response
- Supports trusted/threat IP lists

## Automated Response Pattern
```
GuardDuty Finding → EventBridge Rule → Lambda → (block IP, isolate instance, etc.)
```

## Exam Tips
- GuardDuty = threat detection (not prevention)
- No agents, no infrastructure to manage
- Input: flow logs, DNS, CloudTrail (you don't need to enable these separately)
- Output: findings to EventBridge for automation
- Can detect cryptocurrency mining attacks

---

Related: [[CloudTrail]] | [[Security Hub]] | [[Inspector]]
#aws #guardduty #threat-detection #security #domain1

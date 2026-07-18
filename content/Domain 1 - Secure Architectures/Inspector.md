# Amazon Inspector

## Overview
Automated vulnerability management service that scans workloads for software vulnerabilities and unintended network exposure.

## What It Scans
- **EC2 instances** — CVEs, network reachability (requires SSM Agent)
- **Container images in ECR** — CVEs on push
- **Lambda functions** — code and dependency vulnerabilities

## Key Features
- Continuous scanning (not just one-time)
- Integrates with Systems Manager (SSM) for EC2 scanning
- Risk score for each finding
- Findings sent to Security Hub and EventBridge
- Supports AWS Organizations for multi-account

## Inspector vs GuardDuty vs Macie

| Service | Purpose |
|---------|---------|
| Inspector | Vulnerability scanning (CVEs, network exposure) |
| GuardDuty | Threat detection (malicious activity) |
| Macie | Sensitive data discovery (PII in S3) |

## Exam Tips
- Inspector = vulnerability assessment
- Scans EC2, ECR images, and Lambda
- Requires SSM Agent on EC2
- Continuous and automatic (not manual trigger)
- Reports to Security Hub

---

Related: [[GuardDuty]] | [[Macie]] | [[Security Hub]]
#aws #inspector #vulnerability #security #domain1

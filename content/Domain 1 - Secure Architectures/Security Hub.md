# AWS Security Hub

## Overview
Central security dashboard that aggregates findings from multiple AWS security services and third-party tools.

## Aggregates From
- GuardDuty
- Inspector
- Macie
- IAM Access Analyzer
- AWS Firewall Manager
- Systems Manager Patch Manager
- Third-party partner tools

## Security Standards
- AWS Foundational Security Best Practices
- CIS AWS Foundations Benchmark
- PCI DSS
- Runs automated compliance checks against these standards

## Key Features
- Cross-account aggregation (via Organizations)
- Cross-region aggregation
- Automated remediation via EventBridge + Lambda/SSM
- Findings in AWS Security Finding Format (ASFF)
- Must enable AWS Config first

## Exam Tips
- Security Hub = central aggregation of security findings
- Requires AWS Config to be enabled
- Provides compliance scoring against standards
- Use for unified security view across accounts/regions

---

Related: [[GuardDuty]] | [[Inspector]] | [[Macie]] | [[AWS Config]]
#aws #security-hub #security #domain1

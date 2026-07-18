# AWS Organizations

## Overview
AWS Organizations allows you to centrally manage multiple AWS accounts with consolidated billing and policy-based controls.

## Key Features

### Organizational Units (OUs)
- Hierarchical grouping of accounts
- Policies can be applied at OU level
- Common structure: Root → Management Account → OUs (Dev, Staging, Prod, Security)

### Service Control Policies (SCPs)
- Define maximum permissions for accounts
- Do NOT grant permissions — they restrict what identity-based policies can grant
- Applied to OUs or individual accounts
- Do NOT affect the management account
- Must have an explicit allow (deny by default if using allow-list strategy)

### Consolidated Billing
- Single payment method for all accounts
- Volume pricing discounts across all accounts
- Reserved Instance sharing across accounts

## Key Patterns
- **Multi-account strategy**: separate accounts for different environments, teams, or projects
- **Tag policies**: enforce consistent tagging across accounts
- **Backup policies**: centralized backup configuration
- **AI services opt-out policies**: control data usage by AI services

## Integration with Other Services
- **AWS Control Tower** — automated multi-account setup with guardrails
- **IAM Identity Center** — centralized user access across accounts
- **AWS RAM** — share resources across accounts
- **CloudTrail** — organization-wide trail

## Exam Tips
- SCPs do NOT affect the management account
- SCPs do NOT grant permissions, only restrict
- Use for limiting regions, services, or actions organization-wide
- Consolidated billing provides cost benefits

---

Related: [[IAM]] | [[Cross-Cutting/CloudFormation]]
#aws #organizations #security #domain1

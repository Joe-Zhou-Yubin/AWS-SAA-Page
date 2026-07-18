# Secure Multi-Account AWS Organization

AWS SAA-C03 Reference Architecture — 10

---

## Architecture Overview

An AWS Organization with multiple accounts grouped into Organisational Units (OUs). SCPs enforce guardrails at the org level, security tooling runs from a dedicated Audit account, networking is centralised in a Network account, and workloads are isolated in separate accounts by environment.

---

## Organisational Structure

```
Root (Management Account)
├── Security OU
│   ├── Log Archive Account   ← all CloudTrail, Config, Flow Logs go here
│   └── Audit Account         ← GuardDuty, Security Hub, Inspector, Firewall Manager
├── Infrastructure OU
│   ├── Network Account       ← Transit Gateway, Shared VPCs, Direct Connect
│   └── Shared Services       ← ECR, SSM, Secrets Manager, AWS Backup vault
├── Workloads OU
│   ├── Production Account    ← live workloads, strictest SCPs
│   └── Staging Account       ← pre-prod, mirrors prod config
└── Sandbox OU
    ├── Dev Account (Team A)  ← development, less restrictive
    └── Sandbox Account       ← PoC, experiments, budget alerts + auto-cleanup
```

---

## Service Control Policies (SCPs) — Applied at Root

SCPs enforced at root level apply to ALL member accounts:

| SCP | What it prevents |
|-----|-----------------|
| Deny `cloudtrail:StopLogging` | No account can disable CloudTrail |
| Deny `guardduty:DisassociateFromMasterAccount` | No account can leave GuardDuty org |
| Deny `organizations:LeaveOrganization` | No account can leave the Org |
| Deny regions not in approved list | Prevent resource creation in unwanted regions |
| Deny `s3:DeleteBucketPolicy` on log archive | Protect central log store |

**Critical rule:** SCPs never grant permissions — they only restrict. IAM policies in each account must still grant the actual permissions. SCP Deny overrides ANY IAM Allow, including root credentials.

---

## Account Purposes

### Log Archive Account
- Receives all log data via organisation-level services
- CloudTrail Org Trail → S3 (all API calls from all accounts, all regions)
- AWS Config Org Aggregator → S3 (all resource configurations)
- VPC Flow Logs → S3 or CloudWatch Logs
- Bucket policy prevents any account from deleting logs (SCP + bucket policy)
- Only the security team has read access

### Audit Account (Delegated Admin)
- GuardDuty delegated administrator — sees all findings across all member accounts
- Security Hub delegated administrator — aggregates GuardDuty, Inspector, Macie, Config findings
- Amazon Inspector delegated administrator — CVE scan results from all accounts
- AWS Firewall Manager — centrally deploys WAF rules, Shield Advanced to all accounts
- Amazon Macie — S3 sensitive data findings across accounts

**Why not run security tooling from the Management Account?**
The Management Account should have minimal resources and workloads. It controls billing and Org policies — if compromised, everything is at risk. Security tooling runs from the Audit Account via delegated admin to limit Management Account exposure.

### Network Account
- Transit Gateway (created here, shared to all workload accounts via AWS RAM)
- Shared VPC subnets (shared to workload accounts via VPC Sharing + RAM)
- AWS Direct Connect termination point
- Site-to-Site VPN connections
- Route 53 private hosted zones (centralised DNS)
- Route 53 Resolver rules shared to all accounts

### Shared Services Account
- ECR (shared container image registry — workload accounts pull images)
- SSM Parameter Store (shared config parameters)
- Secrets Manager (centralised secrets for multi-account access)
- AWS Backup central vault (receives backups from all accounts)

### Workload Accounts (Production / Staging)
- Each account is a blast radius boundary
- Production has the strictest SCPs
- No long-term IAM users — all access via IAM Identity Center permission sets
- Resources tagged for cost allocation
- AWS Config rules detect compliance drift

---

## Security Controls (Org-Wide)

| Service | Where managed | What it does |
|---------|--------------|-------------|
| CloudTrail Org Trail | Management Account → Log Archive | API audit log for all accounts |
| AWS Config Aggregator | Audit Account | Resource compliance across all accounts |
| GuardDuty | Audit Account (delegated admin) | Threat detection all accounts |
| Security Hub | Audit Account (delegated admin) | Aggregates all security findings |
| Amazon Inspector | Audit Account (delegated admin) | CVE scanning all accounts |
| AWS Firewall Manager | Audit Account | Central WAF + Shield policy deployment |
| IAM Identity Center | Management Account | SSO + permission sets for all accounts |
| AWS Backup | Shared Services Account | Central backup policy for all accounts |
| Transit Gateway | Network Account (shared via RAM) | Hub-spoke networking all VPCs |
| AWS Budgets | Per account + consolidated | Cost alerts per account and org-wide |
| Cost Allocation Tags | Activated in Management Account | Tag-based spend attribution |

---

## IAM Identity Center — Workforce Access

Users never get long-term IAM user credentials in workload accounts:

```
User (Okta/Azure AD) → SAML → IAM Identity Center
                                    ↓
                          Permission Set applied
                                    ↓
                     Short-lived session credentials
                          to target account
```

Permission Sets are reusable IAM policy bundles (e.g. "ReadOnly", "DeveloperAccess", "DataEngineer"). Assign a user to an account + permission set → they can assume that role via the SSO portal or CLI.

**IAM Identity Center vs IAM roles directly:**
- Identity Center: workforce access to AWS accounts and business apps. Centralised.
- IAM roles: machine-to-machine access (EC2 to S3, Lambda to DynamoDB). Per-account.

---

## Key Design Decisions

### Why separate accounts vs separate VPCs?
- VPC isolation: network-level only. IAM can still allow cross-VPC access.
- Account isolation: full blast radius separation. A compromised IAM credential in Account A cannot touch Account B's resources.
- Regulatory compliance (PCI-DSS, HIPAA) often mandates account-level separation.
- Service quotas: each account has independent limits — dev can't exhaust prod's EC2 quota.

### Why RAM for Transit Gateway sharing?
Creating a TGW in every account is expensive and complex to manage. One TGW in the Network Account, shared via AWS Resource Access Manager to all workload accounts. VPCs in workload accounts attach to the shared TGW directly — no cross-account TGW peering needed.

### Why AWS Backup centralised vault?
Each workload account creates backups, but the vault lives in the Shared Services account. This means even if a workload account is compromised (ransomware), the backups in the centralised vault are protected — the attacker can't delete them because the vault is in a different account with its own access controls.

---

## Exam Traps

| Trap | Reality |
|------|---------|
| "SCPs grant permissions" | SCPs only RESTRICT — IAM policies must still grant permissions |
| "SCP at OU level applies to Management Account" | SCPs do NOT apply to the Management Account itself |
| "GuardDuty runs in Management Account" | Use delegated admin — GuardDuty runs in Audit Account |
| "Config aggregator = Config rule enforcement" | Aggregator provides visibility — remediation still runs per account |
| "IAM Identity Center = Cognito" | Identity Center = workforce access to AWS accounts. Cognito = app user auth |
| "Consolidated billing = cost allocation" | Consolidated billing merges bills. Cost allocation requires activation of tags |
| "VPC Sharing shares the VPC" | VPC Sharing (via RAM) shares subnets, not the VPC itself |
| "Management Account should have workloads" | Management Account should have MINIMAL resources — high-risk if compromised |

---

Related: [[../../CHEATSHEET]] | [[../../Domain 1 - Secure Architectures/AWS Organizations]] | [[../../Domain 1 - Secure Architectures/IAM]]
#aws #architecture #multi-account #organizations #scp #saa-c03

# IAM (Identity and Access Management)

## Overview
IAM is a global service that controls access to AWS resources. It follows the principle of least privilege.

## Core Components

### Users
- Represent a person or application
- Have long-term credentials (password, access keys)
- Best practice: one physical user = one IAM user

### Groups
- Collection of IAM users
- Cannot be nested (no groups within groups)
- A user can belong to multiple groups

### Roles
- Intended for AWS services, applications, or federated users
- Provide temporary credentials via STS (Security Token Service)
- Common use cases: EC2 instance roles, Lambda execution roles, cross-account access

### Policies
- JSON documents that define permissions
- Types:
  - **Identity-based policies** — attached to users, groups, or roles
  - **Resource-based policies** — attached to resources (S3 bucket policy, SQS queue policy)
  - **Service Control Policies (SCPs)** — applied at AWS Organizations level
  - **Permission boundaries** — maximum permissions an identity can have

## Policy Evaluation Logic
1. Explicit Deny → always wins
2. SCP limits → applies if using Organizations
3. Resource-based policy → can allow cross-account
4. Permission boundary → caps what identity-based can grant
5. Identity-based policy → grants access
6. Default → implicit deny

## Best Practices
- Enable MFA for all users (especially root)
- Never use root account for daily tasks
- Use roles instead of sharing credentials
- Rotate access keys regularly
- Use IAM Access Analyzer to identify unused permissions
- Apply least privilege principle

## Federation
- **SAML 2.0** — enterprise identity providers (Active Directory)
- **Web Identity Federation** — social logins (use Cognito instead)
- **AWS IAM Identity Center** — successor to AWS SSO, centralized access management

## Key Exam Topics
- Difference between identity-based and resource-based policies
- When to use roles vs users
- Cross-account access patterns
- Policy evaluation logic
- MFA enforcement

## SAA Decision Addendum

### Choose This When
- **IAM role**: an AWS service, external account, federated user, or application needs temporary credentials.
- **IAM Identity Center**: employees need centralized SSO across many AWS accounts.
- **Resource policy**: the resource itself must trust another account, service, or principal.
- **Permission boundary**: a delegated admin can create roles/users but must not exceed a maximum permission set.
- **SCP**: an organization needs account-level guardrails; remember SCPs never grant access by themselves.

### Exam Traps
- Long-term access keys are rarely the best answer for applications running on AWS.
- Explicit deny beats every allow, including administrator-looking policies.
- IAM groups cannot be nested and generally cannot be named as principals in resource policies.
- Root user is not controlled by IAM policies; protect it with MFA and avoid routine use.
- `iam:PassRole` is high impact because it lets a principal attach a role to an AWS service.

---

Related: [[AWS Organizations]] | [[Cognito]]
#aws #iam #security #domain1

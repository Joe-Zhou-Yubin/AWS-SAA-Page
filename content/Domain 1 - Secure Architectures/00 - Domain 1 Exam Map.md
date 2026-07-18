# Domain 1 Exam Map - Design Secure Architectures

Weight: 30% of scored content.

Domain 1 is the biggest SAA-C03 domain. Think in three layers: identity, workload/network protection, and data protection. Most wrong answers either give too much privilege, expose something publicly, skip encryption/key ownership, or use a detective service where the question asks for preventive control.

## Official Task Alignment

| Task | What AWS expects you to design |
|------|--------------------------------|
| 1.1 Secure access to AWS resources | IAM, IAM Identity Center, federation, roles, SCPs, resource policies, root/MFA, least privilege |
| 1.2 Secure workloads and applications | VPC segmentation, security groups, NACLs, private endpoints, WAF, Shield, Secrets Manager, Cognito, GuardDuty, Macie |
| 1.3 Data security controls | KMS, ACM/TLS, encryption at rest/in transit, key policies, backups, replication, retention, classification |

## Fast Decision Rules

### Identity And Access
- Use IAM roles for AWS services, applications, cross-account access, and temporary credentials.
- Use IAM Identity Center for workforce SSO across many AWS accounts.
- Use Cognito for application users, customer sign-up/sign-in, and web/mobile identity federation.
- Use SCPs to set account-level guardrails in AWS Organizations. SCPs do not grant permissions.
- Use resource-based policies when the resource itself must trust another principal, account, service, or VPC endpoint.

### Network Security
- Use security groups for stateful instance or ENI-level allow rules.
- Use NACLs when the question needs subnet-level filtering or explicit deny by IP/CIDR.
- Use gateway endpoints for private S3/DynamoDB access from a VPC without NAT or internet gateway.
- Use interface endpoints/PrivateLink for most other private AWS service access.
- Use WAF for Layer 7 web threats such as SQL injection, XSS, path rules, IP sets, and rate-based rules.
- Use Shield Advanced when the question requires enhanced DDoS support, cost protection, or response-team help.

### Data Protection
- Use KMS for managed encryption keys with AWS service integration and key policies.
- Use CloudHSM when the requirement says single-tenant HSM, direct key control, or FIPS 140-2 Level 3.
- Use ACM for public/private TLS certificates on supported services such as ALB, NLB, API Gateway, and CloudFront.
- Use Secrets Manager for database credentials, automatic rotation, and secret lifecycle management.
- Use SSM Parameter Store for simpler configuration values and lower-cost secure strings.
- Use Macie when the task is discovering or classifying sensitive data in S3.

## Service Confusion Drills

| Requirement | Prefer | Avoid |
|-------------|--------|-------|
| Central multi-account permission guardrails | SCPs | IAM policy alone |
| Human SSO into many AWS accounts | IAM Identity Center | Long-term IAM users |
| App users sign in with social or enterprise IdP | Cognito | IAM users |
| Find public or cross-account resource exposure | IAM Access Analyzer | CloudTrail alone |
| Detect malicious account or workload activity | GuardDuty | Inspector |
| Scan EC2/ECR/Lambda vulnerabilities | Inspector | GuardDuty |
| Classify sensitive data in S3 | Macie | Security Hub |
| Aggregate security findings | Security Hub | Config alone |
| Track resource configuration drift | AWS Config | CloudTrail |
| Track API calls and who did what | CloudTrail | Config |

## Common Exam Traps
- Root user cannot be restricted by IAM policies; protect it with MFA and avoid daily use.
- Resource policies generally cannot grant permissions to IAM groups.
- A key policy must allow IAM policy use, or IAM permissions alone may not be enough for KMS.
- CloudTrail management events are enabled by default, but data events such as S3 object-level activity require explicit configuration and cost more.
- Security groups do not have deny rules. If the answer needs explicit deny, look at NACLs, SCPs, bucket policies, or WAF rules.
- NAT gateways do not accept security groups.
- CloudFront certificates must be in us-east-1.
- Encryption at rest does not automatically mean secure transport; use TLS/HTTPS for data in transit.

## Study Order
1. [[IAM]]
2. [[AWS Organizations]]
3. [[VPC Security]]
4. [[KMS]]
5. [[Secrets Manager]]
6. [[Certificate Manager]]
7. [[WAF & Shield]]
8. [[CloudTrail]], [[AWS Config]], [[GuardDuty]], [[Inspector]], [[Macie]], [[Security Hub]]
9. [[Cognito]]

## Scenario Prompts
- A company has hundreds of employees and multiple AWS accounts: choose IAM Identity Center plus Organizations/OUs/SCPs.
- A Lambda function needs to read DynamoDB: attach an execution role; do not embed access keys.
- A private subnet workload needs S3 without internet egress: use an S3 gateway endpoint and restrict the bucket policy with endpoint conditions.
- A public ALB receives SQL injection attempts: put AWS WAF on the ALB or CloudFront distribution.
- A compliance team requires immutable retained objects: use S3 Object Lock, retention mode, and lifecycle/backup policies.

---

Related: [[00 - Index]] | [[IAM]] | [[VPC Security]] | [[KMS]]
#aws #saa-c03 #domain1 #security

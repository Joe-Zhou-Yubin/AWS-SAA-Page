# AWS WAF & AWS Shield

## AWS WAF (Web Application Firewall)

### Overview
Protects web applications from common web exploits at Layer 7 (HTTP).

### Deployed On
- ALB (Application Load Balancer)
- API Gateway
- CloudFront
- AppSync GraphQL API
- Cognito User Pool

### Web ACL Rules
- **IP Set** — allow/block specific IPs
- **Rate-based rules** — DDoS protection (e.g., max 5 requests/sec per IP)
- **Managed Rule Groups** — AWS or Marketplace rules (OWASP top 10, bot control)
- **Custom rules** — SQL injection, XSS, geo-match, size constraints
- **Rule Groups** — reusable collections of rules

### Key Features
- Regional (except CloudFront — global)
- Logging to S3, CloudWatch Logs, or Kinesis Data Firehose
- Can use regex pattern matching

## AWS Shield

### Shield Standard (Free)
- Automatically enabled for all AWS customers
- Protects against Layer 3/4 DDoS attacks
- SYN/UDP floods, reflection attacks

### Shield Advanced ($3,000/month)
- Enhanced DDoS protection on EC2, ELB, CloudFront, Global Accelerator, Route 53
- 24/7 DDoS Response Team (DRT)
- Cost protection (refund for DDoS scaling costs)
- Advanced real-time metrics and reporting
- Automatic application layer mitigations (integrates WAF)

## AWS Firewall Manager
- Centrally manage WAF, Shield Advanced, Security Groups, Network Firewall across accounts
- Requires AWS Organizations
- Auto-applies rules to new resources

## Exam Tips
- WAF = Layer 7 (HTTP) protection
- Shield Standard = Layer 3/4, free, automatic
- Shield Advanced = enhanced DDoS, costly, includes DRT
- Use WAF rate-based rules for application-level DDoS
- Firewall Manager for organization-wide security policies

---

Related: [[VPC Security]] | [[Domain 3 - High-Performing Architectures/CloudFront]]
#aws #waf #shield #security #domain1

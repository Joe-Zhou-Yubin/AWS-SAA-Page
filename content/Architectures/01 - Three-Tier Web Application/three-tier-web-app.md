# Classic 3-Tier Web Application

AWS SAA-C03 Reference Architecture

---

## Architecture Overview

A production-grade, highly available 3-tier web application on AWS. Designed for internet-facing workloads requiring security, scalability, and resilience across multiple Availability Zones.

---

## Request Flow (Primary Path)

1. **End User** sends HTTPS request to the domain
2. **Route 53** resolves DNS → returns CloudFront IP. Health checks on origin; failover routing if primary is unhealthy
3. **AWS WAF** (attached to CloudFront) inspects the request — blocks SQL injection, XSS, geo-restricted IPs
4. **CloudFront** checks edge cache. Cache hit → returns content immediately. Cache miss → forwards to ALB
5. **ACM** provides the TLS certificate (must be in us-east-1 for CloudFront)
6. **S3 (Static Assets)** serves JS/CSS/images via CloudFront with Origin Access Control — bucket is private
7. **ALB** receives HTTPS:443, terminates TLS, applies routing rules (path/host-based), distributes to EC2 targets
8. **Auto Scaling Group** maintains desired EC2 fleet. Replaces unhealthy instances automatically
9. **EC2 Instances** process application logic. Retrieve DB credentials from Secrets Manager at startup
10. **ElastiCache Redis** — EC2 checks cache first before hitting Aurora (session data, repeated queries)
11. **Aurora MySQL** — primary writer handles writes; reader endpoint handles read queries
12. **Secrets Manager** provides auto-rotating DB credentials. EC2 IAM role grants GetSecretValue permission

---

## Monitoring & Audit Path (Secondary)

- ALB access logs → S3 (log archive)
- Aurora automated backups → S3
- EC2 metrics (CPU, memory via CW Agent) → CloudWatch
- CloudWatch Alarm → SNS → PagerDuty/email on threshold breach
- All API calls → CloudTrail → S3 (audit trail)

---

## Services Reference

| Service | Tier | Purpose |
|---------|------|---------|
| Route 53 | Edge | DNS resolution, health checks, failover routing |
| AWS WAF | Edge | Layer 7 request filtering (SQLi, XSS, geo-block) |
| CloudFront | Edge | CDN, HTTPS termination, edge caching |
| ACM | Edge | TLS certificate (must be us-east-1 for CF) |
| S3 (static) | Edge | Static asset hosting, private via OAC |
| ALB | Web | Layer 7 load balancing, path/host routing |
| NAT Gateway | Network | Private subnet outbound internet (one per AZ) |
| Auto Scaling Group | Web | Fleet management, health-based replacement |
| EC2 (t3.medium) | Web | Application server instances |
| IAM Role | Security | EC2 instance profile, least privilege |
| ElastiCache Redis | Cache | Session store, query result cache, sub-ms reads |
| Aurora MySQL | Data | Multi-AZ relational DB, auto-failover |
| Secrets Manager | Security | DB credentials with automatic rotation |
| CloudWatch | Ops | Metrics, alarms, dashboards |
| CloudTrail | Audit | API call audit log, all regions |
| SNS | Ops | Alert fan-out to ops team |

---

## Key Design Decisions

### Why CloudFront in front of the ALB?
- Caches static content at edge (~200+ global locations)
- WAF can be attached at CloudFront layer (lower cost than per-ALB WAF)
- S3 origin stays private via OAC
- Reduces origin load significantly for static assets

### Why ElastiCache before Aurora?
- Aurora reads: ~1-5ms. ElastiCache reads: ~0.1ms
- Repeated reads of product catalog, user sessions, config data hit cache first
- Reduces RCU consumption and Aurora CPU at scale

### Why Secrets Manager over env vars?
- Credentials never hardcoded or stored in code/AMI
- Auto-rotation for RDS (Lambda rotation built-in)
- Every credential access logged in CloudTrail

### Why NAT Gateway per AZ?
- Single NAT GW creates AZ dependency — if AZ-a fails, AZ-b loses internet too
- Cross-AZ data transfer charges avoided
- Per-AZ NAT costs more hourly but provides resilience

### Security Group Chain
```
Internet → ALB SG (443 from 0.0.0.0/0)
ALB SG → EC2 SG (80 from ALB SG only)
EC2 SG → Aurora SG (3306 from EC2 SG only)
```
Each layer only accepts traffic from the layer directly above it.

---

## Exam Traps This Architecture Addresses

| Trap | This Architecture Does |
|------|----------------------|
| "Multi-AZ standby serves reads" | Uses Aurora Reader Endpoint for reads, not Multi-AZ standby |
| "CloudFront cert in any region" | ACM cert provisioned in us-east-1 specifically |
| "EC2 memory in CloudWatch by default" | CloudWatch Agent installed for memory + disk metrics |
| "S3 origin public for CloudFront" | OAC keeps S3 private, only CloudFront can read |
| "NAT Gateway is always HA" | One per AZ pattern used to avoid AZ dependency |

---

Related: [[../../CHEATSHEET]] | [[../../Domain 2 - Resilient Architectures/RDS]] | [[../../Domain 2 - Resilient Architectures/Elastic Load Balancing]]
#aws #architecture #three-tier #saa-c03

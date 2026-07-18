# Amazon Route 53

## Overview
Highly available and scalable DNS web service. Also provides domain registration and health checking.

## Record Types
- **A** — hostname to IPv4
- **AAAA** — hostname to IPv6
- **CNAME** — hostname to hostname (cannot be used for zone apex)
- **Alias** — hostname to AWS resource (free, works for zone apex)

## Alias Records
- Point to AWS resources: ELB, CloudFront, S3 website, API Gateway, etc.
- Free of charge (no query fees)
- Native health check integration
- Cannot set TTL (auto-set by AWS)
- Cannot alias to an EC2 DNS name

## Routing Policies

| Policy | Use Case |
|--------|----------|
| Simple | Single resource, no health checks |
| Weighted | Split traffic by percentage |
| Latency-based | Route to lowest latency region |
| Failover | Active-passive with health checks |
| Geolocation | Route by user's continent/country |
| Geoproximity | Route by geographic distance (with bias) |
| Multi-Value | Return multiple healthy records |
| IP-based | Route based on client IP ranges |

## Health Checks
- Monitor endpoint health (HTTP, HTTPS, TCP)
- Calculated health checks (combine multiple checks)
- CloudWatch Alarm health checks (for private resources)
- Failure threshold and interval configurable
- ~15 global health checkers (healthy if >18% report healthy)

## Failover Routing
- Active-passive setup
- Primary record with health check
- Secondary (failover) record
- Automatic DNS failover on primary failure

## Exam Tips
- Alias over CNAME for AWS resources (free + zone apex support)
- Latency-based for global users (not geolocation)
- Geolocation for compliance/localization
- Failover for DR with health checks
- Health checks cannot access private endpoints directly (use CloudWatch Alarm)
- Route 53 is a global service

---

Related: [[Domain 2 - Resilient Architectures/Elastic Load Balancing]] | [[Disaster Recovery]]
#aws #route53 #dns #networking #domain2

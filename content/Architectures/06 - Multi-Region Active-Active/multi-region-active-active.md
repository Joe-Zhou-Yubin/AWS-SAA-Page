# Multi-Region Active-Active Architecture

AWS SAA-C03 Reference Architecture — 06

---

## Architecture Overview

Both `us-east-1` and `eu-west-1` run identical full stacks simultaneously. Users are routed to the nearest region by Global Accelerator and Route 53. Both regions accept reads and writes. Databases replicate bidirectionally (DynamoDB) or uni-directionally with fast promotion capability (Aurora Global DB).

---

## Request Flow

1. **Global Users** send requests to the application
2. **Global Accelerator** receives traffic at the nearest AWS edge location via static anycast IPs. Routes over the AWS backbone to the closest healthy regional endpoint. Failover takes < 30 seconds and bypasses DNS TTL caching
3. **Route 53** with latency-based routing resolves the domain to the lower-latency regional ALB endpoint
4. **CloudFront** serves cached static assets at the edge, forwards API/dynamic requests to the regional ALB
5. **WAF** (attached to CloudFront) filters SQLi, XSS, rate-based attacks
6. **ALB** distributes traffic across EC2 Auto Scaling Group instances in multiple AZs within the region
7. **EC2 ASG** processes application logic, writes to both **Aurora** and **DynamoDB** as needed
8. **Aurora Global Database**: us-east-1 is the writer; eu-west-1 has a read replica. < 1s replication lag. On DR: promote secondary to writer in < 1 minute
9. **DynamoDB Global Tables**: both regions accept reads AND writes simultaneously. Last-writer-wins conflict resolution
10. **S3 Cross-Region Replication (CRR)**: objects written in us-east-1 replicate to eu-west-1 automatically. Both buckets have versioning enabled

---

## Services Reference

| Service | Purpose |
|---------|---------|
| Global Accelerator | Static anycast IPs, AWS backbone routing, instant failover |
| Route 53 | Latency-based DNS routing, health checks per region |
| CloudFront | Edge caching, HTTPS, WAF attachment (both regions) |
| WAF | Layer 7 filtering on CloudFront (both regions) |
| ALB | Multi-AZ Layer 7 load balancing per region |
| EC2 Auto Scaling Group | App tier, multi-AZ, scales on demand |
| Aurora Global Database | Relational DB — one writer, one reader, < 1s replication |
| DynamoDB Global Tables | NoSQL — active-active writes, multi-region |
| ElastiCache Redis | Session store per region, Multi-AZ replica |
| S3 + CRR | Object storage, cross-region replication for durability |

---

## Key Design Decisions

### Why Global Accelerator AND Route 53?
They serve different purposes:

- **Route 53 latency routing** resolves DNS to the closest region. Cheap, works well for most traffic.
- **Global Accelerator** routes actual TCP traffic over the AWS backbone instead of the public internet. Benefits: static IPs (no DNS TTL issues), ~10-20ms latency improvement for distant users, instant failover without waiting for DNS TTL expiry.

For blue-green deployments or fast DR failover, Global Accelerator is critical — DNS changes take minutes to propagate globally due to TTL caching on mobile clients.

### Aurora Global DB vs DynamoDB Global Tables — why both?
They serve different data models:
- **Aurora Global DB**: for relational data requiring SQL, joins, ACID transactions. Only one region writes at a time (us-east-1). eu-west-1 is read-only — good for low-latency reads for European users. On DR, promote the secondary to writer in < 1 minute.
- **DynamoDB Global Tables**: for NoSQL key-value/document data (sessions, carts, user preferences). Both regions accept writes simultaneously — true active-active with last-writer-wins conflict resolution.

### Why S3 CRR?
Even if a region fails completely, objects stored in us-east-1 exist in eu-west-1. CRR requires versioning enabled on both buckets. New objects replicate within minutes. Existing objects before CRR was enabled require S3 Batch Replication.

### Aurora Global ≠ Multi-AZ
A common exam trap:
- **Multi-AZ**: standby in a different AZ within the SAME region. Automatic failover ~30-60s. Standby does NOT serve reads.
- **Aurora Global Database**: read replica in a DIFFERENT region. < 1s replication. Secondary serves reads. Promote to writer on regional DR.

### Cost Considerations
Going multi-region multiplies costs significantly:
- Aurora Global: full compute in each region + replication I/O
- DynamoDB Global Tables: each write costs × number of regions (3 regions = 3× write cost)
- EC2 ASG in each region: full fleet cost doubled
- CloudFront + WAF: per-region attachment costs

Typical result: **3-5× single-region cost**. Only architect multi-region when the business genuinely requires it (regulatory compliance, < 1 minute RTO, global user base with strict latency SLA).

---

## Disaster Recovery Scenario

If us-east-1 fails completely:

1. Route 53 health check detects failure → stops routing to us-east-1 (< 60s TTL)
2. Global Accelerator detects failure → redirects to eu-west-1 (< 30s)
3. Aurora Global DB: promote eu-west-1 secondary to writer (< 1 minute, RPO < 1 min)
4. DynamoDB Global Tables: eu-west-1 was already accepting writes — no action needed
5. ElastiCache: sessions in eu-west-1 are regional — new sessions created (some users re-authenticate)
6. Full regional traffic now served from eu-west-1

---

## Exam Traps This Architecture Addresses

| Trap | This Architecture Does |
|------|----------------------|
| "Aurora Global = Multi-AZ" | Global = multiple regions. Multi-AZ = multiple AZs in one region |
| "Route 53 for instant blue-green" | DNS TTL caching delays shifts — use Global Accelerator |
| "DynamoDB Global Tables = read replicas" | Global Tables = active-active WRITES in all regions |
| "Aurora Global secondary serves writes" | Secondary is READ-ONLY until promoted during DR |
| "CRR covers existing objects" | CRR only covers new objects — use S3 Batch Replication for existing |
| "Multi-region is always necessary" | Multi-region costs 3-5× more — only use when business requires it |

---

Related: [[../../CHEATSHEET]] | [[../../Domain 2 - Resilient Architectures/Aurora]] | [[../../Domain 2 - Resilient Architectures/DynamoDB]] | [[../../Domain 2 - Resilient Architectures/Disaster Recovery]]
#aws #architecture #multi-region #active-active #saa-c03 #dr

# Disaster Recovery Patterns

AWS SAA-C03 Reference Architecture — 07

---

## Overview

AWS defines four DR strategies. Each represents a different tradeoff between cost and recovery speed. The exam asks you to match a scenario (given RTO/RPO requirements and budget constraints) to the correct strategy.

**Golden rule:** Cost ↑ = RTO/RPO ↓

---

## The Four Strategies

### 1. Backup & Restore
**RTO: Hours | RPO: Hours | Cost: $ (cheapest)**

What the DR region has: **nothing running** — only S3 backups and IaC templates.

What you do on disaster:
1. Run CloudFormation / CDK to rebuild infrastructure in the DR region
2. Restore RDS from snapshot (snapshot copied to DR region via CRR)
3. Restore EC2 volumes from EBS snapshots
4. Update Route 53 DNS to point to new endpoints

Why it's slow: infrastructure rebuild takes time even with IaC. Large databases take time to restore from snapshot. Manual steps likely.

**Best practice:** Always pair with CloudFormation templates and tested runbooks. Backups without automation = very long RTO.

---

### 2. Pilot Light
**RTO: 10-30 minutes | RPO: Minutes | Cost: $$**

What the DR region has: **only the core data layer running** — DB replica is live and replicating. The application and web tiers are NOT running.

What you do on disaster:
1. Promote RDS read replica to standalone writer (minutes)
2. Run CloudFormation to spin up web and app tier in DR region
3. Update Route 53 DNS

Why it's faster than Backup & Restore: the database is already caught up — no restore from snapshot needed. App tier still needs to be started.

**Analogy:** The "pilot light" in a gas heater — a small flame is always burning so the boiler ignites quickly, but the heat isn't on yet.

---

### 3. Warm Standby
**RTO: Minutes | RPO: Seconds | Cost: $$$**

What the DR region has: **the full stack running at reduced capacity** — ASG with minimum 1 instance, Aurora reader, smaller instance sizes. Everything is running, just not at production scale.

What you do on disaster:
1. Scale up the ASG to full production size (Auto Scaling takes minutes)
2. Promote Aurora reader to writer (< 1 minute with Aurora Global)
3. Update Route 53 / Global Accelerator to DR region

Why it's faster than Pilot Light: no infrastructure startup time — it's already running. Just scale and promote.

**Key distinction from Pilot Light:** In Warm Standby, the **application tier is already running** (at low scale). In Pilot Light, only the data tier is running.

---

### 4. Active-Active (Multi-Site)
**RTO: < 30 seconds (Global Accelerator) | RPO: Near-zero | Cost: $$$$ (most expensive)**

What the DR region has: **full production stack at full production scale running simultaneously**.

What you do on disaster: nothing manual — Global Accelerator detects the unhealthy region and reroutes traffic automatically within seconds. Route 53 health checks update DNS within TTL.

DynamoDB Global Tables: already accepting writes in both regions — no action needed.

---

## Strategy Comparison Table

| Strategy | RTO | RPO | DR Region Has | Recovery Action | Cost |
|----------|-----|-----|---------------|----------------|------|
| Backup & Restore | Hours | Hours | S3 backups only | Deploy IaC + restore from snapshot | $ |
| Pilot Light | 10-30 min | Minutes | DB replica running; app tier stopped | Promote DB + deploy app tier | $$ |
| Warm Standby | Minutes | Seconds | Full stack scaled down | Scale up ASG + promote DB + update DNS | $$$ |
| Active-Active | < 30s | Near-zero | Full production stack in both regions | Automatic — GA routes away from failed region | $$$$ |

---

## RPO vs RTO — The Key Definitions

**RPO (Recovery Point Objective):** How much data can you lose? Measured as time before the disaster.
- RPO of 1 hour = you can accept losing up to 1 hour of data
- Shorter RPO → more frequent backups or synchronous replication
- Aurora Multi-AZ: RPO ≈ 0 (synchronous replication)
- Daily snapshot: RPO = up to 24 hours

**RTO (Recovery Time Objective):** How long can your system be down?
- RTO of 15 minutes = you must be back online within 15 minutes of disaster
- Shorter RTO → more pre-provisioned infrastructure in DR region
- Active-Active: RTO < 30 seconds (Global Accelerator)
- Backup & Restore: RTO = hours

---

## AWS Services That Enable Each Strategy

| DR Capability | AWS Service |
|---------------|------------|
| Centralised backup policy (EBS, RDS, EFS, DynamoDB, S3) | AWS Backup |
| Infrastructure as Code for DR rebuild | CloudFormation / CDK |
| DB snapshot cross-region copy | RDS / Aurora snapshot copy |
| S3 object cross-region replication | S3 CRR + Batch Replication |
| Relational DB read replica in DR region | RDS Read Replica or Aurora Global DB |
| Automatic DNS failover | Route 53 Failover routing + health checks |
| Sub-minute traffic failover (bypasses DNS TTL) | Global Accelerator |
| NoSQL active-active multi-region | DynamoDB Global Tables |

---

## Key Design Decisions

### Always pair backups with IaC
Backups protect data. Without automated infrastructure templates, the RTO is determined by how long it takes to manually recreate every resource. Even a "Backup & Restore" strategy benefits enormously from CloudFormation templates that can be triggered automatically.

### Warm Standby ≠ Pilot Light — know the difference
The exam frequently uses these interchangeably as distractors:
- **Pilot Light:** Only the data tier runs in DR. The web and app tiers must be started/deployed on failover.
- **Warm Standby:** The entire stack runs in DR at reduced scale. Just scale up and redirect traffic.

Ask yourself: "Is the application tier running in the DR region?" Yes = Warm Standby. No = Pilot Light.

### AWS Backup for centralised policy
Instead of configuring separate backup schedules for each service (RDS automated backups, EBS lifecycle policies, DynamoDB PITR), use AWS Backup to create a single backup plan that covers all supported services across multiple accounts.

---

## Exam Traps

| Trap | Reality |
|------|---------|
| "Warm Standby = Pilot Light" | Warm Standby has full stack running at reduced scale; Pilot Light only has core DB |
| "Backups guarantee low RTO" | Backups protect RPO, not RTO — fast recovery requires IaC + tested runbooks |
| "Route 53 failover is instant" | DNS TTL means failover takes minutes — use Global Accelerator for near-instant |
| "Active-Active requires no planning" | Must ensure both regions can handle full traffic load if one fails |
| "RDS Multi-AZ = DR strategy" | Multi-AZ is single-region HA (AZ failure), not DR (regional failure) |
| "Aurora Global = Active-Active" | Aurora Global DB is one writer + read replicas. DynamoDB Global Tables is truly active-active |

---

Related: [[../../CHEATSHEET]] | [[../../Domain 2 - Resilient Architectures/Disaster Recovery]] | [[06 - Multi-Region Active-Active/multi-region-active-active]]
#aws #architecture #disaster-recovery #rto #rpo #saa-c03

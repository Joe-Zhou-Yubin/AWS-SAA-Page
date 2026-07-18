# Disaster Recovery (DR)

## Overview
Strategies for recovering AWS workloads from catastrophic failures. Key metrics: RPO (Recovery Point Objective) and RTO (Recovery Time Objective).

## DR Strategies (Cost ↑, RTO ↓)

### 1. Backup & Restore
- **RTO**: Hours | **Cost**: Lowest
- Regular backups to S3/Glacier
- Restore infrastructure from backups when disaster strikes
- Use CloudFormation for infrastructure recreation
- Suitable for non-critical workloads

### 2. Pilot Light
- **RTO**: 10s of minutes | **Cost**: Low
- Core services always running (e.g., database replication)
- Other resources provisioned on demand during disaster
- Example: RDS Multi-Region read replica + stopped EC2 instances

### 3. Warm Standby
- **RTO**: Minutes | **Cost**: Medium
- Scaled-down but fully functional environment running
- Scale up during disaster (increase ASG, promote read replica)
- All services running but at minimum capacity

### 4. Multi-Site / Active-Active
- **RTO**: Near zero | **Cost**: Highest
- Full production environment in multiple regions
- Route 53 distributes traffic to both sites
- Immediate failover, no data loss
- Example: DynamoDB Global Tables + multi-region deployment

## Key AWS Services for DR

| Service | DR Role |
|---------|---------|
| S3 CRR | Cross-region data replication |
| RDS Read Replicas | Cross-region DB replication |
| Aurora Global Database | < 1 second cross-region replication |
| DynamoDB Global Tables | Active-active multi-region |
| Route 53 | DNS failover routing |
| CloudFormation | Infrastructure recreation |
| AWS Backup | Centralized backup management |
| EBS Snapshots | Volume backup and cross-region copy |

## RPO vs RTO
- **RPO** — how much data can you afford to lose? (time between last backup and disaster)
- **RTO** — how long can you be down? (time to restore service)

## Exam Tips
- Know the 4 strategies and their relative cost/RTO
- Active-Active = lowest RTO but highest cost
- Pilot Light vs Warm Standby: Warm Standby has everything running (just scaled down)
- Questions about "minimize cost" + DR → Backup & Restore or Pilot Light
- Questions about "minimize RTO" → Multi-Site Active-Active

## SAA Decision Addendum

### Choose This When
- **Backup and restore**: lowest cost is the priority and hours of downtime are acceptable.
- **Pilot light**: core data is replicated, but app capacity can be rebuilt during disaster.
- **Warm standby**: the full stack must already run at reduced scale.
- **Active-active**: near-zero downtime and global traffic distribution justify high cost.
- **Route 53 failover**: DNS-level failover is sufficient and health checks can detect failure.

### Exam Traps
- RTO is how long recovery takes; RPO is how much data loss is acceptable.
- Multi-AZ protects from AZ failure; it is not a complete Region disaster strategy.
- Backups without tested restore automation are not enough for low RTO.
- Service quotas in the standby Region can break DR if they are not pre-raised.

---

Related: [[Route 53]] | [[Aurora]] | [[DynamoDB]]
#aws #disaster-recovery #architecture #domain2

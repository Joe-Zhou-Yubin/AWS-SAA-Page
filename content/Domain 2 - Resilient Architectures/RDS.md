# Amazon RDS (Relational Database Service)

## Overview
Managed relational database service supporting multiple engines: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Aurora.

## Key Features
- Automated backups (point-in-time recovery, 1-35 days retention)
- Manual snapshots (no retention limit)
- Multi-AZ for high availability
- Read Replicas for read scalability
- Automated patching and maintenance
- Monitoring via CloudWatch and Enhanced Monitoring

## Multi-AZ Deployments
- **Synchronous** replication to standby in another AZ
- Automatic failover (DNS change, 60-120 seconds)
- Standby is NOT readable (just for failover)
- Zero data loss on failover
- Single DNS endpoint

## Read Replicas
- **Asynchronous** replication (eventual consistency)
- Up to 15 replicas (Aurora) or 5 (other engines)
- Can be in same AZ, cross-AZ, or cross-region
- Can be promoted to standalone database
- Applications must update connection string to use replicas
- No data transfer fee within same region, fee for cross-region

## Storage
- EBS-backed (gp2, gp3, io1)
- Storage Auto Scaling — automatically increases storage
- Maximum storage threshold configurable

## Security
- Encryption at rest (KMS) — must be set at creation
- In-transit encryption (SSL/TLS)
- IAM authentication (MySQL, PostgreSQL)
- Security groups for network access
- No SSH access (managed service)

## Backup & Restore
- Automated backups: daily full + transaction logs every 5 min
- Point-in-time restore: creates new DB instance
- Manual snapshots: persist until you delete them
- Can copy snapshots cross-region
- Can share encrypted snapshots (share KMS key)

## RDS Proxy
- Fully managed database proxy
- Connection pooling (reduces open connections)
- Reduces failover time by 66%
- Enforces IAM authentication
- Must be in VPC (never publicly accessible)

## Exam Tips
- Multi-AZ = availability (sync, automatic failover)
- Read Replicas = scalability (async, manual promotion)
- Cannot encrypt an existing unencrypted DB (snapshot → copy encrypted → restore)
- RDS Proxy for Lambda → RDS connections (prevents connection exhaustion)
- RDS Custom: for Oracle/SQL Server when you need OS access

## SAA Decision Addendum

### Choose This When
- **RDS Multi-AZ**: the requirement is availability/failover for one Region.
- **Read replica**: the requirement is read scaling or read-heavy reporting.
- **Cross-Region read replica**: the requirement is DR or lower-latency reads in another Region.
- **RDS Proxy**: Lambda or bursty apps create too many database connections.
- **Manual snapshot**: backup must survive DB deletion beyond automated retention.

### Exam Traps
- Standard Multi-AZ standby is not used for reads.
- Restoring from a snapshot creates a new DB instance and endpoint.
- Read replicas are asynchronous, so they can lag behind the writer.
- Storage autoscaling helps capacity, not query performance.
- For major scaling or faster failover, Aurora may be the better relational answer.

---

Related: [[Aurora]] | [[Domain 4 - Cost-Optimized Architectures/Database Cost Optimization]]
#aws #rds #database #domain2

# AWS Migration Services

## AWS Application Migration Service (MGN)
- Lift-and-shift (rehost) migration tool
- Continuous replication of source servers
- Minimal cutover window
- Replaces CloudEndure Migration
- Supports physical, virtual, and cloud servers

## AWS Database Migration Service (DMS)
- Migrate databases to AWS with minimal downtime
- Source stays operational during migration
- **Homogeneous** — same engine (MySQL → MySQL)
- **Heterogeneous** — different engine (Oracle → Aurora) — requires SCT first

### Schema Conversion Tool (SCT)
- Convert database schema between engines
- Identifies conversion issues
- Required for heterogeneous migrations

### Key Features
- Continuous data replication (CDC — Change Data Capture)
- Supports on-premises → AWS, AWS → AWS, AWS → on-premises
- Sources: Oracle, SQL Server, MySQL, PostgreSQL, MongoDB, S3, etc.
- Targets: RDS, Aurora, DynamoDB, S3, Redshift, OpenSearch, Kinesis

## AWS DataSync
- Online data transfer service
- Move large datasets to/from AWS
- Sources: NFS, SMB, HDFS, on-premises, S3, EFS, FSx
- Destinations: S3, EFS, FSx
- Automatic encryption, data validation
- Bandwidth throttling
- Scheduled or one-time transfers
- Agent-based (on-premises) or agentless (AWS-to-AWS)

## Migration Strategies (7 Rs)
1. **Rehost** — lift and shift (MGN)
2. **Replatform** — lift and reshape (e.g., to RDS)
3. **Repurchase** — move to SaaS
4. **Refactor** — re-architect for cloud-native
5. **Retire** — decommission
6. **Retain** — keep as-is (for now)
7. **Relocate** — move to VMware Cloud on AWS

## Exam Tips
- MGN for server migration (lift-and-shift)
- DMS for database migration (supports CDC for minimal downtime)
- SCT needed when changing database engines
- DataSync for bulk file/object transfer (NFS/SMB → S3/EFS)
- Know when to use each service based on source/target

---

Related: [[Snow Family]] | [[Domain 2 - Resilient Architectures/RDS]]
#aws #migration #dms #datasync #cross-cutting

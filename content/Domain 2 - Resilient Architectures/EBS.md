# Amazon EBS (Elastic Block Store)

## Overview
Block storage volumes for EC2 instances. Network-attached, persistent storage.

## Volume Types

| Type | Use Case | IOPS | Throughput |
|------|----------|------|------------|
| gp3 | General purpose SSD | 3,000 - 16,000 | 125 - 1,000 MB/s |
| gp2 | General purpose SSD | Burst to 3,000 (3 IOPS/GB) | 250 MB/s |
| io2 Block Express | Highest performance SSD | Up to 256,000 | 4,000 MB/s |
| io2/io1 | Provisioned IOPS SSD | Up to 64,000 | 1,000 MB/s |
| st1 | Throughput HDD | 500 | 500 MB/s |
| sc1 | Cold HDD (lowest cost) | 250 | 250 MB/s |

## Key Characteristics
- Locked to an Availability Zone
- Must snapshot to move across AZs or regions
- Can only attach to one EC2 instance (except io1/io2 multi-attach)
- Root volume deleted on termination by default (configurable)

## Snapshots
- Incremental backups (only changed blocks)
- Stored in S3 (managed by AWS)
- Can copy across regions
- EBS Snapshot Archive: 75% cheaper, 24-72h restore
- Recycle Bin: protect against accidental deletion

## Encryption
- AES-256 encryption at rest (uses KMS)
- Data in transit between instance and volume is encrypted
- Snapshots of encrypted volumes are encrypted
- To encrypt an unencrypted volume: snapshot → copy with encryption → create volume

## Multi-Attach (io1/io2 only)
- Attach same volume to multiple instances in same AZ
- Up to 16 instances
- Must use cluster-aware filesystem

## EBS vs Instance Store

| Feature | EBS | Instance Store |
|---------|-----|---------------|
| Persistence | Persistent | Ephemeral |
| Network | Network-attached | Physically attached |
| Performance | Good | Very high IOPS |
| Use case | General storage | Cache, temp data, buffers |

## Exam Tips
- gp3 is the default choice (independent IOPS and throughput)
- io2 for databases needing sustained high IOPS
- st1/sc1 cannot be boot volumes
- Multi-attach is rare but exam-relevant for shared storage scenarios
- Snapshots for backup, migration, DR

---

Related: [[Domain 3 - High-Performing Architectures/EC2]] | [[EFS]]
#aws #ebs #storage #domain2

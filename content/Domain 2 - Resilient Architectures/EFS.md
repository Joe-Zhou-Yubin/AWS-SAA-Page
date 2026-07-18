# Amazon EFS (Elastic File System)

## Overview
Managed NFS (Network File System) that can be mounted on many EC2 instances across AZs. POSIX-compliant.

## Key Features
- Multi-AZ (Regional) or One Zone
- Grows and shrinks automatically (no capacity planning)
- Pay per use (no pre-provisioning)
- Compatible with Linux-based AMIs only (not Windows)
- Encryption at rest (KMS) and in transit

## Performance Modes
- **General Purpose** (default) — latency-sensitive (web serving, CMS)
- **Max I/O** — higher latency, higher throughput/IOPS (big data, media processing)

## Throughput Modes
- **Bursting** — throughput scales with storage size
- **Provisioned (now called Elastic)** — set throughput independent of storage
- **Elastic** — automatically scales throughput up/down based on workload

## Storage Classes
- **Standard** — frequently accessed
- **Infrequent Access (IA)** — lower cost, retrieval fee
- **One Zone** — single AZ (lower cost, lower durability)
- **One Zone-IA** — lowest cost EFS option
- Lifecycle policies transition files between classes

## EFS vs EBS vs S3

| Feature | EFS | EBS | S3 |
|---------|-----|-----|-----|
| Type | File (NFS) | Block | Object |
| Multi-instance | Yes | No (except multi-attach io) | Yes (via API) |
| Multi-AZ | Yes | No (single AZ) | Yes |
| OS | Linux only | Any | N/A |
| Scaling | Automatic | Manual | Automatic |
| POSIX | Yes | N/A | No |

## Exam Tips
- EFS for shared file storage across instances/AZs
- Linux only (not Windows — use FSx for Windows)
- Use lifecycle policies to move to IA for cost savings
- Security: security groups control NFS access
- Higher cost than EBS but shared access is the differentiator

---

Related: [[EBS]] | [[Domain 3 - High-Performing Architectures/FSx]]
#aws #efs #storage #domain2

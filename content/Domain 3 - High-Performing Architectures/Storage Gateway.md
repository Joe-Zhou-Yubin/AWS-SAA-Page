# AWS Storage Gateway

## Overview
Hybrid cloud storage service. Bridges on-premises storage with AWS cloud storage (S3, EBS, Glacier).

## Gateway Types

### S3 File Gateway
- NFS / SMB access to S3 objects
- Local cache for frequently accessed data
- Objects stored as S3 objects (accessible via S3 API)
- Active Directory integration for SMB
- Use case: extend on-premises file shares to S3

### FSx File Gateway
- SMB access to FSx for Windows
- Local cache for frequently accessed data
- Use case: on-premises Windows applications accessing FSx

### Volume Gateway
- iSCSI block storage backed by S3
- **Cached Volumes** — primary data in S3, cache on-premises
- **Stored Volumes** — primary data on-premises, async backup to S3
- EBS snapshots for backup
- Use case: block storage with cloud backup

### Tape Gateway
- Virtual Tape Library (VTL) backed by S3/Glacier
- iSCSI-based virtual tape drives
- Compatible with existing backup software (NetBackup, Veeam, etc.)
- Use case: replace physical tape backups

## Key Architecture
```
On-Premises App → Storage Gateway (local cache) → AWS (S3/EBS/Glacier)
```

## Hardware Appliance
- When you can't run a VM on-premises
- Physical appliance shipped to you
- Works with all gateway types

## Exam Tips
- Storage Gateway = hybrid storage bridge
- S3 File Gateway for file-based access to S3
- Volume Gateway Cached = primarily cloud, cache locally
- Volume Gateway Stored = primarily local, backup to cloud
- Tape Gateway for tape backup replacement
- Know the difference between cached vs stored volumes

---

Related: [[Domain 2 - Resilient Architectures/S3]] | [[FSx]]
#aws #storage-gateway #hybrid #storage #domain3

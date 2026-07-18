# Amazon FSx

## Overview
Fully managed third-party file systems. Choose based on workload and OS requirements.

## FSx for Windows File Server
- Fully managed Windows-native file system (SMB protocol)
- Active Directory integration
- Supports Windows ACLs, DFS (Distributed File System)
- Multi-AZ for HA
- Use case: Windows workloads, SharePoint, SQL Server

## FSx for Lustre
- High-performance parallel file system
- Designed for HPC, ML, video processing
- Integrates with S3 (transparent data access)
- Deployment types:
  - **Scratch** — temporary, high burst (no replication)
  - **Persistent** — long-term, data replicated within AZ
- Hundreds of GB/s throughput, millions of IOPS

## FSx for NetApp ONTAP
- Managed NetApp ONTAP (NFS, SMB, iSCSI)
- Multi-protocol access
- SnapMirror replication
- Data deduplication and compression
- Works with Linux, Windows, macOS
- Use case: migration from on-premises NetApp

## FSx for OpenZFS
- Managed ZFS file system (NFS protocol)
- Snapshots, cloning, data compression
- Low-latency (sub-millisecond)
- Use case: Linux workloads migrating from on-premises ZFS

## Choosing the Right FSx

| Requirement | Choose |
|-------------|--------|
| Windows + SMB + AD | FSx for Windows |
| HPC, ML, big data | FSx for Lustre |
| Multi-protocol + NetApp | FSx for NetApp ONTAP |
| Linux + NFS + ZFS | FSx for OpenZFS |

## Exam Tips
- FSx for Windows = SMB + Active Directory
- FSx for Lustre = HPC + S3 integration
- FSx for Lustre Scratch = temporary, high performance
- All FSx options are managed (not EFS, not EBS)
- FSx for Lustre can read/write directly to S3

---

Related: [[Domain 2 - Resilient Architectures/EFS]] | [[Storage Gateway]]
#aws #fsx #storage #domain3

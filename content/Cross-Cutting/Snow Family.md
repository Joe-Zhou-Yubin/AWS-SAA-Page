# AWS Snow Family

## Overview
Physical devices for offline data transfer and edge computing when network transfer is impractical.

## Device Comparison

| Device | Storage | Compute | Use Case |
|--------|---------|---------|----------|
| Snowcone | 8 TB HDD / 14 TB SSD | 2 vCPU, 4 GB | Small edge, IoT |
| Snowball Edge Storage Optimized | 80 TB | 40 vCPU, 80 GB | Large data transfer |
| Snowball Edge Compute Optimized | 42 TB | 104 vCPU, 416 GB, GPU | Edge ML/compute |
| Snowmobile | 100 PB | None | Massive data center migration |

## Snowcone
- Smallest device (2.1 kg / 4.5 lbs)
- Rugged, portable
- DataSync Agent pre-installed
- Can send back to AWS or use DataSync over network
- Use: tactical edge, IoT, space-constrained environments

## Snowball Edge
- Large-scale data transfer (TB to PB)
- Can run EC2 instances and Lambda locally
- Cluster up to 15 devices for more storage
- Storage Optimized: more storage, less compute
- Compute Optimized: more compute, GPU option

## Snowmobile
- Exabyte-scale data transfer
- A literal shipping container on a truck
- 100 PB per Snowmobile
- Use: full data center migration (10+ PB)
- GPS tracking, 24/7 video surveillance, escort

## Process
1. Order device from AWS Console
2. Install on-premises, load data
3. Ship back to AWS
4. AWS uploads to S3
5. Device is wiped (NIST 800-88)

## When to Use Snow vs Network Transfer

| Data Size | Network (100 Mbps) | Network (1 Gbps) | Recommendation |
|-----------|--------------------|--------------------|----------------|
| 10 TB | 12 days | 30 hours | Network or Snowcone |
| 100 TB | 120 days | 12 days | Snowball Edge |
| 1 PB | 3+ years | 120 days | Snowball Edge cluster |

## Exam Tips
- Snow Family for large data transfers where network is too slow
- Rule of thumb: >1 week over network → consider Snow
- Snowball Edge also provides edge computing
- Snowmobile for >10 PB migrations
- Data is encrypted (KMS) on device
- Cannot transfer directly to Glacier (S3 first, then lifecycle)

---

Related: [[Migration Services]] | [[Domain 2 - Resilient Architectures/S3]]
#aws #snow-family #migration #edge #cross-cutting

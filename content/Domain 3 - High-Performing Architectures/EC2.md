# Amazon EC2 (Elastic Compute Cloud)

## Overview
Resizable compute capacity in the cloud. Virtual servers (instances) with full OS control.

## Instance Types

| Family | Use Case | Example |
|--------|----------|---------|
| General Purpose (M, T) | Balanced workloads | t3, m6i |
| Compute Optimized (C) | CPU-intensive | c6i, c7g |
| Memory Optimized (R, X, z) | In-memory databases | r6i, x2idn |
| Storage Optimized (I, D, H) | High sequential I/O | i3, d3 |
| Accelerated Computing (P, G, Inf) | ML, GPU | p4d, g5 |

## Placement Groups
- **Cluster** — single AZ, low latency, high throughput (HPC)
- **Spread** — instances on different hardware (max 7 per AZ, critical apps)
- **Partition** — isolated failure domains (big data: Hadoop, Kafka, Cassandra)

## Enhanced Networking
- **ENA (Elastic Network Adapter)** — up to 100 Gbps
- **EFA (Elastic Fabric Adapter)** — for HPC, MPI, bypasses OS kernel

## Instance Metadata
- `http://169.254.169.254/latest/meta-data/`
- Accessible from within the instance
- Contains: instance ID, security groups, IAM role credentials, etc.
- IMDSv2 recommended (token-based, more secure)

## User Data
- Script run at instance first boot
- Used for bootstrapping (install software, configure environment)
- Runs as root

## Hibernate
- RAM state saved to encrypted EBS root volume
- Faster restart (no OS boot)
- Must be enabled at launch, root volume must be encrypted
- Max 60 days hibernation

## AMI (Amazon Machine Image)
- Pre-configured instance template (OS, software, config)
- Regional (can copy cross-region)
- Use for golden images / fast launch

## Exam Tips
- Know instance families and when to use each
- Cluster placement group for HPC (low latency)
- Spread for individual instance HA
- EFA for MPI/HPC workloads
- Use AMIs + ASG for fast, consistent deployments
- EC2 Instance Connect / SSM Session Manager over bastion hosts

## SAA Decision Addendum

### Choose This When
- **Compute optimized**: CPU-bound workloads, batch processing, gaming servers, high-performance web tiers.
- **Memory optimized**: in-memory databases, caches, large JVM apps, real-time analytics.
- **Storage optimized**: high local I/O, distributed file systems, log processing, data warehousing nodes.
- **Cluster placement group**: lowest latency and highest throughput inside one AZ.
- **Spread placement group**: small number of critical instances must be on separate hardware.
- **Partition placement group**: large distributed systems need isolated failure partitions.

### Exam Traps
- EC2 is AZ-resilient only when you design it with Auto Scaling or failover across AZs.
- Instance store is high performance but ephemeral; stop/terminate events can lose data.
- User data runs at first boot by default, not every reboot unless configured.
- IMDSv2 is the preferred metadata access pattern for security.
- Vertical scaling means changing instance size; horizontal scaling means adding instances.

---

Related: [[Domain 2 - Resilient Architectures/EC2 Auto Scaling]] | [[Domain 4 - Cost-Optimized Architectures/EC2 Pricing]]
#aws #ec2 #compute #domain3

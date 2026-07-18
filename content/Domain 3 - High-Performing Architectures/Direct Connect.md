# AWS Direct Connect

## Overview
Dedicated private network connection from on-premises to AWS. Bypasses the public internet for more consistent performance.

## Connection Types
- **Dedicated Connection** — 1 Gbps, 10 Gbps, or 100 Gbps physical port
- **Hosted Connection** — 50 Mbps to 10 Gbps via AWS partner

## Virtual Interfaces (VIFs)
- **Private VIF** — access VPC resources (private IPs)
- **Public VIF** — access AWS public services (S3, DynamoDB) over private connection
- **Transit VIF** — access VPCs via Transit Gateway

## Key Features
- Lower latency than internet-based connections
- Consistent network performance
- Reduced data transfer costs (vs internet egress)
- Supports IEEE 802.1Q VLANs
- Takes weeks to establish (not instant)

## High Availability
- Single Direct Connect = single point of failure
- **HA options:**
  - Two Direct Connect connections (different locations)
  - Direct Connect + Site-to-Site VPN backup
  - Direct Connect Gateway for multi-region access

## Direct Connect Gateway
- Connect to VPCs in multiple regions through a single Direct Connect
- Works with Private VIF or Transit VIF

## Encryption
- Direct Connect traffic is NOT encrypted by default
- For encryption: establish Site-to-Site VPN over Direct Connect (Public VIF)
- Or use MACsec (Layer 2 encryption, dedicated connections only)

## Exam Tips
- Direct Connect for consistent, high-throughput private connectivity
- Lead time: weeks to months (not a quick-fix solution)
- Not redundant by itself — need backup connection
- Use with VPN for encryption + private connectivity
- Direct Connect Gateway for multi-region/multi-VPC access
- Site-to-Site VPN as backup or interim while DC is provisioned

---

Related: [[Transit Gateway]] | [[Domain 1 - Secure Architectures/VPC Security]]
#aws #direct-connect #networking #domain3

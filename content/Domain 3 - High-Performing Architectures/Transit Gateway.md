# AWS Transit Gateway

## Overview
Network transit hub that connects VPCs, on-premises networks, and other AWS accounts through a central gateway. Simplifies complex network topologies.

## Key Features
- Hub-and-spoke (star) architecture
- Transitive routing (unlike VPC peering)
- Supports thousands of VPC attachments
- Cross-account sharing via AWS RAM
- Cross-region peering
- Works with Direct Connect and VPN

## Attachments
- VPCs
- VPN connections
- Direct Connect Gateway
- Transit Gateway peering (cross-region)

## Route Tables
- Each attachment associates with a route table
- Control traffic flow between attachments
- Can create isolated or shared routing domains

## Use Cases
- Centralized egress (shared NAT Gateway)
- Centralized VPN/Direct Connect access
- Shared services VPC
- Multi-account network connectivity
- Network segmentation (separate route tables per environment)

## Transit Gateway vs VPC Peering

| Feature | Transit Gateway | VPC Peering |
|---------|----------------|-------------|
| Transitive | Yes | No |
| Scale | Thousands of VPCs | Limited |
| Bandwidth | 50 Gbps per attachment | No limit |
| Cost | Per attachment + data | Per data only |
| Complexity | Centralized | Point-to-point |

## Exam Tips
- Transit Gateway for complex multi-VPC networking
- VPC Peering for simple 1:1 connections (no transitive)
- Use with Direct Connect Gateway for hybrid multi-VPC
- ECMP (Equal Cost Multi-Path) for VPN bandwidth aggregation
- Share across accounts with AWS RAM
- IP Multicast support (unique to Transit Gateway)

---

Related: [[Direct Connect]] | [[Domain 1 - Secure Architectures/VPC Security]]
#aws #transit-gateway #networking #domain3

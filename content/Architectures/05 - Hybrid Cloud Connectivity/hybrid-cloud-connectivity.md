# Hybrid Cloud Connectivity

AWS SAA-C03 Reference Architecture — 05

---

## Architecture Overview

Enterprise hybrid architecture connecting an on-premises data centre to AWS over Direct Connect (primary) with Site-to-Site VPN as an encrypted backup path. Includes DNS resolution for on-premises hostnames, File Gateway for cloud-backed NFS/SMB storage, and VPC Endpoints to keep AWS service traffic off the NAT Gateway.

---

## Connectivity Paths

### Primary — Direct Connect
```
On-Premises → Physical cross-connect → DX Location → Direct Connect Gateway → VGW → VPC
```
- Dedicated private circuit — consistent latency, up to 100 Gbps
- Not encrypted by default — add VPN over DX or MACsec for encryption
- Takes weeks/months to provision
- BGP metric configured to prefer DX over VPN

### Backup — Site-to-Site VPN
```
On-Premises Customer Gateway → IPSec tunnels (×2) over internet → VGW → VPC
```
- Encrypted by default (IPSec)
- Provisions in hours
- Variable latency (internet-dependent)
- Activates automatically when DX BGP session drops

### VGW → VPC
- Virtual Private Gateway terminates both DX and VPN connections
- Attached to the VPC — routes propagated to private subnet route tables via BGP

---

## Components

### Direct Connect Gateway
Connects one Direct Connect to VPCs in **multiple AWS regions** via a single physical connection.

```
On-Premises → DX → DX Gateway → VGW (us-east-1 VPC)
                              → VGW (eu-west-1 VPC)
                              → VGW (ap-southeast-1 VPC)
```

Without DX Gateway: one physical DX per region. With DX Gateway: one physical DX, many regions.

**Critical limitation:** DX Gateway does NOT route VPC-to-VPC traffic. Use Transit Gateway if VPCs also need to communicate with each other through the same DX connection.

### Route 53 Resolver Outbound Endpoint
Allows EC2 instances in the VPC to resolve **on-premises hostnames** (e.g. `db.corp.internal`):

```
EC2 DNS query for db.corp.internal
  → VPC Resolver
  → Outbound Endpoint (ENI in private subnet)
  → Forwarding rule: corp.internal → 10.0.1.53 (on-prem DNS)
  → On-premises DNS server
  → Returns IP
```

**Inbound Endpoint** (reverse direction): On-premises resolves AWS private hosted zone records. On-prem DNS forwards `aws.internal` queries to the Inbound Endpoint IP over DX/VPN.

### File Gateway (Storage Gateway)
On-premises application writes to NFS/SMB share backed by S3. Recently accessed files cached locally for low latency. Full dataset stored durably in S3.

```
On-premises app → NFS mount → File Gateway appliance → S3 bucket (via DX/VPN)
```

### S3 Gateway Endpoint
Free route-table entry. Private subnet EC2 reaches S3 without going through NAT Gateway — eliminates NAT GW data processing charges ($0.045/GB) for all S3 traffic.

```
EC2 → route table → S3 Gateway Endpoint → S3
                 (no NAT GW, no internet)
```

---

## Services Reference

| Service | Purpose |
|---------|---------|
| Direct Connect | Dedicated private circuit, consistent bandwidth |
| Direct Connect Gateway | One DX to multiple regions/VPCs |
| Virtual Private Gateway (VGW) | AWS-side VPN/DX termination point on the VPC |
| Site-to-Site VPN | Encrypted backup path (2 tunnels) |
| Route 53 Resolver Outbound Endpoint | VPC → resolves on-premises DNS names |
| Route 53 Resolver Inbound Endpoint | On-premises → resolves AWS private DNS names |
| File Gateway | NFS/SMB on-prem backed by S3 |
| S3 Gateway Endpoint | Free private path from VPC to S3 |
| RDS | Private relational database in private subnets |
| EC2 | Application tier in private subnets |

---

## Key Design Decisions

### Why VPN as DX backup rather than a second DX?
A second DX is more resilient (dual physical paths) but takes months to provision and doubles the port cost. VPN activates within seconds via BGP failover and provides encrypted connectivity immediately. For most enterprises, VPN backup is the pragmatic choice while a second DX is arranged.

### Why NOT encrypt Direct Connect by default?
DX is a private dedicated circuit — it never traverses the public internet. For many organisations, physical isolation is sufficient. When encryption is required, options are:
- **VPN over DX** — IPSec tunnel on top of the DX connection (adds ~$0.05/hr VPN cost)
- **MACsec** — Layer 2 encryption on dedicated 10G/100G connections (newer, hardware-dependent)

### Why S3 Gateway Endpoint in a hybrid architecture?
Even with DX, private subnet EC2 traffic to S3 goes through NAT Gateway by default ($0.045/GB processed). A Gateway Endpoint adds a free route-table entry that sends S3-bound traffic directly — no NAT, no charge. At high S3 transfer volumes this saves significantly.

### Why File Gateway instead of direct S3 API calls from on-premises apps?
Legacy on-premises applications use NFS or SMB — they can't be modified to call S3 APIs. File Gateway presents a standards-based file interface locally while storing data durably in S3. No application changes required.

---

## DNS Resolution Quick Reference

| Direction | Endpoint Type | Use Case |
|-----------|--------------|---------|
| VPC → On-premises | Outbound Endpoint + forwarding rule | EC2 resolves `corp.internal` hostnames |
| On-premises → AWS | Inbound Endpoint | On-prem resolves AWS private hosted zones |
| VPC → AWS services | Default VPC resolver | Works automatically for all AWS service endpoints |

---

## Exam Traps This Architecture Addresses

| Trap | This Architecture Does |
|------|----------------------|
| "DX is encrypted" | DX is private but NOT encrypted by default — add VPN or MACsec |
| "DX Gateway enables VPC-to-VPC routing" | DX GW connects on-premises to multiple VPCs — does NOT route VPC-to-VPC |
| "Route 53 Inbound Endpoint for VPC → on-prem DNS" | Inbound = on-prem queries AWS. Outbound = VPC queries on-prem |
| "S3 Gateway Endpoint works over Direct Connect" | Gateway endpoints are VPC-local only — use Interface Endpoint for hybrid DX/VPN access to S3 |
| "One Direct Connect per region required" | DX Gateway lets ONE DX reach VPCs across multiple regions |
| "VPN is the same as Direct Connect" | VPN: encrypted, internet, hours to set up. DX: private, consistent, weeks to provision |

---

Related: [[../../CHEATSHEET]] | [[../../Domain 3 - High-Performing Architectures/Direct Connect]] | [[../../Domain 3 - High-Performing Architectures/Transit Gateway]]
#aws #architecture #hybrid #direct-connect #vpn #saa-c03

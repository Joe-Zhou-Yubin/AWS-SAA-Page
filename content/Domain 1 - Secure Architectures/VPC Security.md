# VPC Security

## Overview
Amazon VPC provides network-level security through multiple layers of controls.

## Security Groups
- **Stateful** — return traffic automatically allowed
- Applied at ENI / instance level
- Rules: allow only (no deny rules)
- Default: all inbound denied, all outbound allowed
- Can reference other security groups (self-referencing for intra-group communication)

## Network ACLs (NACLs)
- **Stateless** — must explicitly allow return traffic
- Applied at subnet level
- Rules: allow AND deny, evaluated by rule number (lowest first)
- Default NACL: allows all inbound and outbound
- Custom NACL: denies all by default

## Security Groups vs NACLs

| Feature | Security Groups | NACLs |
|---------|----------------|-------|
| Level | Instance (ENI) | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow & Deny |
| Evaluation | All rules evaluated | Rules in number order |
| Default | Deny in, allow out | Allow all (default NACL) |

## VPC Endpoints
- **Interface Endpoints (PrivateLink)** — ENI with private IP, most services
- **Gateway Endpoints** — route table entry, S3 and DynamoDB only (free)
- Keep traffic within AWS network, avoid internet

## VPC Flow Logs
- Capture IP traffic info (not packet content)
- Levels: VPC, Subnet, or ENI
- Destinations: CloudWatch Logs, S3, Kinesis Data Firehose
- Use for troubleshooting connectivity and monitoring

## Other Security Features
- **NAT Gateway/Instance** — outbound internet for private subnets
- **Bastion Host** — SSH/RDP access to private instances
- **VPC Peering** — connect VPCs (no transitive peering)
- **AWS PrivateLink** — expose service to other VPCs privately

## Exam Tips
- Security Groups are stateful, NACLs are stateless
- Use NACLs to block specific IPs (deny rules)
- Gateway Endpoints are free (S3, DynamoDB)
- Interface Endpoints cost money but support more services
- Flow Logs don't capture DNS, DHCP, metadata traffic

## SAA Decision Addendum

### Choose This When
- **Security group**: allow traffic to a specific workload, ENI, ALB, RDS instance, or interface endpoint.
- **NACL**: block a known bad CIDR or enforce subnet-level stateless rules.
- **Gateway endpoint**: private S3/DynamoDB access from a VPC with no NAT data-processing cost.
- **Interface endpoint**: private access to most AWS services through an ENI and security group.
- **PrivateLink**: expose a service privately to other VPCs/accounts without VPC peering or transitive routing.

### Exam Traps
- A private subnet is private because its route table lacks a route to an internet gateway, not because of its name.
- NAT gateway supports outbound IPv4 from private subnets, but it does not allow inbound initiation from the internet.
- NACL return traffic needs ephemeral ports explicitly allowed.
- VPC peering is not transitive; use Transit Gateway for hub-and-spoke designs.
- Flow Logs show metadata about accepted/rejected flows, not packet payloads.

---

Related: [[WAF & Shield]] | [[Domain 3 - High-Performing Architectures/Transit Gateway]]
#aws #vpc #networking #security #domain1

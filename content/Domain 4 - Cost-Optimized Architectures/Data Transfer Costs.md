# Data Transfer Costs

## Overview
Data transfer is a hidden cost that many architects overlook. Understanding transfer pricing is essential for cost optimization.

## General Rules
- **Data IN (ingress)** to AWS — always FREE
- **Data OUT (egress)** to internet — charged per GB
- **Same AZ** — free (using private IPs)
- **Cross-AZ** — $0.01/GB each direction (use private IPs to avoid more)
- **Cross-Region** — $0.02/GB (varies by region)

## Service-Specific Transfer

### S3
- Ingress: Free
- To Internet: ~$0.09/GB (first 10TB)
- To CloudFront: Free
- Cross-Region Replication: data transfer charged
- VPC Gateway Endpoint: Free (S3 to EC2 in same region)

### EC2
- Same AZ (private IP): Free
- Same AZ (public/Elastic IP): $0.01/GB
- Cross-AZ: $0.01/GB per direction
- To Internet: ~$0.09/GB

### NAT Gateway
- Processing charge: $0.045/GB processed
- Plus standard data transfer charges
- Can be significant cost for high-traffic workloads

## Cost Reduction Strategies
1. **VPC Endpoints** — avoid NAT Gateway charges for S3/DynamoDB
2. **CloudFront** — cheaper egress than direct from S3/EC2
3. **Same-AZ placement** — eliminate cross-AZ charges (trade-off: lower HA)
4. **AWS Direct Connect** — lower data transfer rates than internet
5. **Compression** — reduce bytes transferred
6. **S3 Transfer Acceleration** — faster but costs more
7. **Regional services** — keep data in same region as compute

## Common Exam Patterns
- "Reduce data transfer costs" → VPC Endpoints, CloudFront, same-region
- "High NAT Gateway costs" → VPC Gateway Endpoint for S3/DynamoDB
- "Cross-region replication costs" → understand it's not free
- "Minimize cost for data delivery" → CloudFront vs direct S3

## Exam Tips
- Always consider data transfer in architecture decisions
- VPC Gateway Endpoints (S3, DynamoDB) eliminate NAT charges
- CloudFront egress is cheaper than S3 direct egress
- Private IPs between instances reduce or eliminate cross-AZ costs
- Direct Connect has lower data transfer rates than internet

## SAA Decision Addendum

### Choose This When
- **CloudFront**: public internet users repeatedly access cacheable content.
- **S3/DynamoDB gateway endpoint**: private subnet workloads access those services through NAT.
- **Interface endpoint**: private service access is required and the endpoint cost is justified.
- **Direct Connect**: high-volume, predictable hybrid traffic needs consistent performance.
- **Same-AZ placement**: chatty components can safely be co-located to avoid cross-AZ transfer.

### Exam Traps
- Cross-AZ traffic can be introduced by NAT gateways, load balancers, replicas, and mismatched subnet placement.
- Multi-AZ may cost more, but do not remove it when HA is a stated requirement.
- Regional data transfer and internet egress are different cost categories.
- A centralized NAT gateway can reduce hourly cost but adds AZ dependency and possible cross-AZ data charges.

---

Related: [[S3 Cost Optimization]] | [[Cost Management Tools]]
#aws #data-transfer #cost-optimization #networking #domain4

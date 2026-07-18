# Domain 4 Exam Map - Design Cost-Optimized Architectures

Weight: 20% of scored content.

Domain 4 tests whether you can make tradeoffs without breaking requirements. Cost answers should preserve stated availability, durability, performance, compliance, and operational constraints. The cheapest answer is wrong when it violates the workload's access pattern, RTO/RPO, or supportability needs.

## Official Task Alignment

| Task | What AWS expects you to design |
|------|--------------------------------|
| 4.1 Storage cost | Storage class, lifecycle, migration, backups, auto scaling, Requester Pays, object/file/block choice |
| 4.2 Compute cost | Purchasing options, instance family/size, scaling, serverless, containers, hibernation, availability class |
| 4.3 Database cost | Capacity mode, engine choice, retention, replicas, caching, serverless, migrations |
| 4.4 Network cost | NAT strategy, VPC endpoints, transfer paths, Direct Connect vs VPN, CloudFront/edge caching, throttling |

## Fast Decision Rules

### Compute
- Use On-Demand for short-lived, unpredictable, or initial workloads.
- Use Savings Plans for steady compute usage when flexibility matters.
- Use Reserved Instances for steady RDS/ElastiCache/OpenSearch and for EC2 when the commitment shape is clear.
- Use Spot for fault-tolerant, interruptible, stateless, or checkpointed work.
- Use Auto Scaling to stop paying for unused capacity.
- Use Lambda or Fargate when utilization is variable and operational overhead matters.
- Use Compute Optimizer and rightsizing for overprovisioned EC2, EBS, Lambda, and Auto Scaling resources.

### Storage
- Use S3 lifecycle policies when access patterns become colder over time.
- Use S3 Intelligent-Tiering when access patterns are unknown or change unpredictably.
- Use S3 Standard-IA for infrequently accessed data that still needs multi-AZ durability and millisecond access.
- Use S3 One Zone-IA only for reproducible or secondary-copy data.
- Use Glacier classes for archive data based on retrieval-time requirements.
- Use gp3 instead of gp2 when you need independent baseline IOPS/throughput tuning at lower cost.
- Delete unattached EBS volumes and stale snapshots after validating retention requirements.

### Databases
- Use DynamoDB on-demand for unpredictable or spiky traffic.
- Use DynamoDB provisioned capacity with auto scaling for predictable workloads.
- Use Aurora Serverless for variable relational workloads that can tolerate the serverless behavior.
- Use RDS Reserved Instances for stable database usage.
- Use read replicas only when they reduce read pressure enough to justify cost.
- Use caching when it reduces expensive database reads and does not violate freshness requirements.

### Network
- Avoid cross-AZ and cross-Region transfer unless the architecture requires it.
- Put high-volume internet content behind CloudFront when caching is possible.
- Use S3 gateway endpoints and DynamoDB gateway endpoints to avoid NAT gateway data processing for private subnet access.
- Use interface endpoints when private service access is required, but remember they add hourly and data processing cost.
- Centralized NAT can be cheaper but less resilient; one NAT gateway per AZ is more resilient but costs more.
- Use Direct Connect for predictable high-volume hybrid transfer; use VPN for lower-cost encrypted connectivity with less predictable performance.

## Cost Tooling

| Need | Tool |
|------|------|
| Explore historical spend | Cost Explorer |
| Alert on thresholds or forecasts | AWS Budgets |
| Detailed billing data for analysis | Cost and Usage Report |
| Central billing across accounts | AWS Organizations consolidated billing |
| Rightsizing recommendations | Compute Optimizer |
| Savings recommendations and cost checks | Trusted Advisor, Cost Explorer, Savings Plans recommendations |
| Attribute spend to teams/apps | Cost allocation tags |

## Common Exam Traps
- Spot is not for stateful databases or workloads that cannot tolerate interruption.
- Standard-IA and One Zone-IA have retrieval costs and minimum storage duration considerations.
- Intelligent-Tiering has monitoring/automation fees, so it is not always best for tiny short-lived objects.
- Cross-AZ data transfer can appear accidentally through load balancers, NAT gateways, database replicas, and misaligned subnets.
- NAT gateways charge hourly and for data processing; S3/DynamoDB gateway endpoints can remove that path.
- Reserved capacity and Savings Plans reduce cost but do not automatically guarantee capacity unless paired with the right reservation type.
- Cost optimization should not remove required Multi-AZ, backups, encryption, or retention.

## Study Order
1. [[EC2 Pricing]]
2. [[S3 Cost Optimization]]
3. [[Database Cost Optimization]]
4. [[Data Transfer Costs]]
5. [[Cost Management Tools]]

## Scenario Prompts
- Predictable EC2 spend across families and Regions: Compute Savings Plan.
- Predictable database workload: RDS Reserved Instance.
- Interruptible image processing: Spot fleet or Spot in Auto Scaling, with checkpointing if needed.
- Unknown object access pattern: S3 Intelligent-Tiering.
- Private EC2 instances download many objects from S3 through NAT: add S3 gateway endpoint.
- Expensive internet egress from S3 static assets: use CloudFront caching.

---

Related: [[00 - Index]] | [[EC2 Pricing]] | [[S3 Cost Optimization]] | [[Data Transfer Costs]]
#aws #saa-c03 #domain4 #cost-optimization

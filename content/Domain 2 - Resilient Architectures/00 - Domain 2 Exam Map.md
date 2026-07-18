# Domain 2 Exam Map - Design Resilient Architectures

Weight: 26% of scored content.

Domain 2 asks whether the architecture keeps working when demand changes, a component fails, an Availability Zone fails, or a Region-level event occurs. The exam favors managed services, loose coupling, stateless compute, automated recovery, and data replication that matches RPO/RTO.

## Official Task Alignment

| Task | What AWS expects you to design |
|------|--------------------------------|
| 2.1 Scalable and loosely coupled architectures | Multi-tier design, SQS/SNS/EventBridge, API Gateway, containers, serverless, caching, horizontal scaling |
| 2.2 Highly available and fault-tolerant architectures | Multi-AZ, Multi-Region, Route 53, load balancing, DR strategies, backups, proxies, quotas, failover |

## Fast Decision Rules

### Compute And Traffic
- Use EC2 Auto Scaling across at least two AZs for self-healing EC2 fleets.
- Use ALB for HTTP/HTTPS routing, host/path rules, and target groups.
- Use NLB for ultra-low latency TCP/UDP/TLS and static IP requirements.
- Use Gateway Load Balancer for transparent inspection appliances.
- Use health checks and grace periods so Auto Scaling replaces genuinely unhealthy instances.
- Use immutable infrastructure patterns with launch templates, AMIs, and automation instead of patching individual servers.

### Storage And Data
- Use S3 for regional, multi-AZ durable object storage; add versioning, replication, Object Lock, and lifecycle policies when needed.
- Use EBS snapshots to protect zonal block volumes and recreate volumes in another AZ or Region.
- Use EFS for shared Linux file storage across multiple AZs.
- Use FSx when the protocol or workload is specialized, such as Windows SMB or Lustre HPC.
- Use AWS Backup for centralized backup policy management across supported services.

### Databases
- Use RDS Multi-AZ for HA failover, not read scaling.
- Use RDS read replicas for read scaling; cross-Region replicas can support DR.
- Use Aurora when the question wants higher availability, fast replica failover, or a managed relational engine with distributed storage.
- Use Aurora Global Database when cross-Region low-latency reads and DR are key.
- Use DynamoDB Global Tables for active-active multi-Region NoSQL.
- Use RDS Proxy when many short-lived connections, Lambda bursts, or failover connection handling are concerns.

### Decoupling
- Use SQS to buffer work between producers and consumers.
- Use SNS to fan out notifications to multiple subscribers.
- Use SNS plus SQS when each consumer needs its own durable queue.
- Use EventBridge for event bus routing, SaaS/AWS service events, schemas, archives, and loosely coupled application events.
- Use Step Functions for workflow orchestration and retries across services.

## DR Ladder

| Strategy | Relative cost | Relative RTO | Exam signal |
|----------|---------------|--------------|-------------|
| Backup and restore | Lowest | Hours | Non-critical, cheapest, restore from backup |
| Pilot light | Low | Tens of minutes | Core data services running, app infra rebuilt/scaled during disaster |
| Warm standby | Medium | Minutes | Full stack running at reduced size |
| Multi-site active-active | Highest | Near zero | Full production in multiple Regions, global traffic distribution |

## Common Exam Traps
- Multi-AZ is usually the first answer for AZ failure; Multi-Region is for regional disaster, global latency, or strict DR.
- RDS standby instances in standard Multi-AZ are not readable.
- S3 replication needs versioning and is not retroactive unless you use batch replication.
- SQS Standard provides at-least-once delivery, so consumers must be idempotent.
- FIFO queues preserve ordering by message group and have lower throughput unless high-throughput FIFO is configured.
- Route 53 DNS failover depends on health checks and DNS TTL; it is not an instant connection-level failover tool.
- Read replicas improve reads, not writes.
- Caches improve performance but are not the system of record.

## Study Order
1. [[Elastic Load Balancing]]
2. [[EC2 Auto Scaling]]
3. [[S3]]
4. [[EBS]], [[EFS]]
5. [[RDS]], [[Aurora]], [[DynamoDB]], [[ElastiCache]]
6. [[Route 53]]
7. [[SQS]], [[SNS]], [[EventBridge]]
8. [[Disaster Recovery]]

## Scenario Prompts
- Web tier must survive one AZ failure: ALB plus Auto Scaling group across multiple AZs.
- Workers fall behind during traffic spikes: SQS between producers and consumers; scale consumers by queue depth.
- Each downstream team needs independent message processing: SNS fan-out to separate SQS queues.
- Relational database needs HA but no extra read traffic: RDS Multi-AZ.
- Global users need low-latency reads and regional DR: Aurora Global Database or DynamoDB Global Tables depending on data model.

---

Related: [[00 - Index]] | [[Elastic Load Balancing]] | [[SQS]] | [[Disaster Recovery]]
#aws #saa-c03 #domain2 #resilience

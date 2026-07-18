# AWS SAA-C03 Scenario Cheatsheet

> **How to use this:** Each section maps a scenario keyword → decision → tradeoffs. When you see a question, find the matching keyword and follow the decision logic.

---

## Table of Contents
1. [[#Security & Access Control]]
2. [[#Encryption & Key Management]]
3. [[#Networking & Connectivity]]
4. [[#Compute — EC2 & Lambda]]
5. [[#Containers]]
6. [[#Storage — S3]]
7. [[#Storage — Block & File]]
8. [[#Databases — Relational]]
9. [[#Databases — NoSQL & Caching]]
10. [[#Messaging & Event-Driven]]
11. [[#High Availability & Auto Scaling]]
12. [[#Disaster Recovery]]
13. [[#Performance & Global Delivery]]
14. [[#Analytics & Big Data]]
15. [[#Cost Optimization]]
16. [[#Hybrid & Migration]]
17. [[#Monitoring, Audit & Compliance]]
18. [[#Quick-Fire Service Selector]]

---

## Security & Access Control

### Scenario: EC2 / Lambda needs to access AWS services (S3, DynamoDB, etc.)
→ **IAM Role + Instance Profile** (never store access keys in user data, environment vars, or S3)
- Cross-account: role + **bucket policy must also grant access** to the execution role
- EC2 instance profile is the container for the role; a profile holds only one role

### Scenario: Prevent member accounts from disabling CloudTrail / making S3 public
→ **SCP (Service Control Policy) at OU level** via AWS Organizations
- SCPs block even account admins with full IAM
- Permission boundaries cap what a *single identity* can do inside one account — good for delegated role creation

### Scenario: Restrict which countries can access a public web app on ALB
→ **AWS WAF Geo Match** on the ALB
- CloudFront Geo Restriction works only when CloudFront is in front (not a VPC-resident service)
- Security Groups / NACLs cannot filter by geography

### Scenario: Restrict streaming content rights by geography (two correct answers common)
→ **CloudFront Geo Restriction** + **Route 53 Geolocation Routing**
- Weighted, Latency, Failover routing do NOT restrict content rights

### Scenario: Block a specific malicious IP on a CloudFront + ALB setup
→ **AWS WAF IP Match Condition**
- NACLs associate with subnets, not instances; Security Groups have no deny rules

### Scenario: IAM policy evaluation — explicit Deny always wins
→ If a Deny statement exists (e.g. `StringNotEquals` on region), it blocks all listed regions even if an Allow also exists
- Allow only fires when Deny does not apply

### Scenario: Cross-account S3 access (Lambda in Account A, bucket in Account B)
→ IAM execution role (Account A) **AND** S3 bucket policy (Account B) must both grant access
- Same-account: role alone is sufficient

### Scenario: Private application in VPC calls AWS APIs without NAT / internet
→ **Interface VPC Endpoint** (PrivateLink) for the specific service (Secrets Manager, SSM, KMS, etc.)
- S3 and DynamoDB use **Gateway Endpoints** (free, route-table entries)
- Gateway endpoints only cover S3 + DynamoDB; everything else needs Interface endpoints

### Scenario: Restrict S3 access to a specific VPC endpoint only
→ **S3 Bucket Policy** with `aws:sourceVpce` condition

### Scenario: S3 bucket accessed by many apps — fine-grained per-prefix access at scale
→ **S3 Access Points** (one per service, each with its own policy scoped to a prefix)
- Single flat bucket policy with all ARNs listed does not scale well

### Scenario: API Gateway must only accept requests from internal IP ranges
→ **API Gateway Resource Policy** with IpAddress / NotIpAddress conditions
- API Gateway is a managed service — no security groups, no subnets to attach

### Scenario: Temporary upload access to a private S3 object
→ **S3 Pre-Signed URL** (time-limited, no permanent credentials exposed)

### Scenario: Identify resources shared with external AWS accounts
→ **IAM Access Analyzer**

### Scenario: Centrally deploy WAF rules across many accounts / ALBs in an Org
→ **AWS Firewall Manager**

### Scenario: Managed network-layer filtering (stateful, domain filtering, IDS/IPS) inside VPC
→ **AWS Network Firewall** (not WAF — WAF is Layer 7 HTTP only)


---

## Encryption & Key Management

### Scenario: Encrypt S3 data, need audit trail of key usage
→ **SSE-KMS** (logs every KMS API call in CloudTrail)
- SSE-S3: AWS manages keys, no customer key audit trail
- SSE-C: customer provides keys, no AWS managed rotation

### Scenario: Encrypt S3 + annual automatic key rotation + audit
→ **SSE-KMS with automatic key rotation enabled**
- Manual rotation exists but is not operationally efficient
- SSE-C requires you to build your own rotation automation

### Scenario: Separation of duties — S3 admin can manage objects but not read plaintext
→ **SSE-KMS with customer managed key** — S3 admin needs separate KMS decrypt permission

### Scenario: Dedicated single-tenant HSM, full control of key material
→ **AWS CloudHSM** (not KMS — KMS is shared hardware, AWS controls the control plane)

### Scenario: Automatic RDS password rotation without custom code
→ **AWS Secrets Manager** (built-in rotation for RDS, Redshift, DocumentDB)
- Systems Manager Parameter Store can store secrets but rotation requires custom Lambda

### Scenario: Private internal CA for internal service certificates
→ **AWS Private CA** (formerly ACM Private CA)

### Scenario: Encrypt an existing unencrypted RDS database
→ Cannot enable encryption directly. Steps: **Snapshot → Copy snapshot with encryption → Restore from encrypted snapshot → Migrate → Delete old DB**
- Read replicas cannot be encrypted if the primary is unencrypted
- Multi-AZ does not help with encryption

### Scenario: CloudFront viewer HTTPS certificate (custom domain)
→ ACM certificate **must be in us-east-1** regardless of origin region

### Scenario: Share encrypted RDS snapshot with external auditor
→ **Encrypt the snapshot → share snapshot → share the KMS key** with the auditor's AWS account via key policy

### Scenario: S3 cross-region replication with same encryption key in both regions
→ Use an **AWS KMS multi-region key** (same key ID usable in multiple regions)


---

## Networking & Connectivity

### Scenario: Private subnet instances need internet access (IPv4, fully managed)
→ **NAT Gateway** in the **public subnet**
- NAT instance: not managed, you patch it — only if question says "self-managed" or cost is paramount
- Egress-only Internet Gateway: IPv6 only — never for IPv4

### Scenario: High availability — private subnets in 3 AZs need internet access
→ **One NAT Gateway per AZ** (each private route table points to its own NAT GW)
- One NAT GW across AZs creates AZ dependency + cross-AZ data transfer charges
- Tradeoff: per-AZ costs more hourly but avoids cross-AZ charges and improves resilience

### Scenario: IPv6-only private subnet needs outbound internet, block inbound initiation
→ **Egress-Only Internet Gateway** (not NAT gateway — NAT is IPv4 only)

### Scenario: Many VPCs + on-premises need scalable hub-and-spoke routing
→ **AWS Transit Gateway**
- Full mesh VPC peering is non-transitive and grows O(n²); Transit Gateway scales linearly
- VPC peering is fine for a small number of VPCs

### Scenario: Cross-account VPC service sharing without full VPC exposure
→ **AWS PrivateLink** (expose via NLB → Interface VPC Endpoint in consumer account)
- VPC peering exposes the whole VPC CIDR range, PrivateLink exposes only the service

### Scenario: Share subnets across accounts in the same Org (not share the VPC itself)
→ **VPC Sharing** via Resource Access Manager (RAM)
- Shares *subnets*, not the VPC
- VPC peering is a network routing connection, not a resource sharing mechanism

### Scenario: Consistent high-throughput private connectivity from on-premises to AWS
→ **AWS Direct Connect**
- Site-to-Site VPN: encrypted over internet, variable latency
- Direct Connect: dedicated, consistent, not natively encrypted (add VPN over DX for encryption)

### Scenario: Hybrid DNS — VPC queries must resolve on-premises hostnames
→ **Route 53 Resolver Outbound Endpoint** with a forwarding rule to on-premises DNS
- Inbound endpoint: on-premises queries resolve AWS hostnames (opposite direction)
- Do NOT create a private hosted zone manually duplicating on-premises records — operational overhead

### Scenario: Direct Connect needs to connect to VPCs in multiple Regions
→ **Direct Connect Gateway** (connects one DX to multiple VPCs across regions via VGWs or TGW)

### Scenario: Security appliances (NGFW, IDS) need to inspect traffic transparently
→ **Gateway Load Balancer (GWLB)** — inserts appliances transparently in data path
- ALB: Layer 7 HTTP routing; NLB: Layer 4 TCP/UDP performance; GWLB: appliance chaining

### Scenario: NLB target registered by instance ID — how does it route?
→ Uses the **primary private IP** of the primary network interface (rewrites destination IP)
- Not the public IP, not the elastic IP, not the instance ID itself


---

## Compute — EC2 & Lambda

### Scenario: HPC workload — lowest inter-node latency in one AZ
→ **Cluster Placement Group**
- Spread: max failure isolation (max 7 per AZ), not for performance
- Partition: large distributed systems (Hadoop, Kafka) — fault domain separation, not lowest latency

### Scenario: Lambda cold starts causing high p95 latency for a latency-sensitive API
→ **Provisioned Concurrency** (keeps environments pre-initialized)
- Reserved concurrency limits max concurrent executions but does not eliminate cold starts

### Scenario: Lambda + SQS — AWS handles polling automatically
→ **Event Source Mapping** (Lambda polls SQS/Kinesis/DynamoDB Streams in batches)
- Function URL: HTTPS endpoint — no queue polling
- Lambda Destinations: route async invocation outcomes to another service

### Scenario: Lambda + RDS — too many database connections during bursts
→ **RDS Proxy** (pools connections, queues requests, reduces connection storms)

### Scenario: Lambda memory optimization
→ **Test different memory sizes** — more memory = more CPU = potentially faster = lower billed duration
- Lowest memory is NOT always cheapest; test for best duration × memory product

### Scenario: Scheduled serverless job (weekly cron, ~5 minutes)
→ **EventBridge Scheduler (cron expression) → Lambda**
- EC2 spot/reserved instances: not serverless
- AWS Glue: ETL service, not a general cron runner
- Glue is correct for ETL; Lambda is correct for lightweight scheduled tasks

### Scenario: Single-tenant hardware for compliance (cost-effective)
→ **Dedicated Instances** (hardware isolated per account, less expensive than Dedicated Hosts)
- Dedicated Hosts: full physical server control — needed for BYOL licensing, more expensive
- On-Demand / Spot: share hardware with other customers

### Scenario: Blue-green deployment with DNS caching on mobile clients
→ **AWS Global Accelerator** (traffic shifts take effect in seconds via anycast, bypasses DNS cache)
- Route 53 weighted routing: subject to DNS TTL caching — not reliable for fast cutover

### Scenario: EC2 Auto Scaling — unhealthy instance behavior
→ ASG **terminates first, then launches** a replacement (for health check failures)
- Rebalancing (AZ imbalance): ASG **launches first, then terminates** to protect availability

### Scenario: ASG won't terminate an unhealthy instance — why?
Possible reasons:
1. **Health check grace period** has not expired
2. Instance is in **Impaired** status (ASG waits a few minutes)
3. ASG health check type is **EC2** but instance failed **ELB** health check (mismatch — must set health check type to ELB)


---

## Containers

### Scenario: Run containers without managing EC2 worker nodes, no Kubernetes needed
→ **ECS with Fargate** (fully serverless container runtime)
- ECS on EC2: you manage EC2 capacity (good for cost when high steady utilization)
- EKS: Kubernetes API — only choose if Kubernetes compatibility is explicitly required

### Scenario: Containers at high steady utilization, host-level agent required
→ **ECS on EC2 with Reserved Instances / Savings Plans** (cheaper than Fargate for steady load)

### Scenario: "Least operational overhead" + containers + no Kubernetes
→ Always pick **ECS on Fargate**

---

## Storage — S3

### Scenario: S3 hitting 5000 req/s limit from one prefix
→ Use **multiple custom prefixes** per customer/dataset (each prefix: 3,500 write + 5,500 read req/s)
- Do NOT create one bucket per customer — not scalable and bucket names are globally unique
- EFS is costlier and not the right replacement

### Scenario: S3 upload performance for large files from global users
→ **S3 Transfer Acceleration** (uses CloudFront edge → AWS backbone → S3)
- S3TA: best for large objects (>1 GB) from distant locations
- CloudFront: best for *downloads* of static content (caching)
- Global Accelerator: TCP/UDP apps, not S3 acceleration
- If S3TA doesn't accelerate, **you pay nothing** for S3TA charges (and no S3 transfer-in charge)

### Scenario: S3 upload — very large file, resilience to network errors
→ **Multipart Upload** (parallel parts, restart only failed parts)

### Scenario: Objects must be immutable for compliance (WORM)
→ **S3 Object Lock** (Governance or Compliance mode + optional Legal Hold)
- Retention period on explicit object version → specify Retain Until Date
- Bucket default setting → specify duration (days/years), NOT a Retain Until Date
- Different versions of one object can have different retention modes/periods
- Explicit object-version settings **override** bucket defaults (not the other way around)

### Scenario: Protect against accidental deletion (two correct answers type)
→ **S3 Versioning** + **MFA Delete**
- SNS triggers fire after deletion (reactive, not preventive)

### Scenario: Backfill replication for existing objects (replication rule added late)
→ **S3 Batch Replication** (replicates pre-existing objects)
- Standard cross-region replication only handles new objects going forward

### Scenario: Unknown / changing access patterns, millisecond access always needed
→ **S3 Intelligent-Tiering** (auto-moves objects between tiers, no retrieval fee for frequent/infrequent tiers)
- One Zone-IA: trades availability; only use when data is re-creatable and access is infrequent
- Deep Archive: hours retrieval — cannot guarantee millisecond access

### Scenario: Data accessed for 30 days → rarely 6 months → compliance archive
→ **S3 Lifecycle Policy** to transition: Standard → Standard-IA → Glacier → Deep Archive

### Scenario: Rarely accessed archive, can wait several hours for retrieval
→ **S3 Glacier Deep Archive** (lowest cost, 12-hour retrieval SLA)
- Glacier Instant Retrieval: archive data but needs millisecond access
- Glacier Flexible Retrieval: minutes to hours

### Scenario: Reduce high S3 egress costs for a static website with global users
→ **CloudFront in front of S3** (no S3→CloudFront transfer fee; only pay CloudFront→internet)

### Scenario: S3 object owned by a different account (e.g. Redshift UNLOAD)
→ By default, **object is owned by the uploading account**, not the bucket owner
- Fix: use cross-account IAM roles so the bucket-owner account runs the UNLOAD


---

## Storage — Block & File

### Scenario: Shared POSIX file storage for multiple Linux EC2 across AZs
→ **Amazon EFS** (NFS protocol, multi-AZ by default)
- EFS General Purpose: lower latency for most workloads
- EFS Max I/O: higher aggregate throughput + more parallel clients, but higher metadata latency
- EFS Provisioned Throughput: when throughput needs exceed the Bursting baseline
- EFS Bursting Throughput: default; uses credit system — fine for spiky workloads

### Scenario: Shared SMB file storage for Windows workloads / AD integration
→ **FSx for Windows File Server** (SMB, NTFS, AD integration, Windows ACLs)
- EFS: NFS only, not native Windows
- FSx for Windows also supports Linux clients via SMB (cifs-utils)

### Scenario: Both Windows (SMB/NTFS/AD) and Linux clients need same share
→ **FSx for Windows File Server** (Linux can mount via SMB using cifs-utils)
- EFS does NOT support SMB or Windows ACLs natively

### Scenario: High-performance computing file workload linked to S3 datasets
→ **FSx for Lustre** (Lustre protocol, integrates with S3 as data repository)

### Scenario: High random I/O, data replicated across a fleet, can tolerate instance loss
→ **EC2 Instance Store** (physically attached, highest IOPS per dollar, ephemeral)
- If data must survive instance stop/terminate: use EBS
- Instance store cost is included in the instance usage price

### Scenario: Move from gp2 to reduce cost + independently tune IOPS/throughput
→ **EBS gp3** (separates size from IOPS/throughput settings; cheaper than gp2 at baseline)
- io2/io2 Block Express: for very high IOPS requirements (>16k IOPS)
- st1: throughput-optimized HDD — large sequential workloads
- sc1: cold HDD — lowest cost for infrequent access

### Scenario: Same EBS volume attached to two instances in the same AZ (clustered DB)
→ **EBS Multi-Attach** (io1/io2 volumes only, same AZ, up to 16 instances)

### Scenario: Restore EBS volume in a different AZ after failure
→ **EBS Snapshot → Restore new volume in target AZ** (snapshots are regional, not AZ-scoped)

### Scenario: On-premises NFS/SMB app stores data durably in S3
→ **Storage Gateway — File Gateway** (NFS or SMB front-end backed by S3)

### Scenario: On-premises block storage (iSCSI) with S3 as primary + local cache (most recent data)
→ **Storage Gateway — Volume Gateway Cached Mode** (full data in S3, recently accessed data cached locally)
- Stored Mode: full data stored locally, async copy to S3 — good when low-latency for ALL data needed

### Scenario: On-premises tape backup modernization
→ **Storage Gateway — Tape Gateway** (virtual tape library backed by S3/Glacier)

### Scenario: On-premises app, most frequently accessed logs cached locally, full logs in S3
→ **Volume Gateway Cached Mode** (not Stored Mode — Stored keeps everything local)


---

## Databases — Relational

### Scenario: Relational DB — automatic failover, standby does NOT serve reads
→ **RDS Multi-AZ** (synchronous replication, standby is passive, automatic failover)
- Read Replica: asynchronous, serves reads, does NOT auto-failover
- Key: Multi-AZ = synchronous; Read Replica = asynchronous

### Scenario: Reduce read load on production RDS without touching Multi-AZ standby
→ **RDS Read Replica** — point reporting/analytics queries at the replica endpoint
- You CANNOT read from Multi-AZ standby — it exists only for failover
- For Aurora: use the **reader endpoint** which load-balances across Aurora Replicas

### Scenario: Aurora — always route writes to current writer after failover
→ **Aurora Cluster Endpoint** (always points to current writer)
- Reader endpoint: load-balances reads across replicas
- Instance endpoint: ties to a specific instance (avoid for writes — won't follow failover)

### Scenario: Global relational workload — low-latency reads in another Region + DR
→ **Aurora Global Database** (< 1s replication lag, RPO < 1 min, RTO < 1 min for promoted secondary)

### Scenario: Same game table needs global access; other tables regional only
→ **Aurora Global Database** for the global table + **standard Aurora** for regional tables (keeps same SQL engine = minimal refactoring)
- Mixing Aurora + DynamoDB requires application refactoring

### Scenario: Dev database copy without impacting production DB (no mysqldump load)
→ **Aurora: restore from automated backups** (continuous, incremental, no performance impact)
- mysqldump still generates load on primary
- Multi-AZ standby: cannot be used as dev DB

### Scenario: RDS engine upgrade on Multi-AZ — downtime?
→ **Yes, downtime** — both primary and standby upgrade simultaneously (not rolling)

### Scenario: RDS storage running out unexpectedly
→ **Enable RDS Storage Auto-Scaling** (automatically scales when free space < 10% for > 5 minutes)
- Easiest fix — no migration needed

### Scenario: Migrate SQL Server to Aurora PostgreSQL with minimal app refactoring
→ **Babelfish for Aurora PostgreSQL** (understands T-SQL + SQL Server wire protocol) + **AWS SCT + DMS** for schema/data migration
- Babelfish handles the T-SQL compatibility; SCT converts schema; DMS migrates data

### Scenario: Migrate heterogeneous databases (e.g. Oracle → Aurora, SQL Server → MySQL)
→ **AWS SCT** (schema conversion) + **AWS DMS** (data migration)
- Basic Schema Copy: does not handle stored procedures, secondary indexes, foreign keys
- Snowball Edge: physical data transfer, not DB migration
- Glue: batch ETL, not DB migration

### Scenario: Oracle/SQL Server — custom OS/DB patches required + HA
→ **RDS Custom** (gives OS + DB access) with **Multi-AZ**
- Standard RDS: no OS access
- EC2: max control but maximum management overhead

### Scenario: RDS access control for EFS-like pattern (two correct answers)
→ For EFS: **VPC Security Groups** (network-level) + **IAM Policy** (mount permissions)

### Scenario: Financial services SQL Server migration — minimize management + security
→ **RDS for SQL Server Multi-AZ + KMS encryption at rest**
- EC2-based SQL Server: full control but high operational overhead
- Timestream: time-series data only, not general relational


---

## Databases — NoSQL & Caching

### Scenario: DynamoDB — active-active writes in multiple Regions
→ **DynamoDB Global Tables** (multi-region active-active, managed conflict resolution)
- RDS Multi-AZ: single-region, passive standby
- S3 CRR: object storage, not a database

### Scenario: DynamoDB — unpredictable launch traffic, avoid overprovisioning
→ **DynamoDB On-Demand Capacity** (pay per request, no preplanning)
- Provisioned: cheaper when traffic is predictable; can autoscale but may throttle on sudden spikes

### Scenario: DynamoDB throttling despite sufficient table-level capacity
→ **Hot Partition Key** — one partition receives too many writes
- Fix: randomize or add suffixes to partition key; use write sharding

### Scenario: DynamoDB TTL
→ Automatically deletes expired items — reduces storage cost, no extra charge per deletion

### Scenario: Session store — sub-millisecond, automatic failover, rich data structures (sorted sets, pub/sub)
→ **ElastiCache for Redis** with replication group + Multi-AZ
- Memcached: simpler, horizontal scaling, no persistence or failover — for pure cache with no HA
- DAX: DynamoDB Accelerator only — not for general session stores or Redis replacement

### Scenario: DAX use case
→ **DynamoDB read acceleration** — microsecond latency for DynamoDB reads. Does NOT work for RDS or other databases.

### Scenario: Cache to reduce load on RDS while keeping DB as source of truth
→ **ElastiCache** (Redis or Memcached in front of RDS)
- Read replicas also help but add DB engine cost; ElastiCache absorbs repeated reads more cheaply


---

## Messaging & Event-Driven

### Scenario: Decouple producers from consumers, buffer spikes, allow retry
→ **Amazon SQS**
- Standard: at-least-once, best-effort order, unlimited throughput
- FIFO: exactly-once, strict order, 300 msg/s without batching / **3,000 msg/s with batching** (max 10 per batch)
- FIFO queue name **must end in `.fifo`**
- You **cannot convert** a Standard queue to FIFO — must delete and recreate

### Scenario: Need ordered message processing at 1,000 msg/s
→ **SQS FIFO with batching of 4 messages per operation** (4 × 300 = 1,200 msg/s ≥ 1,000 req)
- Batch of 2: only 600 msg/s — insufficient
- No batching: 300 msg/s — insufficient

### Scenario: Reduce SQS costs (minimize empty receives)
→ **SQS Long Polling** (waits up to 20s for a message before returning empty)
- Short polling: returns immediately even when empty — more API calls = higher cost

### Scenario: One event must fan out to multiple independent consumers with their own retry
→ **SNS topic → multiple SQS queues** (fan-out pattern)
- Single shared SQS queue: competing consumers (only one consumer gets each message)
- SNS alone: push delivery, no durable retry queue for each consumer

### Scenario: SaaS events, rule-based routing, optional event replay
→ **Amazon EventBridge** (event bus with rules, schema registry, archive + replay)
- SNS: simpler pub/sub, no event bus routing or replay
- Replay requires **EventBridge Archive** feature

### Scenario: Near-real-time streaming, multiple consumers, replay within retention window
→ **Amazon Kinesis Data Streams** (multiple consumers, ordering per shard, configurable retention)
- SQS: work queue, not replayable, not multi-consumer streaming
- SNS: push notification, not streaming with replay
- Kinesis Firehose: delivery to data stores (S3, Redshift, OpenSearch) — no custom consumer processing

### Scenario: Real-time analytics on streaming data + REST API front-end
→ **API Gateway + Kinesis Data Analytics** (Kinesis Analytics does stream processing with Apache Flink)

### Scenario: Lambda concurrency throttled during SNS spike
→ Default Lambda concurrency limit is **1,000 per account per region** — raise via AWS Support


---

## High Availability & Auto Scaling

### Scenario: Web tier must survive one AZ failure, stateless, relational DB
→ **ALB + Auto Scaling Group across multiple AZs + RDS Multi-AZ**
- ALB: Layer 7, content-based routing, health checks
- NLB: Layer 4, ultra-high performance TCP/UDP, static IPs
- CLB: legacy, avoid

### Scenario: Choose the right load balancer
| Need | Load Balancer |
|------|--------------|
| HTTP/HTTPS host/path routing, microservices | ALB |
| TCP/UDP, static IPs, ultra-high performance | NLB |
| Third-party security appliances (NGFW, IDS) | GWLB |
| Legacy HTTP/HTTPS + TCP | CLB (avoid) |

### Scenario: Multi-AZ Auto Scaling — AZ rebalancing vs unhealthy replacement
- **Rebalancing** (AZ imbalance): **launches first, terminates second** (preserves availability)
- **Unhealthy replacement** (health check failure): **terminates first, launches second**

### Scenario: Instance must drain jobs before ASG terminates it
→ **ASG Lifecycle Hook** (pauses the termination, runs your script, then continues)
- Scheduled scaling: time-based capacity changes
- Warm pool: pre-initialized instances to speed launch (not termination draining)

### Scenario: Scale based on SQS backlog (not CPU/memory)
→ **Target Tracking scaling policy** on a custom CloudWatch metric (backlog per instance)
- Simple scaling: waits for cooldown after each action — too slow for sudden spikes
- Step scaling: approximates correct count; less precise than target tracking
- Scheduled: time-based only, can't react to real-time demand

### Scenario: DNS-based active-passive failover (primary → secondary on health failure)
→ **Route 53 Failover Routing** (primary + secondary health-checked records)

### Scenario: Lightweight DNS multi-answer with health-awareness (no full LB needed)
→ **Route 53 Multivalue Answer Routing** (returns multiple healthy IPs)
- NOT a substitute for ELB; limited to DNS-level distribution

### Scenario: Route traffic to lowest-latency endpoint for global users
→ **Route 53 Latency-Based Routing**

### Scenario: Route users to different resources by country / location
→ **Route 53 Geolocation Routing**

### Scenario: Split traffic by percentage (A/B test, canary)
→ **Route 53 Weighted Routing**

### Scenario: Zone apex (root domain e.g. example.com) pointing to another record or AWS resource
→ **Route 53 Alias Record** (free for AWS resource queries; supports zone apex)
- CNAME: cannot be used at zone apex; charged per query
- MX: mail server records only; NS: name server records only

### Scenario: Centralized backup policy for EBS, RDS, EFS, DynamoDB across accounts
→ **AWS Backup**


---

## Disaster Recovery

### DR Strategy Decision Tree (cost ↑ = RTO/RPO ↓):

```
Can tolerate hours downtime + rebuild from scratch?  → Backup & Restore    (lowest cost, highest RTO)
Need infrastructure ready but scaled down?           → Warm Standby        (full stack, smaller size, scales up on DR)
Core services only pre-running?                     → Pilot Light          (only DB + core, rest spun up on DR)
Zero / near-zero downtime required?                 → Active-Active Multi-Site (highest cost, lowest RTO/RPO)
```

### Scenario: "Scaled-down full stack running in another Region" — what DR strategy?
→ **Warm Standby** (not pilot light — pilot light runs only core components)

### Scenario: Backups exist but no IaC / runbook — biggest risk?
→ **Infrastructure rebuild may be slow and manual** (data is safe but RTO will be long)

### Scenario: Global NoSQL app — active-active writes in two Regions, low RTO
→ **DynamoDB Global Tables**

### Scenario: Aurora cross-region DR + global low-latency reads
→ **Aurora Global Database**

### Scenario: EBS cross-region DR
→ Copy EBS snapshot to target region → restore volume there


---

## Performance & Global Delivery

### Scenario: Global HTTPS caching for static assets, keep S3 origin private
→ **CloudFront + Origin Access Control (OAC)** (replaces legacy OAI)
- OAC restricts S3 to only accept requests from the CloudFront distribution

### Scenario: CloudFront with on-premises / EC2 origin (dynamic website)
→ **CloudFront with Custom Origin** (HTTP server, ALB, or on-premises server)
- Route 53 DNS record ≠ a valid CloudFront custom origin (distractor)
- S3 only for static content (no server-side scripting like PHP/JSP)

### Scenario: Rewrite/redirect URLs at the edge with minimal latency, very high scale
→ **CloudFront Functions** (lightweight JS at viewer request/response, sub-ms latency)
- Lambda@Edge: heavier logic, runs at regional edge caches, higher latency than CF Functions
- Both are triggered by CloudFront events, not standard Lambda invocations

### Scenario: Improve global performance for TCP/UDP apps, static anycast IPs, fast failover
→ **AWS Global Accelerator** (anycast IPs → AWS backbone → regional endpoints)
- CloudFront: HTTP/HTTPS caching CDN — not for raw TCP/UDP
- Global Accelerator: good for gaming (UDP), IoT (MQTT), VoIP, HTTP apps needing static IPs
- Both integrate with AWS Shield for DDoS

### Scenario: DNS-cached clients (mobile) + blue-green deployment + fast traffic shift
→ **AWS Global Accelerator** (traffic dials + endpoint weights take effect in seconds, bypasses DNS cache)
- Route 53 weighted routing: subject to TTL DNS caching — cannot guarantee fast shift

### Scenario: Managed public API endpoint — throttling, auth, Lambda integration
→ **Amazon API Gateway**
- HTTP API: lower cost + latency, supports JWT auth + Lambda proxy (most common serverless pattern)
- REST API: more features (request transformation, usage plans, caching, VTL mapping)
- WebSocket API: bidirectional persistent connections

### Scenario: Large objects (>1 GB) uploaded globally to S3
→ **S3 Transfer Acceleration** (not CloudFront — CloudFront is for downloads)
- CloudFront: good for distributing content down to users
- S3TA: good for uploads and downloads of large files from distant locations

### Scenario: Streaming data with SQL analytics in real time
→ **API Gateway → Kinesis Data Streams → Kinesis Data Analytics (Apache Flink)**

### Scenario: Managed public REST API for serverless backend
→ **API Gateway** (not ALB alone, not Direct Connect, not NAT)


---

## Analytics & Big Data

### Scenario: Ad-hoc SQL queries over S3 data, no cluster to manage
→ **Amazon Athena** (serverless SQL, pay per TB scanned)
- Reduce cost/latency: convert to **Parquet + partition data** (columnar + partition pruning = less data scanned)

### Scenario: Data warehouse — columnar storage, MPP, BI queries at scale
→ **Amazon Redshift** (not RDS — RDS is OLTP)

### Scenario: Full-text search + log analytics dashboards
→ **Amazon OpenSearch Service** (not Athena — Athena can query S3 but is not a search cluster)

### Scenario: Big data — managed Hadoop / Spark cluster
→ **Amazon EMR**

### Scenario: Managed ETL + Data Catalog for data lakes
→ **AWS Glue** (ETL jobs + Glue Data Catalog for Athena/Redshift Spectrum schema discovery)
- Glue is ETL, not a database migration tool (use DMS for that)

### Scenario: NLP / entity extraction from text files, no ML expertise
→ **Amazon Comprehend** (custom entity recognition) triggered via S3 Event → Lambda
- SageMaker: requires ML expertise — overkill
- Lookout for Vision: image anomaly detection, not text
- Transcribe: speech to text, not document NLP

### Scenario: Detailed billing dataset for custom Athena analysis
→ **Cost and Usage Report (CUR)** delivered to S3
- Cost Explorer: interactive UI, less granular
- CloudWatch Logs Insights: application logs, not billing

---

## Cost Optimization

### Scenario: Steady workload, want flexibility across instance families/regions/OS
→ **Compute Savings Plans** (most flexible — covers EC2, Lambda, Fargate)
- Standard RIs: committed to specific instance family/region/OS — cheapest but rigid
- Convertible RIs: can change attributes, less discount than Standard RI

### Scenario: Batch / interruptible workload, can checkpoint progress
→ **Spot Instances** (biggest discount, ~70-90% off On-Demand)
- Spot Fleet: manages a fleet across pools to maintain target capacity at lowestPrice
- Do NOT use Spot for databases or anything that cannot tolerate interruption

### Scenario: Cost for private subnet → S3 traffic through NAT Gateway
→ **S3 Gateway Endpoint** (free, eliminates NAT Gateway processing charge for S3 traffic)

### Scenario: Cost for private subnet → DynamoDB traffic through NAT Gateway
→ **DynamoDB Gateway Endpoint** (free, same principle as S3)

### Scenario: Monitoring costs + forecasted spend alerts
→ **AWS Budgets** (alerts on actual and forecasted cost/usage thresholds)
- Cost Explorer: visualization and analysis, not alerting on forecasts

### Scenario: Rightsizing recommendations for EC2, ASGs, EBS, Lambda
→ **AWS Compute Optimizer** (ML-based rightsizing)

### Scenario: Allocate monthly spend to applications / teams
→ **Cost Allocation Tags** (activate tags in Billing Console → appear in CUR and Cost Explorer)

### Scenario: Unencrypted EBS volumes costing money after instance termination
→ Review and delete **unattached EBS volumes** (they charge until deleted)

### Scenario: Container workload — Fargate vs EC2 cost decision
- Fargate: variable load, no host management → usually cheaper for bursty
- ECS/EKS on EC2 with Savings Plans: steady high utilization + host-level agents → usually cheaper

### Scenario: EBS type migration for cost savings
→ Migrate from gp2 → **gp3** (same or better performance, lower cost per GB)

### Scenario: Guarantee EC2 capacity in a specific AZ for a one-time event
→ **On-Demand Capacity Reservation** (Savings Plans discount usage but don't reserve capacity)

### Scenario: S3 high egress cost for static content + repeated downloads
→ **CloudFront** (caches at edge, reduces origin fetches; no data transfer fee from S3 to CloudFront)

### Scenario: NAT Gateway HA per AZ vs single NAT Gateway — cost tradeoff
- Single NAT GW: lower hourly cost, but AZ dependency + cross-AZ transfer charges
- NAT per AZ: higher hourly cost, AZ resilience, eliminates cross-AZ data transfer charges
- Choose per-AZ when resilience or cross-AZ transfer volume justifies the extra cost


---

## Hybrid & Migration

### Scenario: SFTP uploads from legacy vendors directly to S3, fully managed, federated identity
→ **AWS Transfer Family with SFTP** + IAM roles per vendor + optionally Cognito/custom identity provider
- EC2-based OpenSSH server: self-managed, high overhead
- AppFlow: API-based SaaS integrations (Salesforce, Zendesk), not SFTP

### Scenario: Migrate large amounts of data to AWS quickly (petabytes, limited bandwidth)
→ **AWS Snowball Edge Storage Optimized** (physical device, petabyte-scale offline transfer)
- Snowball: cannot be used for ongoing sync or low-latency access

### Scenario: IoT / sensor data overwhelms EC2 workers, data replay needed
→ **SQS queue** to buffer (decouples sensors from workers, at-least-once delivery)
- Kinesis: if replay + multiple consumers + streaming analytics needed
- SNS: push only, no buffering or pull-based consumption

### Scenario: Fully serverless sensor/IoT pipeline
→ **SQS Standard → Lambda (event source mapping) → DynamoDB**
- Kinesis Data Firehose cannot write to DynamoDB directly
- EC2-based consumers: not serverless

---

## Monitoring, Audit & Compliance

### The Big Four — Know When to Use Which:

| Question Asks For | Service |
|-------------------|---------|
| Resource performance, metrics, logs, alarms | **CloudWatch** |
| Who called what API, caller identity, audit trail | **CloudTrail** |
| Resource configuration history, compliance rules, remediation | **AWS Config** |
| Threat detection (unusual IPs, compromised creds, suspicious calls) | **GuardDuty** |

### Scenario: ACM certificate expiry notification for *imported* third-party certs
→ **AWS Config managed rule** (e.g. `acm-certificate-expiration-check`) → SNS notification
- ACM auto-renews certs *it* created; imported certs need external monitoring
- CloudWatch alarm on days-to-expiry metric: also works but more configuration than Config managed rule

### Scenario: Continuous vulnerability scanning for EC2, ECR images, Lambda
→ **Amazon Inspector** (automated vulnerability management for those workloads)

### Scenario: Discover sensitive data (PII, PCI) in S3
→ **Amazon Macie** (managed data identifiers, S3 discovery jobs)

### Scenario: Aggregate findings from GuardDuty, Inspector, Macie + compliance standards
→ **AWS Security Hub** (centralizes and normalizes across services + compliance frameworks)

### Scenario: Investigate relationships around a GuardDuty finding (related IPs, users, resources)
→ **Amazon Detective** (behavior graphs from log data for security investigations)

### Scenario: AWS compliance reports (SOC, ISO, PCI)
→ **AWS Artifact** (download compliance documents and agreements)

### Scenario: S3 object-level activity logging (GetObject, PutObject)
→ **CloudTrail Data Events** (not enabled by default — must be explicitly turned on)
- Management events only: control-plane actions (CreateBucket, DeleteBucket)
- Flow Logs: network metadata, not API caller identity

### Scenario: Detect compromised credentials / unusual API call patterns
→ **Amazon GuardDuty** (analyzes CloudTrail, VPC Flow Logs, DNS logs)
- Inspector: software vulnerabilities, not behavioral anomalies
- Config: resource compliance state, not threat detection


---

## Quick-Fire Service Selector

Use this as a final check when stuck between two options.

### "Least operational overhead" signals:
- Containers → **ECS Fargate** (not EC2-based)
- Scheduled job → **EventBridge + Lambda** (not EC2 cron)
- File transfer → **Transfer Family** (not EC2 + OpenSSH)
- SQL analytics on S3 → **Athena** (not Redshift unless warehousing)
- Certificate expiry → **AWS Config managed rule** (not custom CloudWatch alarm)
- Key rotation → **KMS automatic rotation** (not manual)

### "Most cost-effective" signals:
- Interruptible batch → **Spot Fleet**
- Steady workload → **Compute Savings Plans or RIs**
- S3 to private subnet → **S3 Gateway Endpoint** (free, removes NAT charges)
- Archive, hours retrieval → **S3 Glacier Deep Archive**
- Variable unknown patterns → **S3 Intelligent-Tiering**
- Large files globally → **S3 Transfer Acceleration** (pay only if accelerated)
- Static content CDN → **CloudFront** (no S3→CF transfer fee)

### "Highly available / survive AZ failure" signals:
- Compute → Multi-AZ ASG + ALB
- Relational DB → RDS Multi-AZ
- Cache → ElastiCache Redis replication group + Multi-AZ
- NoSQL → DynamoDB (already multi-AZ by default) or Global Tables for multi-Region
- DNS → Route 53 Failover routing with health checks

### "Global users / multi-Region" signals:
- Static content → **CloudFront**
- TCP/UDP apps → **Global Accelerator**
- Relational DB → **Aurora Global Database**
- NoSQL → **DynamoDB Global Tables**
- Routing → **Route 53 Geolocation / Latency-Based**

### Common Traps to Avoid:
| Trap | Correct Answer |
|------|----------------|
| "Store RDS creds securely" → SSM Parameter Store | → **Secrets Manager** (built-in rotation) |
| "Encrypt existing RDS" → enable via console | → **Snapshot → encrypt copy → restore** |
| "CloudFront cert in any region" | → Must be **ACM in us-east-1** |
| "Multi-AZ standby for reads" | → **Standby is passive** — use Read Replica for reads |
| "CNAME for zone apex" | → **Alias record** (CNAME not allowed at zone apex) |
| "NAT Gateway for IPv6 egress" | → **Egress-Only Internet Gateway** |
| "Spot for primary database" | → **Never** — Spot is interruptible |
| "Config detects AND prevents" | → Config detects; SCPs/IAM **prevent** |
| "GuardDuty for vulnerabilities" | → Inspector for vulnerabilities; GuardDuty for threats |
| "SNS for replayable stream" | → SNS pushes; **Kinesis** supports replay |
| "CloudFront for UDP/gaming" | → **Global Accelerator** for non-HTTP |
| "Standard RI flexibility" | → **Compute Savings Plans** for flexibility |
| "NLB for content routing" | → **ALB** for Layer 7 content-based routing |

---

#aws #saa-c03 #cheatsheet #scenarios

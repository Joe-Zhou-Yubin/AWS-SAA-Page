# Domain 3 Exam Map - Design High-Performing Architectures

Weight: 24% of scored content.

Domain 3 is about matching the workload to the right compute, storage, database, network, and data-ingestion shape. Performance questions often hide in words like latency, throughput, concurrency, scale, hot partition, global users, streaming, ad hoc analytics, and least operational overhead.

## Official Task Alignment

| Task | What AWS expects you to design |
|------|--------------------------------|
| 3.1 Storage | Object/file/block choice, hybrid storage, scalable storage, performance configuration |
| 3.2 Compute | EC2, Lambda, Fargate, containers, Batch, EMR, scaling metrics, elastic design |
| 3.3 Databases | Engine/type selection, capacity planning, read replicas, caching, proxies |
| 3.4 Networking | CloudFront, Global Accelerator, VPN, Direct Connect, PrivateLink, load balancer choice, topology |
| 3.5 Ingestion and transformation | Kinesis, Glue, Athena, EMR, data lakes, streaming, file transfer, format conversion |

## Fast Decision Rules

### Compute
- Use EC2 when you need OS access, custom agents, long-running servers, specialized instance types, or lift-and-shift.
- Use Lambda for event-driven code, short tasks, bursty workloads, and low operational overhead.
- Use Fargate when containers are required but you do not want to manage EC2 capacity.
- Use ECS when AWS-native container orchestration is enough.
- Use EKS when Kubernetes APIs, ecosystem tooling, or portability are required.
- Use AWS Batch for batch jobs with managed scheduling across compute environments.

### Storage
- Use S3 for object storage, static assets, data lakes, massive scale, and decoupled ingestion.
- Use EBS for low-latency block storage attached to one EC2 instance in an AZ.
- Use EFS for shared POSIX file storage across Linux instances and multiple AZs.
- Use FSx for Windows File Server for SMB/Windows file workloads.
- Use FSx for Lustre for high-performance processing of file data, especially with S3-backed datasets.
- Use Storage Gateway for hybrid on-premises access to AWS-backed storage.

### Database And Cache
- Use DynamoDB for key-value or document access patterns at any scale.
- Use RDS/Aurora for relational joins, transactions, and SQL engines.
- Use Redshift for warehousing and analytics over structured data at scale.
- Use Athena for ad hoc SQL over data in S3 without managing clusters.
- Use ElastiCache for Redis/Memcached-compatible in-memory caching.
- Use DAX specifically for DynamoDB read caching.
- Use RDS Proxy for pooled database connections, especially with Lambda.

### Network And Edge
- Use CloudFront for HTTP/HTTPS caching and content delivery.
- Use Global Accelerator for static anycast IPs and improved TCP/UDP global routing to regional endpoints.
- Use Direct Connect for private, consistent, high-throughput hybrid connectivity.
- Use Site-to-Site VPN for encrypted connectivity over the internet.
- Use Transit Gateway for hub-and-spoke routing across many VPCs and on-premises networks.
- Use VPC peering for simple non-transitive VPC-to-VPC connectivity.

### Data Ingestion
- Use Kinesis Data Streams when multiple consumers need near-real-time stream processing.
- Use Kinesis Data Firehose for managed delivery to S3, Redshift, OpenSearch, Splunk, or HTTP endpoints.
- Use SQS when the pattern is decoupled task queues, not ordered replayable streaming.
- Use Glue for ETL and the Data Catalog.
- Use Athena for serverless querying of S3 data; optimize with partitioning, compression, and columnar formats such as Parquet.

## Common Exam Traps
- More CPU is not always performance architecture; pick the purpose-built service first.
- S3 is not a file system and should not be selected for POSIX mount requirements.
- EBS is AZ-scoped; use snapshots or replication to recover elsewhere.
- Lambda memory affects CPU allocation; increasing memory can improve runtime and sometimes reduce cost.
- DynamoDB performance depends heavily on partition key design; hot partitions are a classic failure mode.
- Redshift is not the answer for simple OLTP queries; RDS/Aurora or DynamoDB usually is.
- CloudFront improves HTTP content delivery; Global Accelerator improves network path performance for TCP/UDP to endpoints.
- Direct Connect is not encrypted by default; add VPN over DX or MACsec where supported if encryption is required.

## Study Order
1. [[EC2]], [[Lambda]], [[Containers (ECS, EKS, Fargate)]]
2. [[CloudFront]]
3. [[Direct Connect]], [[Transit Gateway]]
4. [[Kinesis]], [[Athena]], [[Redshift]], [[OpenSearch]]
5. [[FSx]], [[Storage Gateway]]
6. [[API Gateway]]

## Scenario Prompts
- Millions of static image requests worldwide: CloudFront in front of S3.
- Low-latency game or financial app traffic over TCP/UDP: Global Accelerator.
- Spiky event processing with minimal operations: Lambda plus SQS/EventBridge.
- Real-time clickstream with multiple consumers: Kinesis Data Streams.
- Query logs in S3 occasionally with SQL: Athena, preferably with partitioned Parquet.
- Kubernetes requirement from platform team: EKS; otherwise prefer ECS/Fargate for lower AWS-native overhead.

---

Related: [[00 - Index]] | [[EC2]] | [[CloudFront]] | [[Kinesis]]
#aws #saa-c03 #domain3 #performance

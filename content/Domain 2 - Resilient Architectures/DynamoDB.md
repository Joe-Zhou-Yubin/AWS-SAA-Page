# Amazon DynamoDB

## Overview
Fully managed NoSQL key-value and document database. Serverless, single-digit millisecond performance at any scale.

## Core Concepts
- **Tables** — collection of items
- **Items** — collection of attributes (max 400KB)
- **Primary Key**: Partition Key (PK) or Partition Key + Sort Key (PK+SK)

## Capacity Modes
- **Provisioned** — specify RCU/WCU, auto-scaling available
- **On-Demand** — pay per request, no capacity planning
- Can switch between modes every 24 hours

## Secondary Indexes
- **GSI (Global Secondary Index)** — different PK+SK, eventual consistency, own RCU/WCU
- **LSI (Local Secondary Index)** — same PK, different SK, must be defined at table creation

## Key Features

### DynamoDB Accelerator (DAX)
- In-memory cache for DynamoDB
- Microsecond latency for cached reads
- No application code changes (drop-in replacement)
- 5-minute default TTL

### DynamoDB Streams
- Ordered stream of item-level modifications
- Triggers Lambda functions
- 24-hour data retention
- Use cases: replication, analytics, event-driven architecture

### Global Tables
- Multi-region, multi-active replication
- Requires DynamoDB Streams enabled
- Sub-second replication
- Active-active (read/write in any region)

### TTL (Time to Live)
- Automatically delete expired items
- No extra cost, no RCU/WCU consumed for deletes
- Use case: session data, temporary records

## DynamoDB vs RDS/Aurora

| Feature | DynamoDB | RDS/Aurora |
|---------|----------|-----------|
| Model | NoSQL (key-value) | Relational (SQL) |
| Schema | Flexible | Fixed schema |
| Scaling | Horizontal (auto) | Vertical (manual) |
| Joins | No | Yes |
| Transactions | Yes (limited) | Full ACID |
| Best for | Known access patterns | Complex queries |

## Exam Tips
- DynamoDB for serverless, high-throughput, key-value workloads
- DAX for read-heavy caching (vs ElastiCache for aggregation/computation)
- Global Tables for multi-region active-active
- On-demand for unpredictable traffic
- 400KB item size limit — store large objects in S3 with pointer in DynamoDB
- Use WCU/RCU calculations for provisioned mode questions

## SAA Decision Addendum

### Choose This When
- **On-demand capacity**: traffic is unpredictable, spiky, or new.
- **Provisioned + auto scaling**: traffic is predictable enough to tune capacity.
- **DAX**: read-heavy DynamoDB workload needs microsecond cached reads.
- **Global tables**: active-active multi-Region NoSQL is required.
- **Streams + Lambda**: item changes should trigger downstream processing.
- **TTL**: old items should expire automatically without consuming write capacity.

### Exam Traps
- DynamoDB is great for known access patterns, not ad hoc relational joins.
- A bad partition key can create hot partitions even if total table capacity looks high.
- Strongly consistent reads cost more capacity than eventually consistent reads.
- GSIs are eventually consistent and have their own capacity considerations.
- Scans are flexible but inefficient; prefer Query when the key pattern allows it.

---

Related: [[ElastiCache]] | [[Domain 4 - Cost-Optimized Architectures/Database Cost Optimization]]
#aws #dynamodb #database #nosql #domain2

# Amazon Aurora

## Overview
AWS proprietary relational database engine, compatible with MySQL and PostgreSQL. Cloud-optimized with 5x MySQL and 3x PostgreSQL performance.

## Architecture
- Storage: 6 copies across 3 AZs (auto-healing, self-replicating)
- 10GB increments, up to 128TB auto-scaling
- One writer endpoint + one reader endpoint (load balanced)
- Shared storage volume (no replication lag between writer and readers)

## Key Features
- Up to 15 read replicas (sub-10ms replica lag)
- Continuous backup to S3
- Point-in-time recovery
- Automatic failover (< 30 seconds)
- Backtrack: rewind DB to any point without restore

## Aurora Serverless
- Auto-scales compute capacity based on demand
- Pay per second of usage
- Use cases: infrequent, intermittent, or unpredictable workloads
- v2: scales instantly, no warmup time

## Aurora Global Database
- 1 primary region (read/write)
- Up to 5 secondary regions (read-only, < 1 second replication)
- Promotes secondary region in < 1 minute (DR)
- Cross-region read replicas for low-latency reads

## Aurora Multi-Master
- All nodes can read AND write
- Continuous availability (no failover needed)
- Use case: applications requiring 100% write availability

## Custom Endpoints
- Define subsets of Aurora replicas (e.g., larger instances for analytics)
- Route specific query types to specific instances

## Exam Tips
- Aurora = best choice for relational DB on AWS (performance + availability)
- 6 copies of data across 3 AZs (tolerates 2 AZ failure for reads)
- Aurora Serverless for unpredictable workloads
- Aurora Global for cross-region DR (< 1 second replication)
- More expensive than RDS but more performant and resilient
- No free tier

---

Related: [[RDS]] | [[Domain 4 - Cost-Optimized Architectures/Database Cost Optimization]]
#aws #aurora #database #domain2

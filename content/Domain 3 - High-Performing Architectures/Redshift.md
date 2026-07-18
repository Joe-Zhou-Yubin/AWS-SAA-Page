# Amazon Redshift

## Overview
Fully managed petabyte-scale data warehouse. Optimized for OLAP (Online Analytical Processing) and complex SQL queries on large datasets.

## Key Features
- Columnar storage (optimized for analytics)
- Massively Parallel Processing (MPP)
- SQL interface (PostgreSQL compatible)
- Up to 10x better performance than other warehouses
- Integrates with BI tools (QuickSight, Tableau)

## Architecture
- **Leader Node** — query planning, client connections
- **Compute Nodes** — execute queries, store data
- Node types: RA3 (managed storage), DC2 (dense compute)

## Key Features
- **Redshift Spectrum** — query S3 data directly without loading
- **Redshift Serverless** — auto-scales, pay per query
- **Concurrency Scaling** — auto-adds capacity for burst reads
- **Materialized Views** — pre-computed results for faster queries
- **AQUA** — hardware-accelerated cache

## Data Loading
- COPY command from S3 (most efficient)
- Kinesis Data Firehose
- AWS DMS
- NOT designed for real-time inserts (batch-oriented)

## Backup & DR
- Automated snapshots (every 8 hours or 5 GB, 1-35 day retention)
- Manual snapshots (no retention limit)
- Cross-region snapshot copy
- No Multi-AZ (single-AZ, use snapshots for DR)

## Redshift vs Athena vs RDS

| Feature | Redshift | Athena | RDS |
|---------|----------|--------|-----|
| Type | Data warehouse | Query service | Database |
| Data location | Loaded into Redshift | S3 (in-place) | In database |
| Best for | Complex analytics, large datasets | Ad-hoc queries, small datasets | OLTP, transactions |
| Management | Cluster (or serverless) | Serverless | Managed instance |
| Cost | Running cluster | Per query | Running instance |

## Exam Tips
- Redshift for data warehousing and large-scale analytics
- Not for OLTP (use RDS/Aurora instead)
- Redshift Spectrum for querying S3 without loading
- Single AZ only — use snapshots for DR
- Enhanced VPC Routing forces COPY/UNLOAD through VPC
- Use Redshift Serverless for intermittent analytical workloads

---

Related: [[Athena]] | [[Kinesis]]
#aws #redshift #analytics #data-warehouse #domain3

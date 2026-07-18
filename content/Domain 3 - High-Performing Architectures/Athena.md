# Amazon Athena

## Overview
Serverless interactive query service. Analyze data in S3 using standard SQL. Pay per query (data scanned).

## Key Features
- No infrastructure to manage
- Supports CSV, JSON, Parquet, ORC, Avro
- Uses Presto/Trino engine under the hood
- Integrates with AWS Glue Data Catalog (schema management)
- Results stored in S3

## Performance Optimization
- **Columnar formats** (Parquet, ORC) — scan less data
- **Partitioning** — organize data by key (e.g., year/month/day)
- **Compression** — reduce data scanned
- **Larger files** — avoid many small files (combine into larger)

## Use Cases
- Ad-hoc querying of S3 data lakes
- Log analysis (VPC Flow Logs, CloudTrail, ALB logs)
- Business intelligence with QuickSight
- ETL pipeline queries

## Pricing
- $5 per TB of data scanned
- Use columnar formats to reduce cost (up to 90% savings)
- Cancelled queries charged for data already scanned

## Federated Query
- Query data across sources (RDS, DynamoDB, Redshift, on-premises)
- Uses Lambda-based data source connectors

## Exam Tips
- Athena = serverless SQL on S3
- Cost optimization: use Parquet/ORC + partitioning
- Use Glue for ETL and Data Catalog
- Common pattern: CloudTrail/VPC Logs → S3 → Athena for analysis
- No ongoing costs (pay only when you query)

---

Related: [[Kinesis]] | [[Redshift]] | [[Domain 2 - Resilient Architectures/S3]]
#aws #athena #analytics #serverless #domain3

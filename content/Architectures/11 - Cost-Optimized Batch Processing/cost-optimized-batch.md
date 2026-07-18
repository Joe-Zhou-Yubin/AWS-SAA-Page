# Cost-Optimised Batch Processing Architecture

AWS SAA-C03 Reference Architecture — 11

---

## Architecture Overview

A batch processing pipeline designed to minimise compute costs by using Spot compute, eliminate network charges with Gateway Endpoints, and reduce storage costs with S3 lifecycle policies. Every design decision is explicitly motivated by cost.

---

## Flow

### Trigger
- **S3 event notification** fires when new input data lands in the S3 input bucket
- OR **EventBridge Scheduler** (cron: nightly 2am) triggers the pipeline on a schedule

### Orchestrate
- **Step Functions** orchestrates the batch steps: validate input → launch compute → wait → validate output → notify
- **SQS Queue** buffers work items for worker-based patterns (Lambda or Fargate Spot consumers)
- Both Step Functions and SQS provide built-in retry and dead-letter queue for failed jobs

### Process (Spot Compute)
Four options — choose based on job characteristics:

| Option | Max Duration | Cost | Managed? | Use When |
|--------|-------------|------|----------|---------|
| Lambda | 15 minutes | Pay per invocation | Fully managed | Short transformations, event-driven |
| Fargate Spot | ~hours | ~70% off On-Demand | No EC2 | Containerised, medium jobs |
| EC2 Spot Fleet | Days | 60-90% off On-Demand | EC2 nodes | Long jobs, GPU, custom instance type |
| AWS Batch | Days | Spot + On-Demand mix | Managed queues | Complex job dependencies |
| EMR Spot | Days | 60-80% off On-Demand | Managed Spark | Large-scale Spark/Hadoop |

### Store
- **S3 Output Bucket** — Parquet format, partitioned by date
- **DynamoDB On-Demand** — for aggregated results needing key-value access
- **S3 Lifecycle Policy** — automatically transitions output to cheaper tiers over time
- **S3 + DynamoDB Gateway Endpoints** — eliminate NAT Gateway charges for private subnet access

### Query & Visualise
- **Glue Data Catalog** — crawls S3 output, registers schema and partitions
- **Athena** — ad-hoc SQL directly on S3 output (pay per TB scanned)
- **QuickSight** — business dashboards sourced from Athena

---

## Cost Optimisation Decisions

### Spot Compute — Where the Big Savings Are

Batch jobs are inherently fault-tolerant:
- They process independent chunks (partitioned input)
- They checkpoint progress to S3
- Failed chunks can be retried without affecting other chunks

This makes them ideal for Spot. If AWS reclaims an instance:
- EC2 Spot Fleet automatically launches a replacement from another pool
- Fargate Spot tasks are re-queued and picked up by the next available Fargate capacity
- SageMaker / EMR handle checkpointing and resumption automatically

**Cost impact:** Spot instances save 60-90% vs On-Demand. A job costing $1,000 on On-Demand costs $100-400 on Spot.

### S3 Gateway Endpoints — Eliminate NAT Charges

Without endpoints, batch compute in private subnets reaches S3 and DynamoDB via NAT Gateway:
```
EC2/Fargate → NAT Gateway ($0.045/GB processed) → S3/DynamoDB
```

With Gateway Endpoints:
```
EC2/Fargate → S3/DynamoDB Gateway Endpoint (free) → S3/DynamoDB
```

At high batch volumes (e.g. 10 TB/month to S3):
```
Without endpoint: 10,000 GB × $0.045 = $450/month NAT charges
With endpoint:    $0 in NAT charges
```

**Only covers S3 and DynamoDB.** For SSM Parameter Store, Secrets Manager, ECR — use Interface Endpoints (charged).

### S3 Lifecycle Policy — Automate Storage Tiering

```
Day 0:    S3 Standard       ($0.023/GB)  — actively queried results
Day 30:   Standard-IA       ($0.0125/GB) — occasional re-runs
Day 90:   Glacier Flexible  ($0.004/GB)  — compliance archive
Day 365:  Glacier Deep Archive ($0.001/GB) — long-term retention
```

No manual intervention — lifecycle rules run automatically. At 100 TB of batch output, this progression saves ~$100,000+ over 3 years vs keeping everything in Standard.

### Output Format — Parquet + Partitioning Reduces Athena Cost

Athena charges $5 per TB scanned. Raw CSV/JSON → every query scans the full dataset.

Parquet + partitioning:
- Parquet: columnar format — queries only read needed columns (80-90% less data scanned)
- Partitioning by date: `WHERE date = '2024-01-15'` only scans that day's prefix

Combined savings: a query that would scan 1 TB of JSON typically scans ~50-100 GB of Parquet → 90-95% cost reduction.

### AWS Batch vs DIY Spot

AWS Batch manages:
- Compute environment scaling (adds/removes EC2 based on job queue depth)
- Spot interruption handling and retry
- Job priority queues (higher priority jobs get capacity first)
- Job dependencies (Job B only starts after Job A finishes)

For simple jobs: Lambda or Fargate Spot + SQS is simpler and cheaper.
For complex multi-step jobs with dependencies: AWS Batch saves significant orchestration code.

### Cost Allocation Tags

Tag every batch job with `Project`, `Environment`, `Team`. Activate tags in Billing Console. Use AWS Budgets per tag to alert when a specific batch workload exceeds its budget.

---

## Choosing the Right Spot Option

```
Task takes < 15 minutes?               → Lambda (no Spot needed, cheapest possible)
Containerised, < 2-3 hours?           → Fargate Spot
Need GPU or custom instance family?   → EC2 Spot Fleet
Complex job dependencies?             → AWS Batch (manages Spot + queues)
Spark/Hadoop on large dataset?        → EMR with Spot core nodes
```

---

## Key Cost Comparison (example: 100 EC2 vCPUs for 8 hrs/night)

```
On-Demand (c5.2xlarge = $0.34/hr):
100 vCPUs ÷ 8 vCPU/instance = ~13 instances × $0.34 × 8 hrs = $35/night = $1,050/month

Spot (~70% off = $0.10/hr):
13 instances × $0.10 × 8 hrs = $10/night = $300/month

Savings: $750/month = $9,000/year from compute alone
+ S3 NAT Gateway elimination: ~$200-500/month at scale
+ Lifecycle tiering: significant over months/years
```

---

## Exam Traps

| Trap | Reality |
|------|---------|
| "Spot Instances for primary databases" | Spot = batch/fault-tolerant only. Never for databases |
| "Fargate Spot same as Lambda" | Fargate Spot: containerised tasks, hours duration. Lambda: functions, 15 min max |
| "S3 data transfer to NAT is free" | NAT Gateway charges $0.045/GB processed — use Gateway Endpoint to eliminate |
| "Lifecycle policies need CloudWatch triggers" | S3 lifecycle policies run automatically — no trigger needed |
| "Athena cost is fixed" | Athena charges **per TB scanned** — Parquet + partitioning directly reduces cost |
| "AWS Batch = ECS" | AWS Batch is a managed job scheduler that can use EC2 or Fargate underneath |
| "Standard RI for monthly batch" | Monthly batch workload running once/month → Spot wins; RI is for continuous steady load |

---

Related: [[../../CHEATSHEET]] | [[../../Domain 4 - Cost-Optimized Architectures/00 - Domain 4 Exam Map]] | [[../../Cross-Cutting/Step Functions]]
#aws #architecture #batch #cost-optimization #spot #saa-c03

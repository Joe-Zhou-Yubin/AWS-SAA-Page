# Apache Airflow & Amazon MWAA

## Overview
Apache Airflow is an open-source **workflow orchestration platform** for scheduling and managing data pipelines. Workflows are defined as Python code using **DAGs (Directed Acyclic Graphs)**.

Amazon MWAA (Managed Workflows for Apache Airflow) is the AWS-managed version — AWS handles infrastructure, scaling, patching, and HA. You just write DAGs.

**Airflow vs Step Functions:**
- Airflow → data pipelines, ETL, ML workflows, scheduled batch jobs (data engineering domain)
- Step Functions → service orchestration, business workflows, real-time event-driven (application domain)
- Both can be used together in mature architectures

---

## Core Concepts

### DAG (Directed Acyclic Graph)
A workflow definition written in Python. Consists of Tasks with dependency relationships.
- No cycles allowed (hence "acyclic")
- Each run is a **DAG Run** with a unique execution date
- Supports backfill — re-run historical date ranges automatically

### Tasks and Operators
Each step in a DAG is a **Task**, implemented by an **Operator**:
- `PythonOperator` — run a Python function
- `BashOperator` — run a shell command
- `GlueJobOperator` — trigger an AWS Glue ETL job
- `S3ToRedshiftOperator` — load S3 data into Redshift
- `KubernetesPodOperator` — run a Docker container as a K8s pod
- `SageMakerTrainingOperator` — start a SageMaker training job
- `LambdaInvokeFunctionOperator` — invoke a Lambda function
- `EcsRunTaskOperator` — run an ECS/Fargate task
- `AthenaOperator` — execute an Athena query and wait for result
- `EmrCreateJobFlowOperator` — spin up an EMR Spark cluster

### Scheduling
Cron-based or preset intervals:
```
@daily        → 0 0 * * *
@hourly       → 0 * * * *
0 6 * * MON-FRI → 6am on weekdays
```

---

## Architecture

```
┌──────────────────────────────────────────────────┐
│                  Airflow / MWAA                  │
│                                                  │
│  Scheduler → decides what to run and when        │
│  Executor  → dispatches tasks to workers         │
│  Workers   → execute task code                   │
│  Web UI    → monitor DAGs, logs, trigger runs    │
│  Metadata DB (PostgreSQL) → stores run state     │
└──────────────────────────────────────────────────┘
```

**Executor types:**
- `LocalExecutor` — tasks run on the scheduler node (dev only)
- `CeleryExecutor` — tasks distributed to worker pool via Redis/RabbitMQ
- `KubernetesExecutor` — each task gets its own K8s pod (best isolation, scales to zero)

---

## AWS Best Practice Architecture

```
GitHub/CodeCommit ──(git sync)──→ MWAA / EKS Airflow
                                        │
Secrets Manager ←──(fetch at runtime)──┤
RDS PostgreSQL (metadata DB) ←──────────┤
S3 (task logs + DAG artifacts) ←────────┤
CloudWatch (metrics + alarms) ←─────────┘

Tasks trigger AWS services:
├── Glue ETL jobs
├── EMR Spark clusters
├── ECS/Fargate containers
├── SageMaker training/inference
├── Redshift queries
├── Athena queries
└── Lambda functions
```

---

## Key Integrations (AWS)

| Service | What Airflow Does |
|---------|------------------|
| **S3** | Read/write data, store logs, store DAG files |
| **Glue** | Trigger ETL jobs, wait for completion |
| **EMR** | Create clusters, submit steps, terminate |
| **Redshift** | Execute SQL, load from S3 |
| **Athena** | Run queries, wait for results |
| **SageMaker** | Trigger training, model deployment |
| **ECS/Fargate** | Run containerised task workloads |
| **Lambda** | Invoke functions as pipeline steps |
| **Secrets Manager** | Fetch credentials at runtime (never hardcode) |
| **CloudWatch** | Emit metrics (task duration, failures, queue depth) |
| **SNS/Slack** | Alert on DAG failures |

---

## Best Practices

### 1. DAGs in Git (GitOps)
- Never edit DAGs in the Airflow UI
- All DAGs live in a Git repo with CI/CD (lint, unit test on PR)
- Git sync sidecar pulls latest DAGs automatically
- Rollback = revert the commit

### 2. Secrets via Secrets Manager Backend
```python
# airflow.cfg
secrets_backend = airflow.providers.amazon.aws.secrets.secrets_manager.SecretsManagerBackend
# Connections stored as: airflow/connections/<conn_id>
# Variables stored as: airflow/variables/<var_name>
```
Credentials never stored in the Airflow metadata DB.

### 3. Task Isolation with KubernetesPodOperator
```python
train = KubernetesPodOperator(
    task_id='train_model',
    image='my-ml-image:latest',   # PyTorch, CUDA etc.
    resources={'request_memory': '16Gi', 'limit_gpu': '1'}
)
```
Each task runs in its own pod with exactly the dependencies it needs.

### 4. Idempotent Tasks
Tasks should produce the same result if run multiple times.
- Use `INSERT OVERWRITE` not `INSERT INTO`
- Delete output before writing
- Enables safe retries and backfills

### 5. Metadata DB → RDS PostgreSQL Multi-AZ
- Never use SQLite in production (can't handle concurrent writes)
- Use RDS PostgreSQL Multi-AZ for HA
- db.t3.medium for small deployments, db.r5.large for large

### 6. Logs → S3
```python
remote_logging = True
remote_base_log_folder = s3://my-airflow-logs/
```
Logs persist after worker pods are destroyed. Accessible from UI via presigned URLs.

### 7. Error Handling
```python
default_args = {
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'email_on_failure': True,
    'email': ['data-team@company.com']
}
```
SLA misses and failure callbacks alert via SNS → Slack/PagerDuty.

---

## Amazon MWAA (Managed Airflow)

AWS fully manages the Airflow infrastructure:
- Scheduler, workers, web server all managed by AWS
- Scales workers automatically based on task queue depth
- Integrated with IAM (role-based access to UI and DAGs)
- Runs inside your VPC (private networking to RDS, Redshift, etc.)
- S3 bucket for DAGs and plugins
- CloudWatch for logs and metrics

**Pricing:**
- Environment class charge (small/medium/large): ~$0.49–$1.50/hr
- Worker charge: per worker-hour when scaling
- ~$350–$1,100/month for a small production environment

**When MWAA vs self-hosted:**
- MWAA: less ops overhead, AWS handles upgrades, faster to start
- Self-hosted on EKS: more control, cheaper at scale, better GPU/custom image support

---

## Airflow vs Step Functions — Decision Guide

| Scenario | Use |
|----------|-----|
| Daily ETL pipeline: S3 → Glue → Redshift | Airflow |
| ML training pipeline with multiple jobs | Airflow |
| dbt model runs on a schedule | Airflow |
| Re-run last 30 days of data (backfill) | Airflow |
| Order processing with human approval | Step Functions |
| Microservice orchestration (real-time) | Step Functions |
| High-volume event-driven workflows | Step Functions Express |
| Both data + application orchestration | Both (Airflow for data layer, SF for app layer) |

---

## Exam Tips (SAA-C03)
- MWAA is the AWS exam answer for "managed Apache Airflow"
- Appears in analytics/data pipeline scenarios
- Key distinction: Airflow/MWAA for **scheduled data pipelines**, Step Functions for **real-time service orchestration**
- MWAA deploys inside a VPC — can access private RDS, Redshift, EMR
- DAG files stored in S3, synced to the environment
- Logs go to CloudWatch and/or S3

---

Related: [[Step Functions]] | [[Domain 3 - High-Performing Architectures/Athena]] | [[Domain 3 - High-Performing Architectures/Redshift]] | [[Domain 3 - High-Performing Architectures/Kinesis]]
#aws #airflow #mwaa #orchestration #data-pipeline #cross-cutting

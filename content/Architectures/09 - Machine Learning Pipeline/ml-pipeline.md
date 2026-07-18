# Machine Learning Pipeline on AWS

AWS SAA-C03 Reference Architecture — 09

---

## Architecture Overview

A five-phase ML pipeline from raw data ingestion through to inference serving. Each phase uses purpose-built AWS services. The pipeline is orchestrated by Step Functions or Airflow (MWAA) and monitored for model quality degradation via SageMaker Model Monitor.

---

## Phase-by-Phase Flow

### Phase 1 — Ingest
Data arrives from multiple sources:
- **IoT Sensors** → Kinesis Data Streams (real-time, sub-second, replay capable)
- **Application DB** → Kinesis Firehose (batch export, buffered delivery)
- **Log Files** → Kinesis Firehose (log ingestion)

Both Kinesis paths deliver to **S3 Raw Data Lake** — raw CSV/JSON partitioned by date/hour.

### Phase 2 — Process
1. **AWS Glue ETL** job reads raw S3 data, cleans and transforms it
2. Feature engineering produces Parquet files written to **S3 Processed**
3. For very large datasets: **Amazon EMR** Spark cluster does distributed preprocessing
4. **Glue Data Catalog** stores schema metadata — makes data queryable via Athena

**Step Functions / MWAA** orchestrates the sequence: Glue job → evaluate output → trigger training.

### Phase 3 — Train
1. **SageMaker Training Job** reads features from S3 Processed
2. Runs on GPU instances (p3, p4, g4) using managed containers (PyTorch, TensorFlow, XGBoost)
3. **Spot Instances for training**: SageMaker handles checkpointing — up to 90% cost reduction
4. **Amazon EMR** alternative: for Spark MLlib or when custom frameworks needed
5. Model artifacts saved to **S3 Model Artifacts** as `model.tar.gz`

### Phase 4 — Deploy
1. **SageMaker Model Registry**: model is registered and reviewed (approved/rejected)
2. On approval, model deployed to either:
   - **SageMaker Endpoint** (real-time inference, always-on, auto-scales)
   - **SageMaker Batch Transform** (async bulk inference, S3 in → S3 out, no endpoint needed)
3. **SageMaker Model Monitor** continuously checks endpoint for data drift and quality degradation

### Phase 5 — Serve
1. **API Gateway** provides managed REST endpoint for inference requests
2. **Lambda** handles pre/post processing (feature transformation, response formatting)
3. Lambda calls **SageMaker Endpoint** for the actual prediction
4. Response returned to calling **Applications**

---

## Services Reference

| Service | Phase | Purpose |
|---------|-------|---------|
| Kinesis Data Streams | Ingest | Real-time event ingestion, replay, multi-consumer |
| Kinesis Firehose | Ingest | Buffered delivery to S3, no custom consumer code |
| S3 Raw Data Lake | Ingest/Process | Durable object storage, all raw data |
| AWS Glue ETL | Process | Serverless Spark, feature engineering, data catalog |
| Amazon EMR | Process/Train | Managed Hadoop/Spark for large-scale processing |
| S3 Processed | Process/Train | Parquet features ready for training |
| SageMaker Training Job | Train | Managed ML training on GPU/CPU instances |
| S3 Model Artifacts | Train/Deploy | Stores serialised model files |
| SageMaker Model Registry | Deploy | Model versioning, approval workflow |
| SageMaker Endpoint | Deploy/Serve | Real-time inference, auto-scales |
| SageMaker Batch Transform | Deploy/Serve | Async bulk inference, no always-on cost |
| SageMaker Model Monitor | Monitor | Data drift detection, quality alerts |
| API Gateway | Serve | Managed REST API for inference requests |
| Lambda | Serve | Pre/post processing, feature transformation |
| Step Functions / MWAA | Orchestration | Pipeline coordination |
| CloudWatch | Monitor | Endpoint metrics, alarms, dashboards |

---

## Key Design Decisions

### Spot Instances for Training — Why?
ML training jobs are:
- Long-running (hours to days)
- Can checkpoint progress to S3
- Stateless between checkpoints

This makes them ideal for Spot — up to 90% cost reduction. SageMaker Training Jobs have built-in managed Spot + checkpointing: on interruption, SageMaker resumes from the last checkpoint automatically.

**Do NOT use Spot for inference endpoints** — they must be always-on and cannot tolerate interruption.

### Real-time Endpoint vs Batch Transform

| Factor | SageMaker Endpoint | Batch Transform |
|--------|-------------------|----------------|
| Latency | Milliseconds | Minutes to hours |
| Cost | Ongoing (always-on instance) | Pay per job only |
| Use case | Fraud detection, recommendations, live ranking | Nightly score refresh, bulk classification |
| Input | One request at a time | All inputs from S3 |
| Output | Direct response | Written to S3 |

### Glue vs EMR for Feature Engineering

| Factor | AWS Glue | Amazon EMR |
|--------|----------|-----------|
| Management | Serverless — no cluster sizing | Managed cluster — you choose instance types |
| Scale | Automatic | Manual (or Spot Fleet) |
| Flexibility | Standard Spark + limited custom libraries | Full control — any Spark, ML framework, GPU |
| Cost model | Per DPU-second | Per instance-hour |
| Best for | Standard ETL, moderate data (< 1 TB) | Very large datasets, custom frameworks, GPU preprocessing |

### Pipeline Orchestration Options

**Step Functions** — coordinate Glue job → wait → SageMaker training → conditional model deploy. Good for AWS-native workflows with < 15 step complexity.

**Amazon MWAA (Airflow)** — DAG-based, better for data teams who manage multiple daily pipeline schedules, backfill requirements, complex dependencies. Richer ecosystem of operators.

**SageMaker Pipelines** — purpose-built MLOps pipeline with native Model Registry integration, lineage tracking, and CI/CD for models. Best when the team is fully invested in SageMaker.

---

## Exam Traps

| Trap | Reality |
|------|---------|
| "SageMaker = only for training" | SageMaker covers the full ML lifecycle: data labeling, training, tuning, registry, deployment, monitoring |
| "Spot for inference endpoints" | Spot = training only. Endpoints must be on-demand (always-on) |
| "Glue always replaces EMR" | For very large datasets or custom ML frameworks, EMR is still needed |
| "Batch Transform needs an endpoint" | Batch Transform does NOT use an endpoint — it reads from S3 and writes to S3 |
| "SageMaker Endpoint doesn't auto-scale" | SageMaker Endpoints support Application Auto Scaling based on invocations per instance |
| "Comprehend is part of SageMaker" | Amazon Comprehend (NLP) is a separate managed AI service — no ML expertise needed |

---

Related: [[../../CHEATSHEET]] | [[../../Cross-Cutting/Airflow (MWAA)]] | [[../../Cross-Cutting/Step Functions]]
#aws #architecture #machine-learning #sagemaker #saa-c03

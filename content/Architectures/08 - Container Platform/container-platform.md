# Container Platform Architecture (ECS)

AWS SAA-C03 Reference Architecture — 08

---

## Architecture Overview

Two deployment options for container workloads on ECS, sharing a common data and security layer. The choice between Fargate and EC2 launch type is primarily driven by workload variability and operational preferences.

---

## Request Flow

1. **Clients** send requests to the domain
2. **Route 53** resolves DNS → CloudFront
3. **CloudFront** caches static content at edge, forwards API requests to ALB
4. **ALB** routes to ECS tasks via target groups (path-based routing to different services)

### Option A — ECS on Fargate
5. ALB forwards to **ECS Cluster (Fargate)**
6. ECS schedules **Fargate Tasks** — AWS provisions the compute automatically
7. Tasks pull container images from **ECR** using the Task Execution Role
8. **Fargate Spot** tasks run interruptible workloads (batch jobs, dev) at ~70% discount

### Option B — ECS on EC2
5. ALB forwards to **ECS Cluster (EC2)**
6. ECS schedules tasks onto EC2 instances via the **ECS Agent** running on each node
7. **Auto Scaling Group** manages EC2 node capacity — scales in/out based on cluster capacity provider
8. EC2 instances use Reserved Instances or Savings Plans for cost at steady utilization

### Shared Data Layer
- Tasks connect to **Aurora** via **RDS Proxy** (connection pooling — critical for Fargate which can spawn many tasks)
- **DynamoDB On-Demand** for session data or NoSQL workloads
- **ElastiCache Redis** for caching and session store
- **EFS** for shared file storage mounted across tasks (Fargate 1.4+ supports EFS volumes)
- **S3** for static assets and task artifacts
- **SQS** for async work queues — worker tasks scale based on queue depth

### Security
- **ECR** stores private container images — ECR image scanning (via Amazon Inspector) detects CVEs
- **Secrets Manager** injects DB credentials at task launch via Task Execution Role
- **Task Role** grants the application code inside the container access to AWS services
- **Task Execution Role** grants ECS infrastructure the ability to pull images and write logs
- Container stdout → **CloudWatch Logs** via awslogs driver

---

## Services Reference

| Service | Purpose |
|---------|---------|
| ECR | Private container image registry, image vulnerability scanning |
| ECS | Container orchestration (task scheduling, service management) |
| Fargate | Serverless compute engine for ECS tasks — no EC2 nodes |
| Fargate Spot | Interruptible Fargate capacity at ~70% discount |
| EC2 ASG | Worker node fleet for ECS on EC2 launch type |
| ALB | Layer 7 load balancing to ECS target groups |
| CloudFront | CDN + HTTPS, reduces ALB load for static content |
| Aurora + RDS Proxy | Relational DB with connection pooling for containerised workloads |
| DynamoDB On-Demand | NoSQL for session data, spiky workloads |
| ElastiCache Redis | In-memory cache, session store |
| EFS | Shared NFS file storage mountable by multiple tasks/nodes |
| S3 | Object storage for assets and artifacts |
| SQS | Async work queue — worker containers scale on queue depth |
| Secrets Manager | DB credentials injected at task launch |
| Task Role | IAM role for app code inside the container |
| Task Execution Role | IAM role for ECS/Fargate infrastructure |
| CloudWatch Logs | Container log collection via awslogs driver |

---

## Key Design Decisions

### Fargate vs ECS on EC2 — When to Choose

| Factor | ECS Fargate | ECS on EC2 |
|--------|------------|-----------|
| Node management | None — AWS handles it | You manage EC2 lifecycle |
| Cost at variable load | Better (pay per task-second, scale to zero) | Worse (idle EC2 still costs) |
| Cost at steady high utilisation | Worse (Fargate premium ~20%) | Better (RI/Savings Plans) |
| Host-level access (custom agents, GPU) | Not available | Full OS access |
| Startup time | ~30s (no EC2 to wait for) | EC2 launch + ECS agent registration |
| Exam default answer | "least ops overhead" + containers | "steady utilisation" + "host-level agents" |

### The Two IAM Roles — Most Tested Exam Topic for ECS

**Task Execution Role** — ECS/Fargate infrastructure needs this to:
- Pull the container image from ECR (`ecr:GetAuthorizationToken`, `ecr:BatchGetImage`)
- Write container logs to CloudWatch Logs
- Fetch secrets from Secrets Manager or SSM Parameter Store at container startup

**Task Role** — Your application code needs this to:
- Read/write to S3
- Query DynamoDB
- Send messages to SQS
- Call any other AWS service

**Common mistake:** Attaching only the Task Execution Role and wondering why the app can't call S3. These are two completely separate roles with completely different purposes.

### Why RDS Proxy for containerised workloads?
Fargate tasks scale from 1 to hundreds instantly. Each task opens a DB connection. Aurora has a connection limit. Without RDS Proxy, Aurora is overwhelmed during scale-out events.

RDS Proxy pools and multiplexes connections — hundreds of Fargate tasks share a small pool of actual DB connections. Also speeds up Aurora failover handling.

### EFS with Fargate
Fargate platform version 1.4+ supports EFS volume mounts. Useful when multiple tasks need access to the same files (shared configuration, media processing, ML model weights).

### Container Networking Mode
For both Fargate and ECS on EC2 (awsvpc mode):
- Each task gets its own ENI and private IP
- Security groups can be applied per task
- Full VPC networking features available
- Recommended for production workloads

### Auto Scaling Containers
**Service Auto Scaling** — scales the number of ECS tasks:
- Target tracking on CPU/memory utilisation
- Target tracking on custom metric (e.g. SQS queue depth per task)
- Scheduled scaling for known traffic patterns

**Cluster Auto Scaling (EC2 only)** — scales the EC2 node pool:
- Capacity Provider with Managed Scaling
- Automatically adds/removes EC2 nodes based on task demand

---

## Exam Traps

| Trap | Reality |
|------|---------|
| "Task Role = Task Execution Role" | Two separate roles with different purposes |
| "Fargate is always cheaper" | Fargate wins for variable load; EC2 wins at steady high utilisation |
| "ECS requires Kubernetes" | ECS is AWS's own orchestrator — EKS is for Kubernetes. No Kubernetes needed for ECS |
| "Fargate can't use EFS" | Fargate 1.4+ supports EFS volume mounts |
| "Security groups attach to ECS clusters" | In awsvpc mode, security groups attach to individual **tasks** (ENIs) |
| "ECR is just S3 with a different name" | ECR manages image tags, lifecycle policies, image scanning, pull-through cache |
| "ECS Agent is customer-managed" | ECS Agent runs on EC2 nodes — for Fargate, AWS manages all infrastructure |

---

Related: [[../../CHEATSHEET]] | [[../../Domain 3 - High-Performing Architectures/Containers (ECS, EKS, Fargate)]]
#aws #architecture #containers #ecs #fargate #saa-c03

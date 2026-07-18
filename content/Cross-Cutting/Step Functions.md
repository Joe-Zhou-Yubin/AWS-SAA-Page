# AWS Step Functions

## Overview
Serverless orchestration service for coordinating multiple AWS services into workflows (state machines).

## Workflow Types
- **Standard** — up to 1 year duration, exactly-once execution, audit history
- **Express** — up to 5 minutes, at-least-once, high volume (100,000+/sec)

## State Types
- **Task** — invoke a service (Lambda, ECS, DynamoDB, SQS, etc.)
- **Choice** — branching logic (if/else)
- **Parallel** — execute branches concurrently
- **Wait** — delay execution
- **Map** — iterate over array items (dynamic parallelism)
- **Succeed/Fail** — end execution

## Key Features
- Visual workflow designer
- Built-in error handling (Retry, Catch)
- Service integrations (200+ AWS services)
- Input/output processing (JSONPath)
- Execution history and logging
- Human approval steps (callback pattern)

## Integration Patterns
- **Optimized** — direct SDK integration (DynamoDB, SQS)
- **AWS SDK** — call any AWS API
- **Activity Tasks** — external workers poll for tasks
- **Callback** — wait for external signal (human approval, external system)

## Common Patterns
- Orchestrate Lambda functions (avoid chaining)
- Order processing workflows
- ETL pipelines
- Human approval workflows
- Saga pattern for distributed transactions

## Exam Tips
- Step Functions > chaining Lambda functions (better error handling, visibility)
- Standard for long-running workflows (up to 1 year)
- Express for high-volume, short-duration workloads
- Map state for parallel processing of items
- Use with Lambda for complex serverless workflows
- Error handling built-in (Retry with backoff, Catch for fallback)

---

Related: [[Domain 3 - High-Performing Architectures/Lambda]] | [[Domain 2 - Resilient Architectures/SQS]] | [[Airflow (MWAA)]]
#aws #step-functions #serverless #orchestration #cross-cutting

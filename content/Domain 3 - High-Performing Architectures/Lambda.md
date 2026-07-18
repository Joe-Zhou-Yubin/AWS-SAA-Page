# AWS Lambda

## Overview
Serverless compute service. Run code without provisioning servers. Pay only for compute time consumed.

## Key Limits
- **Memory**: 128 MB - 10 GB (CPU scales proportionally)
- **Timeout**: max 15 minutes
- **Deployment package**: 50 MB zipped, 250 MB unzipped
- **Concurrent executions**: 1,000 (soft limit, can increase)
- **Ephemeral storage (/tmp)**: 512 MB - 10 GB
- **Environment variables**: 4 KB total

## Invocation Types
- **Synchronous** — caller waits for response (API Gateway, ALB)
- **Asynchronous** — fire and forget (S3, SNS, EventBridge)
- **Event Source Mapping** — Lambda polls source (SQS, Kinesis, DynamoDB Streams)

## Key Features
- **Layers** — shared libraries/code across functions
- **Versions & Aliases** — immutable deployments, alias routing (canary)
- **Provisioned Concurrency** — pre-warm instances (eliminate cold starts)
- **Reserved Concurrency** — guarantee capacity for a function
- **SnapStart** — reduce cold start for Java (snapshot after init)
- **Destinations** — route async results to SQS, SNS, Lambda, EventBridge

## Lambda + VPC
- By default runs outside your VPC
- Can attach to VPC (needs ENI in subnet)
- Requires NAT Gateway for internet access from private subnet
- VPC-to-VPC access via ENI

## Event Source Mappings
- Polls from stream/queue (Kinesis, DynamoDB Streams, SQS)
- Batch processing (configurable batch size)
- Error handling: retry, bisect on error, DLQ
- Scaling: Kinesis (1 Lambda per shard), SQS (up to 1000 batches)

## Pricing
- Per request ($0.20 per 1M requests)
- Per duration (GB-second of compute time)
- Free tier: 1M requests + 400,000 GB-seconds per month

## Exam Tips
- 15-minute max timeout (longer tasks → Step Functions, Fargate, Batch)
- Cold starts for VPC-attached functions (mitigate with Provisioned Concurrency)
- Lambda@Edge / CloudFront Functions for edge computing
- RDS Proxy to avoid connection exhaustion
- Use Lambda Layers for shared dependencies
- Event source mapping processes records in batches

## SAA Decision Addendum

### Choose This When
- **Lambda**: event-driven work, short tasks, bursty traffic, and low operational overhead matter.
- **Provisioned concurrency**: cold starts are unacceptable for latency-sensitive functions.
- **Reserved concurrency**: protect downstream services or reserve capacity for a function.
- **Event source mapping**: Lambda should poll SQS, Kinesis, or DynamoDB Streams.
- **Lambda in VPC**: the function needs private subnet resources such as RDS or ElastiCache.

### Exam Traps
- Lambda is not ideal for long-running jobs or workloads requiring full OS control.
- More memory also grants more CPU, so higher memory can improve performance and sometimes reduce total cost.
- A VPC-enabled Lambda still needs NAT or VPC endpoints for outbound access to public AWS services unless using private endpoints.
- Use SQS between producers and Lambda when you need buffering and retry control.

---

Related: [[Cross-Cutting/Step Functions]] | [[Domain 3 - High-Performing Architectures/API Gateway]]
#aws #lambda #serverless #compute #domain3

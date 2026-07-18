# Amazon SQS (Simple Queue Service)

## Overview
Fully managed message queuing service for decoupling application components.

## Queue Types

### Standard Queue
- Unlimited throughput
- At-least-once delivery (possible duplicates)
- Best-effort ordering (not guaranteed)

### FIFO Queue
- Exactly-once processing
- First-in-first-out ordering
- 300 messages/sec (3,000 with batching)
- Message group ID for ordering within groups
- Deduplication ID (5-minute window)

## Key Concepts
- **Visibility Timeout** — time message is hidden after being read (default 30s, max 12h)
- **Message Retention** — 1 minute to 14 days (default 4 days)
- **Max Message Size** — 256 KB (use S3 for larger via Extended Client Library)
- **Long Polling** — reduces empty responses, saves cost (wait up to 20s)
- **Dead Letter Queue (DLQ)** — messages that fail processing after X attempts

## Common Patterns
- **Fan-out**: SNS → multiple SQS queues
- **Decoupling**: Producer → SQS → Consumer (ASG scales on queue depth)
- **Request buffering**: Smooth traffic spikes

## Security
- In-flight encryption (HTTPS)
- At-rest encryption (KMS)
- IAM policies for send/receive
- SQS Access Policies (resource-based, cross-account)

## Exam Tips
- SQS for decoupling and buffering
- Standard for high throughput, FIFO for ordering guarantees
- ApproximateNumberOfMessages → CloudWatch → scale ASG consumers
- Long Polling > Short Polling (cost and efficiency)
- DLQ for debugging failed messages
- Max 256KB — larger payloads use S3 pointer pattern

## SAA Decision Addendum

### Choose This When
- **Standard queue**: maximum throughput and at-least-once delivery are acceptable.
- **FIFO queue**: strict ordering and deduplication by message group are required.
- **DLQ**: failed messages need isolation for later inspection or replay.
- **Long polling**: reduce empty receives and request cost.
- **Queue depth metric**: Auto Scaling workers based on backlog.

### Exam Traps
- Exactly-once processing still requires idempotent consumers in real designs.
- Visibility timeout is not message retention; it only hides an in-flight message.
- SQS is for decoupled work queues, not multi-consumer replayable streaming. Compare with Kinesis.
- A Lambda event source mapping can retry and send failures to a DLQ or destination depending on configuration.

---

Related: [[SNS]] | [[EventBridge]] | [[Cross-Cutting/Step Functions]]
#aws #sqs #messaging #domain2

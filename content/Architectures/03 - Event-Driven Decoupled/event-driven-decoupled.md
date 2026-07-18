# Event-Driven Decoupled (Fan-Out) Architecture

AWS SAA-C03 Reference Architecture — 03

---

## Architecture Overview

A fully decoupled event-driven system using SNS fan-out to independent SQS queues. When one event (e.g. "Order Placed") is published, three independent downstream systems each receive their own copy and process it at their own pace. No downstream service can block or affect another.

---

## Event Flow

### Producer → SNS
1. **Producers** (EC2, Lambda, API Gateway) publish a message to the **SNS Topic**
2. SNS immediately fans the message out to all three subscribed SQS queues simultaneously

### Lane A — Fulfillment
3. **SQS: Fulfillment** buffers the message
4. **Lambda: Fulfillment** processes the message (event source mapping, batch polling)
5. **Auto Scaling Group** (EC2 fulfillment workers) scales based on queue depth — target tracking on `ApproximateNumberOfMessages ÷ InService instances`
6. Workers write fulfillment state to **DynamoDB** (On-Demand — no capacity planning)

### Lane B — Email Notifications
3. **SQS: Email Notifications** buffers the message
4. **Lambda: Email Sender** processes the message
5. Lambda calls **Amazon SES** to deliver the transactional email

### Lane C — Analytics
3. **SQS: Analytics** buffers the message
4. **Lambda: Analytics Writer** processes the message
5. Lambda writes to **Kinesis Firehose**, which buffers and delivers to **S3** as Parquet files for Athena queries

### Failure Path
- After `maxReceiveCount` (e.g. 3) failed processing attempts, messages automatically move to the **Dead Letter Queue**
- **CloudWatch Alarm** fires when DLQ depth > 0
- **SNS Ops Alerts** notifies the engineering team via email/PagerDuty

---

## Services Reference

| Service | Purpose |
|---------|---------|
| SNS Topic | Fan-out: delivers one message to all subscribers simultaneously |
| SQS: Fulfillment | Durable queue for fulfillment worker backlog |
| SQS: Email Notifications | Durable queue for email sending backlog |
| SQS: Analytics | Durable queue for analytics writer backlog |
| Lambda: Fulfillment | Processes fulfillment work (event source mapping) |
| Lambda: Email Sender | Sends transactional emails via SES |
| Lambda: Analytics Writer | Writes events to Kinesis Firehose |
| Auto Scaling Group | EC2 workers for heavy fulfillment processing |
| DynamoDB On-Demand | Stores fulfillment state, order status |
| Amazon SES | Transactional email delivery |
| Kinesis Data Firehose | Buffers and delivers analytics events to S3 |
| S3 (Analytics) | Raw event store in Parquet — queryable via Athena |
| Dead Letter Queue | Captures failed messages after max retry |
| CloudWatch Alarm | Monitors DLQ depth, triggers alert |
| SNS Ops Alerts | Notifies engineering team of failures |

---

## Key Design Decisions

### Why SNS fan-out to separate SQS queues — not one shared queue?

A single shared SQS queue = **competing consumers**. Only one consumer gets each message. If you have three services (fulfillment, email, analytics), they'd fight over messages and each would only see a third of them.

SNS fan-out sends an **independent copy** to each queue. Each downstream service has:
- Its own queue → its own backlog
- Its own retry policy and maxReceiveCount
- Its own DLQ
- Its own scaling signal
- Complete isolation — email failure has zero impact on fulfillment

### Why SQS between SNS and the Lambda/workers?

SNS pushes directly — if the downstream Lambda is throttled or the service is down, the message is lost. Adding SQS provides:
- **Durability** — message sits in queue for up to 14 days
- **Retry** — failed processing retried automatically
- **Backpressure** — workers process at their own rate regardless of producer speed
- **DLQ** — poison pill messages isolated after max retries

### Why target tracking (not simple/step scaling) for the ASG?

Simple scaling waits for a cooldown period after each action — too slow for sudden queue spikes.

Target tracking on `backlog per instance` continuously adjusts:
```
Target metric = ApproximateNumberOfMessages ÷ InService instances
Target value  = 10 (e.g. each worker processes 10 messages comfortably)
```
ASG scales out when backlog per instance > 10. Scales in when queue drains. No cooldown waiting.

### Why DynamoDB On-Demand for fulfillment state?

Order events are spiky (Black Friday) and unpredictable. On-Demand charges per request — no capacity planning, no throttling during spikes.

### Why Kinesis Firehose for analytics (not direct S3)?

Lambda writing directly to S3 creates many small files (one per invocation). Small files are expensive to query in Athena (each file = one S3 API call regardless of size).

Firehose buffers records and delivers batched files (e.g. every 5 minutes or 128 MB). Result: fewer, larger Parquet files → faster and cheaper Athena queries.

---

## SQS Key Configuration

| Setting | Recommended Value | Why |
|---------|------------------|-----|
| Visibility Timeout | 6× Lambda timeout | Prevents duplicate processing during Lambda execution |
| maxReceiveCount | 3 | After 3 failures, move to DLQ |
| Message Retention | 14 days (max) | Keeps messages safe during extended outages |
| Long Polling | 20 seconds | Reduces empty receive API calls → lower cost |

---

## Exam Traps This Architecture Addresses

| Trap | This Architecture Does |
|------|----------------------|
| "One SQS queue for all consumers" | Separate queue per consumer — SNS fan-out |
| "SNS is durable — no messages lost" | SNS is fire-and-forget — SQS adds durability |
| "Simple scaling reacts fast to spikes" | Target tracking on queue depth — no cooldown wait |
| "DLQ on Lambda handles SQS failures" | DLQ configured on the SQS queue, not on Lambda |
| "Visibility timeout = message TTL" | Visibility timeout hides message during processing — not the same as retention |

---

Related: [[../../CHEATSHEET]] | [[../../Domain 2 - Resilient Architectures/SQS]] | [[../../Domain 2 - Resilient Architectures/SNS]]
#aws #architecture #event-driven #saa-c03 #fanout #sqs #sns

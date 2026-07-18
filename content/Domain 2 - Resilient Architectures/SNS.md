# Amazon SNS (Simple Notification Service)

## Overview
Fully managed pub/sub messaging service for A2A (application-to-application) and A2P (application-to-person) messaging.

## Key Concepts
- **Topics** — logical access point for publishers
- **Subscriptions** — endpoints that receive messages
- **Publishers** — send messages to topics

## Subscription Types
- SQS queues
- Lambda functions
- HTTP/HTTPS endpoints
- Email / Email-JSON
- SMS
- Kinesis Data Firehose
- Mobile push notifications

## Features
- Message filtering (JSON policy on subscription)
- Message fanout (one message → many subscribers)
- FIFO topics (with FIFO SQS subscribers)
- Message archiving (Kinesis Data Firehose → S3)
- Large message support (Extended Client Library → S3)
- Cross-region delivery

## SNS + SQS Fan-Out Pattern
```
Producer → SNS Topic → SQS Queue 1 (service A)
                     → SQS Queue 2 (service B)
                     → Lambda (service C)
```
- Guaranteed delivery to each subscriber
- Decoupled, independently scalable consumers

## Message Filtering
- JSON filter policy on each subscription
- Only matching messages delivered to subscriber
- Reduces processing and cost

## Exam Tips
- SNS for fan-out (one-to-many)
- SQS for point-to-point (one-to-one)
- SNS + SQS fan-out is a very common exam pattern
- FIFO SNS only works with FIFO SQS
- Use message filtering to reduce unnecessary processing

---

Related: [[SQS]] | [[EventBridge]]
#aws #sns #messaging #domain2

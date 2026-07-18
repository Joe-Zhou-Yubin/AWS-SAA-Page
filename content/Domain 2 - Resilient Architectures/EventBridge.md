# Amazon EventBridge

## Overview
Serverless event bus for building event-driven architectures. Successor to CloudWatch Events.

## Key Concepts
- **Event Bus** — receives events (default, partner, or custom)
- **Rules** — match events and route to targets
- **Targets** — services that process events (Lambda, SQS, Step Functions, etc.)

## Event Sources
- AWS services (EC2 state change, S3, etc.)
- Custom applications
- SaaS partners (Zendesk, Datadog, Auth0)
- Scheduled events (cron/rate expressions)

## Key Features
- Schema Registry — discover and manage event schemas
- Archive & Replay — store events and replay them later
- Cross-account / cross-region event delivery
- Up to 300+ AWS service integrations as targets
- Event pattern matching (content-based filtering)
- Input transformation (modify event before target)

## EventBridge vs SNS

| Feature | EventBridge | SNS |
|---------|------------|-----|
| Sources | 300+ AWS, SaaS, custom | AWS, custom publishers |
| Filtering | Content-based (JSON pattern) | Message attributes |
| Targets | 20+ native targets | Limited (SQS, Lambda, HTTP) |
| Archive/Replay | Yes | No |
| Schema Discovery | Yes | No |
| Throughput | Lower (varies by region) | Very high |

## Exam Tips
- EventBridge for event-driven with complex routing/filtering
- Default event bus receives all AWS service events
- Use for reacting to AWS resource state changes
- Archive & Replay for debugging/reprocessing
- Scheduled rules replace CloudWatch Events cron jobs
- Better than SNS when you need content-based filtering from AWS events

---

Related: [[SQS]] | [[SNS]] | [[Cross-Cutting/Step Functions]]
#aws #eventbridge #messaging #serverless #domain2

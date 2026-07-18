# Amazon OpenSearch Service

## Overview
Managed service for search, log analytics, and real-time application monitoring. Successor to Amazon Elasticsearch Service.

## Key Features
- Full-text search engine
- Log analytics (ELK stack equivalent)
- Real-time dashboards (OpenSearch Dashboards / Kibana)
- Supports SQL query syntax
- Multi-AZ deployment for HA

## Common Patterns
```
CloudWatch Logs → Subscription Filter → Lambda → OpenSearch
Kinesis Data Streams → Lambda/Firehose → OpenSearch
DynamoDB Streams → Lambda → OpenSearch
```

## Use Cases
- Application search (e-commerce, content search)
- Log analytics (centralized logging)
- Security analytics (SIEM)
- Clickstream analytics
- Real-time monitoring dashboards

## Integration
- Kinesis Data Firehose (direct delivery)
- Lambda (custom ingestion)
- CloudWatch Logs (subscription filters)
- DynamoDB Streams

## Exam Tips
- OpenSearch for search + analytics (not a primary database)
- Common pattern: DynamoDB (primary DB) + OpenSearch (search layer)
- Use for centralized log analytics
- Does NOT support serverless natively (has a serverless option now but know the cluster option)
- Complementary to other databases, not a replacement

---

Related: [[Kinesis]] | [[Domain 2 - Resilient Architectures/DynamoDB]]
#aws #opensearch #analytics #search #domain3

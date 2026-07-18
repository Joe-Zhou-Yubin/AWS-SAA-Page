# Amazon Kinesis

## Overview
Real-time data streaming platform. Collect, process, and analyze streaming data at any scale.

## Kinesis Services

### Kinesis Data Streams
- Real-time data streaming (ingest)
- Producers → Shards → Consumers
- Retention: 1-365 days
- Immutable data (cannot be deleted once inserted)
- Capacity: 1 MB/sec per shard IN, 2 MB/sec per shard OUT
- Consumers: Lambda, KCL applications, Kinesis Data Analytics

#### Capacity Modes
- **Provisioned** — choose number of shards
- **On-Demand** — auto-scales based on throughput

### Amazon Data Firehose (formerly Kinesis Data Firehose)
- Load streaming data into destinations
- Near real-time (60-second buffer minimum)
- Fully managed, auto-scaling
- Destinations: S3, Redshift, OpenSearch, HTTP endpoints, third-party (Splunk, Datadog)
- Data transformation via Lambda
- No data retention (delivery service only)

### Kinesis Data Analytics
- SQL or Apache Flink on streaming data
- Real-time analytics on Kinesis streams
- Use cases: time-series analytics, real-time dashboards, anomaly detection

### Kinesis Video Streams
- Capture, process, and store video streams
- Used for ML, analytics, playback
- Not commonly tested on SAA

## Kinesis vs SQS

| Feature | Kinesis Data Streams | SQS |
|---------|---------------------|-----|
| Model | Streaming (multiple consumers) | Queue (single consumer group) |
| Ordering | Per shard | FIFO only |
| Replay | Yes (data retention) | No (deleted after processing) |
| Consumers | Multiple (fan-out) | One consumer group |
| Throughput | Provisioned (shards) | Unlimited (standard) |
| Use case | Real-time analytics | Decoupling, buffering |

## Exam Tips
- Kinesis Data Streams = real-time, custom processing, multiple consumers
- Firehose = near real-time delivery to storage/analytics (no code)
- Firehose cannot deliver to Kinesis Data Streams
- Data Streams for replay capability
- Use Enhanced Fan-Out for multiple consumers (dedicated throughput)
- Kinesis vs SQS: Kinesis for streaming analytics, SQS for decoupling

## SAA Decision Addendum

### Choose This When
- **Kinesis Data Streams**: multiple consumers need near-real-time processing and replayable ordered records per shard.
- **Kinesis Data Firehose**: managed delivery to S3, Redshift, OpenSearch, Splunk, or HTTP endpoints is enough.
- **Kinesis Data Analytics**: SQL or Apache Flink processing over streams is required.
- **Enhanced fan-out**: consumers need dedicated read throughput from streams.

### Exam Traps
- Kinesis is a streaming service; SQS is usually better for decoupled background jobs.
- Ordering is per shard, not globally across the entire stream.
- Poor partition-key choice can create hot shards.
- Firehose is easier operationally but does not provide the same custom consumer/replay model as Data Streams.

---

Related: [[Domain 3 - High-Performing Architectures/Athena]] | [[Domain 3 - High-Performing Architectures/Redshift]]
#aws #kinesis #streaming #analytics #domain3

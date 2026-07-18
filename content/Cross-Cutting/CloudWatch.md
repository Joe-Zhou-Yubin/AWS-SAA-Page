# Amazon CloudWatch

## Overview
Monitoring and observability service for AWS resources and applications.

## Components

### CloudWatch Metrics
- Default metrics: CPU, Network, Disk (every 5 min for EC2)
- Detailed monitoring: every 1 minute (extra cost)
- Custom metrics: push your own data (PutMetricData)
- Namespaces, dimensions, timestamps
- Resolution: Standard (60s) or High Resolution (1s)
- Retention: auto-aggregated over time (1min→5min→1hr)

### CloudWatch Alarms
- Trigger actions based on metric thresholds
- States: OK, INSUFFICIENT_DATA, ALARM
- Actions: EC2 (stop, terminate, reboot), Auto Scaling, SNS
- Composite Alarms: AND/OR logic on multiple alarms
- Evaluation: period × datapoints

### CloudWatch Logs
- Collect, monitor, and store log files
- Log Groups → Log Streams → Log Events
- Sources: EC2 (CloudWatch Agent), Lambda, ECS, CloudTrail, Route 53, etc.
- **Subscription Filters** → Lambda, Kinesis, OpenSearch
- **Insights** — query and analyze logs (SQL-like)
- **Metric Filters** — extract metrics from logs (e.g., count ERROR occurrences)
- Cross-account log sharing

### CloudWatch Dashboards
- Custom visualization of metrics
- Cross-account, cross-region
- Auto-refresh

## CloudWatch Agent
- Collect additional EC2 metrics (memory, disk usage — not default)
- Collect logs from EC2 instances
- Also works on on-premises servers
- Unified Agent (preferred over older Logs Agent)

## CloudWatch Contributor Insights
- Identify top-N contributors (top talkers, heaviest users)
- Works with VPC Flow Logs, CloudTrail, etc.

## CloudWatch Evidently
- A/B testing and feature flags
- Safer launches with controlled rollouts

## Exam Tips
- EC2 default metrics: CPU, Network, Disk I/O (NOT memory, NOT disk space)
- Memory/disk space requires CloudWatch Agent
- Metric Filters: turn log patterns into metrics → alarms
- Logs Insights for ad-hoc log querying
- Alarms can trigger Auto Scaling, SNS, EC2 actions
- Subscription Filters for real-time log processing

---

Related: [[Domain 1 - Secure Architectures/CloudTrail]] | [[Systems Manager]]
#aws #cloudwatch #monitoring #management #cross-cutting

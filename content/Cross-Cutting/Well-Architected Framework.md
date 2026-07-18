# AWS Well-Architected Framework

## Overview
Set of best practices and guidelines for designing and operating reliable, secure, efficient, cost-effective, and sustainable systems in the cloud.

## Six Pillars

### 1. Operational Excellence
- Run and monitor systems to deliver business value
- Continuously improve processes and procedures
- Key practices: IaC, small frequent changes, anticipate failure, learn from failures
- Services: CloudFormation, Config, CloudTrail, CloudWatch, X-Ray

### 2. Security
- Protect information, systems, and assets
- Key practices: least privilege, defense in depth, automate security, protect data at rest/transit
- Services: IAM, KMS, CloudTrail, GuardDuty, Shield, WAF, Security Hub

### 3. Reliability
- Recover from failures and meet demand
- Key practices: auto-recover, scale horizontally, stop guessing capacity, manage change through automation
- Services: Auto Scaling, CloudWatch, Route 53, S3, Multi-AZ, backup strategies

### 4. Performance Efficiency
- Use computing resources efficiently as demand changes
- Key practices: use managed services, go global in minutes, experiment, mechanical sympathy
- Services: Auto Scaling, Lambda, CloudFront, ElastiCache, appropriate instance types

### 5. Cost Optimization
- Avoid unnecessary costs
- Key practices: adopt consumption model, measure efficiency, stop spending on undifferentiated heavy lifting
- Services: Cost Explorer, Budgets, Reserved Instances, Spot, right-sizing

### 6. Sustainability
- Minimize environmental impact
- Key practices: understand your impact, maximize utilization, use managed services, reduce downstream impact
- Services: Compute Optimizer, Auto Scaling, Graviton, serverless

## Well-Architected Tool
- Free AWS service
- Review workloads against best practices
- Generates improvement plan
- Track progress over time

## Design Principles
- Stop guessing capacity needs
- Test systems at production scale
- Automate for experimentation
- Allow for evolutionary architectures
- Drive architectures using data
- Improve through game days

## Exam Tips
- Know the 6 pillars and their focus areas
- Exam questions often align with pillar principles
- "Operational overhead" questions → managed services, serverless
- "Resilient" → Multi-AZ, Auto Scaling, backups
- "Cost-effective" → right-sizing, Spot, reserved, lifecycle policies
- "Secure" → least privilege, encryption, monitoring

---

Related: [[Domain 4 - Cost-Optimized Architectures/Cost Management Tools]] | [[CloudFormation]]
#aws #well-architected #architecture #cross-cutting

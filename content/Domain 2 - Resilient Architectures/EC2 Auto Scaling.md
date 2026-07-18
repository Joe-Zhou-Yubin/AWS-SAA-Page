# EC2 Auto Scaling

## Overview
Automatically adjusts the number of EC2 instances to handle load changes while maintaining availability and minimizing cost.

## Components

### Launch Template
- AMI, instance type, key pair, security groups, user data
- Versioned (preferred over Launch Configurations which are legacy)
- Supports multiple instance types and purchase options

### Auto Scaling Group (ASG)
- Minimum, desired, maximum capacity
- Health checks: EC2 (default) or ELB
- Can span multiple AZs within a region

## Scaling Policies

### Dynamic Scaling
- **Target Tracking** — maintain a target metric (e.g., CPU at 50%)
- **Step Scaling** — add/remove based on CloudWatch alarm thresholds
- **Simple Scaling** — single adjustment, waits for cooldown

### Predictive Scaling
- Uses ML to forecast traffic patterns
- Pre-provisions capacity ahead of predicted demand

### Scheduled Scaling
- Scale at specific times (known traffic patterns)

## Key Features
- **Cooldown period** — prevents rapid scaling (default 300s)
- **Warm pools** — pre-initialized instances for faster scaling
- **Instance refresh** — rolling update of instances
- **Lifecycle hooks** — custom actions during launch/terminate
- **Termination policies** — control which instances terminate first

## Health Check Grace Period
- Time to wait before checking health of new instance
- Important: set long enough for app to start

## Exam Tips
- ASG + ALB = high availability pattern
- Target Tracking is simplest and most common
- Use lifecycle hooks for custom startup/shutdown scripts
- Warm pools reduce scale-out latency
- Default termination: AZ with most instances → oldest launch template

---

Related: [[Elastic Load Balancing]] | [[Domain 3 - High-Performing Architectures/EC2]]
#aws #auto-scaling #compute #domain2

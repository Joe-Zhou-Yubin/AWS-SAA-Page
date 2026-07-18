# AWS Systems Manager (SSM)

## Overview
Operational management service for managing AWS resources and on-premises servers at scale.

## Key Capabilities

### Session Manager
- Secure shell access without SSH/port 22
- No bastion hosts needed
- Audit via CloudTrail, logs to S3/CloudWatch
- IAM-based access control

### Parameter Store
- Centralized storage for config data and secrets
- Hierarchical (paths: /app/prod/db-password)
- Types: String, StringList, SecureString (KMS encrypted)
- Free tier: Standard parameters (4 KB, 10,000 max)
- Advanced: 8 KB, 100,000 max, policies (expiration, notification)
- Versioning and history

### Patch Manager
- Automate OS and software patching
- Patch Baselines (define which patches to apply)
- Maintenance Windows (schedule when to patch)
- Compliance reporting

### Run Command
- Execute scripts/commands on managed instances remotely
- No SSH required (uses SSM Agent)
- Rate control (concurrency, error threshold)
- Output to S3 or CloudWatch

### Automation
- Automate common maintenance tasks
- Runbooks (predefined or custom)
- Example: restart instances, create AMIs, patch and reboot
- EventBridge integration for triggered automation

### Inventory
- Collect metadata from managed instances
- Applications, network config, Windows updates, etc.
- Centralized view of fleet state

### State Manager
- Define and maintain consistent configuration
- Associations: define desired state (e.g., ensure agent installed)

## SSM Agent
- Pre-installed on Amazon Linux 2, Ubuntu, Windows AMIs
- Required for Session Manager, Run Command, Patch Manager
- Instance needs IAM role with SSM permissions

## Exam Tips
- Session Manager replaces bastion hosts (more secure, auditable)
- Parameter Store for configuration (vs Secrets Manager for rotation)
- Run Command for fleet-wide script execution
- Patch Manager + Maintenance Windows for patching strategy
- SSM Agent required (verify IAM role + network connectivity)

---

Related: [[Domain 1 - Secure Architectures/Secrets Manager]] | [[CloudWatch]]
#aws #systems-manager #ssm #management #cross-cutting

# AWS CloudFormation

## Overview
Infrastructure as Code (IaC) service. Define AWS resources in declarative JSON/YAML templates.

## Key Concepts
- **Template** — YAML/JSON file defining resources
- **Stack** — collection of resources created from a template
- **StackSet** — deploy stacks across multiple accounts/regions
- **Change Set** — preview changes before applying

## Template Structure
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: My template
Parameters:       # Input values
Mappings:         # Static key-value lookup
Conditions:       # Conditional resource creation
Resources:        # AWS resources (REQUIRED)
Outputs:          # Return values (cross-stack references)
```

## Key Features
- **Drift Detection** — identify manual changes to stack resources
- **Rollback** — automatic on failure (configurable)
- **Nested Stacks** — reusable templates within templates
- **Cross-Stack References** — share outputs between stacks (Export/Import)
- **Stack Policies** — protect resources from updates
- **DeletionPolicy** — Retain, Snapshot, or Delete on stack deletion

## Helper Scripts (EC2)
- `cfn-init` — install packages, configure services
- `cfn-signal` — signal creation complete (with WaitCondition)
- `cfn-hup` — detect metadata changes and rerun cfn-init

## Exam Tips
- CloudFormation for repeatable, automated infrastructure
- Use Parameters for environment-specific values
- Use Mappings for static lookups (region → AMI)
- Nested Stacks for reusable components
- StackSets for multi-account/multi-region deployment
- DeletionPolicy: Retain for databases to prevent data loss

---

Related: [[Cross-Cutting/Systems Manager]] | [[Domain 1 - Secure Architectures/AWS Organizations]]
#aws #cloudformation #iac #management #cross-cutting

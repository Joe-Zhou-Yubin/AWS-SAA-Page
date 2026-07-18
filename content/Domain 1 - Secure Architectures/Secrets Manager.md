# AWS Secrets Manager

## Overview
Service for storing, rotating, and managing secrets (database credentials, API keys, tokens).

## Key Features
- Automatic rotation (using Lambda functions)
- Native integration with RDS, Redshift, DocumentDB
- Encrypted using KMS
- Cross-account access via resource policies
- Versioning (current, previous, pending)

## Secrets Manager vs Parameter Store

| Feature | Secrets Manager | SSM Parameter Store |
|---------|----------------|-------------------|
| Rotation | Built-in automatic | Manual (Lambda needed) |
| Cost | ~$0.40/secret/month | Free tier available |
| Max size | 64 KB | 8 KB (standard) / 4 KB (advanced) |
| Cross-account | Yes (resource policy) | No |
| RDS integration | Native | No |
| KMS required | Always encrypted | Optional |

## Rotation
- Automatic rotation schedule (days)
- Multi-user rotation strategy for zero-downtime
- Custom rotation via Lambda for non-RDS secrets

## Exam Tips
- Use Secrets Manager for database credentials that need rotation
- Use Parameter Store for configuration values, feature flags, non-rotating secrets
- If question mentions "automatic rotation" → Secrets Manager
- If question mentions "cost-effective parameter storage" → Parameter Store

---

Related: [[KMS]] | [[Cross-Cutting/Systems Manager]]
#aws #secrets-manager #security #domain1

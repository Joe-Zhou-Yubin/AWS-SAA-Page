# Database Cost Optimization

## RDS Cost Strategies
- **Reserved Instances** — 1 or 3 year commitment (up to 72% off)
- **Right-sizing** — use CloudWatch metrics to identify over-provisioned instances
- **Aurora Serverless** — pay per ACU-second for variable workloads
- **Stop unused instances** — stop dev/test RDS instances when not needed (max 7 days)
- **Read Replicas** — offload reads from expensive primary
- **Storage Auto Scaling** — avoid over-provisioning storage

## DynamoDB Cost Strategies
- **On-Demand vs Provisioned** — On-Demand for unpredictable, Provisioned for steady
- **Reserved Capacity** — commit WCU/RCU for discounts
- **Auto Scaling** — adjust provisioned capacity to demand
- **TTL** — auto-delete expired items (no WCU cost)
- **DAX** — reduce read costs by caching
- **Global Tables** — only enable in regions that need it

## ElastiCache vs DAX Decision
- **DAX** — DynamoDB-specific caching (simple, drop-in)
- **ElastiCache** — general purpose, aggregation/computation caching, session store
- DAX for simple key-value lookups on DynamoDB
- ElastiCache for complex computed results or multi-source caching

## Aurora Serverless Use Case
- Development and testing environments
- Infrequent or unpredictable workloads
- Multi-tenant SaaS (scale per tenant activity)
- Eliminates waste from idle provisioned capacity

## General Database Optimization
- Choose the right database for the workload (avoid using RDS where DynamoDB fits)
- Use caching layers to reduce database load
- Optimize queries and indexes before scaling up
- Consider Graviton-based instances (better price/performance)

## Exam Tips
- Aurora Serverless for variable/intermittent workloads (cost effective)
- DynamoDB On-Demand for spiky traffic (no idle capacity cost)
- RDS Reserved Instances for steady production databases
- Stop non-production databases outside business hours
- DAX specifically for DynamoDB read-heavy patterns

## SAA Decision Addendum

### Choose This When
- **DynamoDB on-demand**: new, unpredictable, or spiky NoSQL traffic.
- **DynamoDB provisioned + auto scaling**: predictable access patterns and capacity tuning are possible.
- **RDS Reserved Instance**: relational database usage is steady.
- **Aurora Serverless**: relational usage is intermittent or variable and compatible with serverless behavior.
- **Caching**: repeated reads can be served without violating freshness requirements.

### Exam Traps
- Read replicas cost money; add them only when read scaling or DR justifies them.
- Multi-AZ is an availability cost, not a read-scaling optimization.
- Over-retaining snapshots and PITR can quietly drive database cost.
- Migrating to DynamoDB only saves money if the access pattern fits NoSQL.

---

Related: [[Domain 2 - Resilient Architectures/RDS]] | [[Domain 2 - Resilient Architectures/DynamoDB]] | [[Cost Management Tools]]
#aws #database #cost-optimization #domain4

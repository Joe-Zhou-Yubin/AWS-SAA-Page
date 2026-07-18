# Amazon ElastiCache

## Overview
Managed in-memory caching service. Supports Redis and Memcached engines.

## Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data structures | Rich (strings, lists, sets, hashes) | Simple (strings) |
| Persistence | Yes (backup/restore) | No |
| Replication | Multi-AZ with failover | No |
| Cluster mode | Yes (sharding) | Yes (sharding) |
| Pub/Sub | Yes | No |
| Lua scripting | Yes | No |
| Backup/Restore | Yes | No |
| Multi-threaded | No | Yes |

## Use Cases
- Database query caching (reduce RDS load)
- Session store (stateless applications)
- Real-time leaderboards (Redis sorted sets)
- Pub/Sub messaging (Redis)

## Caching Strategies
- **Lazy Loading** — load data on cache miss (stale data possible)
- **Write Through** — write to cache on DB write (higher write latency)
- **TTL** — set expiration to balance freshness and load

## Redis Cluster Mode
- Data is sharded across multiple nodes
- Each shard has a primary and replicas
- Scales both reads and writes
- Supports up to 500 nodes

## Security
- Redis AUTH (password/token)
- IAM auth (Redis 7+)
- In-transit encryption (TLS)
- At-rest encryption (KMS)
- VPC security groups

## Exam Tips
- Redis for HA, persistence, advanced data types
- Memcached for simple caching, multi-threaded performance
- ElastiCache for computed aggregations (vs DAX for DynamoDB caching)
- Use for session management to make apps stateless
- Cannot be accessed from outside VPC (no public access)

---

Related: [[DynamoDB]] | [[RDS]]
#aws #elasticache #database #caching #domain2

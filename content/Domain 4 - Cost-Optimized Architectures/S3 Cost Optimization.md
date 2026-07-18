# S3 Cost Optimization

## Storage Class Costs (approximate, us-east-1)

| Class | Storage $/GB/month | Retrieval |
|-------|-------------------|-----------|
| Standard | $0.023 | Free |
| Intelligent-Tiering | $0.023 (frequent) | Monitoring fee |
| Standard-IA | $0.0125 | Per GB retrieved |
| One Zone-IA | $0.01 | Per GB retrieved |
| Glacier Instant | $0.004 | Per GB retrieved |
| Glacier Flexible | $0.0036 | Per retrieval request + time |
| Glacier Deep Archive | $0.00099 | Per retrieval request + time |

## Cost Optimization Strategies

### 1. Lifecycle Policies
- Automatically transition objects between classes
- Example: Standard → IA (30 days) → Glacier (90 days) → Delete (365 days)
- Apply to specific prefixes or tags

### 2. S3 Intelligent-Tiering
- Automatic movement between access tiers
- No retrieval fees
- Small monthly monitoring fee per object
- Best for unknown or changing access patterns
- Tiers: Frequent → Infrequent (30 days) → Archive Instant (90 days) → Archive (optional) → Deep Archive (optional)

### 3. S3 Analytics / Storage Class Analysis
- Analyzes access patterns
- Recommends when to move to IA
- Takes 24-48 hours to show initial results

### 4. Compression & Efficient Formats
- Store data in compressed formats
- Use columnar formats (Parquet) for analytics data

### 5. Requester Pays
- Requester pays for requests and data transfer
- Use when sharing large datasets

## Data Transfer Costs
- **Ingress** — always free
- **Same region transfer** — free (within S3)
- **Cross-region** — charged per GB
- **To Internet** — charged per GB (most expensive)
- **CloudFront** — cheaper than direct S3 internet transfer
- **S3 Transfer Acceleration** — higher cost, better performance
- **VPC Gateway Endpoint** — free data transfer to S3 from VPC

## Exam Tips
- Use lifecycle policies aggressively for cost savings
- Intelligent-Tiering for unpredictable access patterns
- One Zone-IA only for reproducible data (lower durability)
- Glacier Deep Archive is cheapest storage ($1/TB/month)
- Use VPC Gateway Endpoint to avoid data transfer charges
- Multipart upload doesn't save money but improves reliability

## SAA Decision Addendum

### Choose This When
- **Lifecycle policy**: data naturally gets colder or should expire after a known period.
- **Intelligent-Tiering**: access pattern is unknown or changes over time.
- **Standard-IA**: data is infrequently accessed but must remain multi-AZ and quickly retrievable.
- **One Zone-IA**: data is reproducible or a secondary copy and lower availability is acceptable.
- **Requester Pays**: external consumers should pay request and transfer charges for shared datasets.

### Exam Traps
- Storage class choice must include retrieval cost, request cost, minimum duration, and minimum object size.
- Intelligent-Tiering is not automatically cheapest for many tiny or short-lived objects.
- S3 Transfer Acceleration improves long-distance upload/download performance but costs more.
- CloudFront can reduce origin egress and request load when content is cacheable.

---

Related: [[Domain 2 - Resilient Architectures/S3]] | [[Cost Management Tools]]
#aws #s3 #cost-optimization #storage #domain4

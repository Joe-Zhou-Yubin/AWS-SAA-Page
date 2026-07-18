# Fully Serverless Web/API Architecture

AWS SAA-C03 Reference Architecture — 02

---

## Architecture Overview

A fully serverless architecture for a web application and REST API. No EC2 instances, no servers to patch or scale manually. Every component scales automatically — from zero to millions of requests — and you pay only for what you use.

---

## Request Flow (Primary Path)

1. **End User** opens the app in a browser or mobile client
2. **Route 53** resolves the domain → returns CloudFront's anycast IP
3. **CloudFront** + **WAF** receives the request. WAF inspects for SQLi, XSS, rate limits. Cache hit → static asset returned from edge immediately
4. **S3 (Static Frontend)** serves the React/Next.js build via CloudFront with OAC — bucket stays private
5. API calls matching `/api/*` are forwarded from CloudFront → **API Gateway (HTTP API)**
6. **API Gateway** validates the **JWT token** against **Cognito User Pool** (built-in JWT authorizer — no Lambda needed)
7. **API Gateway** invokes **Lambda** (business logic function)
8. Lambda retrieves DB credentials from **Secrets Manager** at cold start (cached in memory for warm invocations)
9. Lambda connects via **RDS Proxy** → **Aurora Serverless v2** for relational SQL queries
   - OR directly to **DynamoDB On-Demand** for key-value / document access patterns
10. Lambda publishes domain events to **EventBridge** (async, fire-and-forget)
11. **EventBridge** rule matches the event → invokes **Notification Lambda**
12. **Notification Lambda** publishes to **SNS** → fans out to email, SMS, push subscribers

---

## Monitoring Path

- Lambda automatically sends logs to **CloudWatch Logs** (every invocation)
- Lambda emits metrics (duration, errors, throttles) to **CloudWatch Metrics**
- **X-Ray** traces the full request path: API Gateway → Lambda → RDS Proxy → Aurora — shows cold starts, slow DB queries, downstream latency
- CloudWatch Alarm → SNS alert when error rate or p99 latency breaches threshold

---

## Services Reference

| Service | Purpose |
|---------|---------|
| Route 53 | DNS, health checks |
| CloudFront | CDN, HTTPS, OAC for S3, WAF attachment point |
| AWS WAF | Rate limiting, SQLi/XSS rules at edge |
| S3 | Static frontend hosting (React/Next.js build) |
| Cognito User Pool | User sign-up, sign-in, JWT issuance, social login |
| API Gateway (HTTP API) | Managed API endpoint, JWT authorizer, throttling |
| Lambda (Business Logic) | Stateless compute, scales 0–1000 concurrency automatically |
| RDS Proxy | Connection pooling between Lambda and Aurora |
| Aurora Serverless v2 | Relational DB, scales in ACU increments, no idle if paused |
| DynamoDB On-Demand | NoSQL alternative for key-value patterns, sub-10ms, no provisioning |
| Secrets Manager | DB credentials with auto-rotation |
| EventBridge | Async event bus for domain events (order placed, user registered) |
| Notification Lambda | Handles async notification logic |
| SNS | Fan-out to email, SMS, mobile push |
| CloudWatch | Metrics, logs, alarms, dashboards |
| X-Ray | Distributed tracing, cold start analysis |

---

## Key Design Decisions

### Why HTTP API over REST API for API Gateway?
- HTTP API: ~70% cheaper, lower latency, native JWT authorizer
- REST API: needed only for API keys, usage plans, request transformation, VTL mapping
- For Lambda proxy + Cognito JWT auth → HTTP API is the default answer

### Why RDS Proxy between Lambda and Aurora?
Lambda can scale to 1,000+ concurrent executions. Each opens a new DB connection. Aurora has a connection limit (~100 for small instances). Without RDS Proxy, Aurora rejects connections during traffic spikes. RDS Proxy pools connections, queuing Lambda requests and releasing connections when Lambda finishes. Also speeds up Aurora failover — proxy holds connections during the ~30s failover window.

### Aurora Serverless v2 vs DynamoDB — When to Choose
| Factor | Aurora Serverless v2 | DynamoDB On-Demand |
|--------|---------------------|-------------------|
| Schema | Relational, complex joins | Key-value / document |
| Cold start | ~15s to resume from pause | None |
| Cost model | Per ACU-second | Per request |
| Latency | 1-5ms | Sub-10ms |
| Best for | Existing SQL apps, complex queries | Shopping carts, sessions, IoT |

### Why Cognito over custom auth?
- No auth code to maintain
- JWT issued by Cognito, verified by API Gateway natively — Lambda never touches auth tokens
- Social login (Google, Facebook, Apple) built-in
- MFA, password policies, email verification all managed

### Why EventBridge for async side effects?
After the primary write succeeds, Lambda publishes one event. EventBridge routes it to:
- Notification Lambda (send email/SMS)
- Analytics Lambda (update dashboards)
- Audit Lambda (write to compliance log)
All three are fully decoupled — adding a new consumer doesn't change Lambda's code.

---

## Exam Traps This Architecture Addresses

| Trap | This Architecture Does |
|------|----------------------|
| "Lambda needs RDS = connection exhaustion" | RDS Proxy handles connection pooling |
| "Reserved concurrency eliminates cold starts" | Provisioned Concurrency eliminates cold starts — not Reserved |
| "HTTP API and REST API are the same" | HTTP API is cheaper + lower latency; REST API has more features |
| "Cognito = IAM Identity Center" | Cognito = app users. IAM Identity Center = workforce AWS access |
| "VPC-attached Lambda has internet access" | VPC Lambda loses internet — needs NAT GW if both VPC + internet needed |

---

Related: [[../../CHEATSHEET]] | [[../../Domain 3 - High-Performing Architectures/Lambda]] | [[../../Domain 3 - High-Performing Architectures/API Gateway]]
#aws #architecture #serverless #saa-c03

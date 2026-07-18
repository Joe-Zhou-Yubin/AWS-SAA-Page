# Amazon API Gateway

## Overview
Fully managed service to create, publish, maintain, and secure APIs at any scale. Handles REST, HTTP, and WebSocket APIs.

## API Types

| Type | Use Case | Features |
|------|----------|----------|
| REST API | Full-featured | Caching, WAF, usage plans, API keys |
| HTTP API | Simple, low-cost | Faster, cheaper, less features |
| WebSocket API | Real-time two-way | Chat, streaming, notifications |

## Integrations
- Lambda (most common — serverless API)
- HTTP backends (on-premises, ALB)
- AWS services (Step Functions, SQS, DynamoDB directly)
- Mock (return static responses)

## Key Features
- **Stages** — dev, staging, prod (each with own URL)
- **Throttling** — 10,000 req/sec default (account-level), burst 5,000
- **Caching** — TTL 0-3600 seconds, reduces backend calls
- **Usage Plans & API Keys** — rate limiting per customer
- **Request/Response transformation**
- **OpenAPI/Swagger import**
- **Custom domain names** (with ACM certificate)

## Authentication & Authorization
- IAM policies (Sig v4)
- Cognito User Pools (JWT tokens)
- Lambda Authorizer (custom auth logic, returns IAM policy)
- API Keys (not for auth — for tracking/throttling only)

## Deployment
- Canary deployments (split traffic between stages)
- Stage variables (environment-specific configuration)
- Integration with CloudWatch for logging and metrics

## Exam Tips
- API Gateway + Lambda = fully serverless API
- REST API for features (caching, WAF); HTTP API for cost/simplicity
- Lambda Authorizer for custom/third-party token validation
- Cognito Authorizer for standard JWT auth
- 29-second timeout for all integrations
- Use caching to reduce Lambda invocations and cost

## SAA Decision Addendum

### Choose This When
- **HTTP API**: low-cost, low-latency API for common proxy use cases.
- **REST API**: advanced API Gateway features such as usage plans, API keys, request validation, or caching are required.
- **WebSocket API**: bidirectional real-time client communication is required.
- **Edge-optimized endpoint**: global public clients should enter through CloudFront edge locations.
- **Private API**: API must be accessible only through an interface VPC endpoint.

### Exam Traps
- API Gateway throttling protects backends and can be part of cost control.
- Lambda proxy integration passes most request details to Lambda; non-proxy mapping templates add transformation control.
- API Gateway cache can reduce backend load but can serve stale responses if TTLs are wrong.
- Use WAF for Layer 7 protection when exposed publicly.

---

Related: [[Lambda]] | [[Domain 1 - Secure Architectures/Cognito]] | [[Domain 1 - Secure Architectures/WAF & Shield]]
#aws #api-gateway #serverless #domain3

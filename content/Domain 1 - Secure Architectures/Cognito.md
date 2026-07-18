# Amazon Cognito

## Overview
Amazon Cognito provides authentication, authorization, and user management for web and mobile apps.

## Components

### User Pools
- User directory for sign-up and sign-in
- Provides JWT tokens
- Supports MFA, email/phone verification
- Social identity providers (Google, Facebook, Apple)
- SAML and OIDC federation
- Customizable sign-in UI
- Lambda triggers for custom workflows

### Identity Pools (Federated Identities)
- Provides temporary AWS credentials
- Users can be authenticated (from User Pool or external IdP) or guest
- Maps users to IAM roles
- Fine-grained access control with policy variables

## Common Architecture
```
User → Cognito User Pool (authenticate) → JWT Token
     → Cognito Identity Pool (exchange token) → Temporary AWS Credentials
     → Access AWS Services (S3, DynamoDB, etc.)
```

## Key Use Cases
- Mobile/web app authentication
- API Gateway authorization (Cognito Authorizer)
- Providing temporary AWS access to app users
- Federated sign-in with social/enterprise providers

## Exam Tips
- User Pools = authentication (who are you?)
- Identity Pools = authorization (what can you access?)
- Use Cognito over custom auth solutions when possible
- ALB can integrate natively with Cognito User Pools

---

Related: [[IAM]] | [[Domain 3 - High-Performing Architectures/API Gateway]]
#aws #cognito #security #domain1

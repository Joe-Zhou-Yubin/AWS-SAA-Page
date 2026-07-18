# AWS Certificate Manager (ACM)

## Overview
Provision, manage, and deploy SSL/TLS certificates for AWS services and internal resources.

## Key Features
- Free public SSL/TLS certificates
- Automatic renewal
- Integrated with ALB, CloudFront, API Gateway, Elastic Beanstalk
- Cannot be used directly on EC2 instances
- Regional service (except CloudFront which uses us-east-1)

## Certificate Types
- **Public certificates** — free, issued by Amazon
- **Private certificates** — via ACM Private CA (paid), for internal resources

## Validation Methods
- **DNS validation** — add CNAME record (recommended, enables auto-renewal)
- **Email validation** — verification email to domain contacts

## Exam Tips
- ACM certificates are FREE for public certs
- Must be in the same region as the resource (except CloudFront → us-east-1)
- For EC2, use a third-party cert or ACM Private CA
- Auto-renewal only works with DNS validation
- Use with ALB for SSL termination

---

Related: [[Domain 2 - Resilient Architectures/Elastic Load Balancing]] | [[Domain 3 - High-Performing Architectures/CloudFront]]
#aws #acm #ssl #security #domain1

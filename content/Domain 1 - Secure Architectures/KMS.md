# AWS KMS (Key Management Service)

## Overview
Managed service for creating and controlling encryption keys used to encrypt data across AWS services.

## Key Types

| Type | Management | Usage |
|------|-----------|-------|
| AWS Managed Keys | AWS manages (aws/service) | Default encryption for services |
| Customer Managed Keys (CMKs) | You manage | Full control, rotation, policies |
| AWS Owned Keys | AWS owns | Used internally, invisible to you |

## Encryption Types
- **Symmetric (AES-256)** — single key for encrypt/decrypt, never leaves KMS unencrypted
- **Asymmetric (RSA, ECC)** — public/private key pair, public key downloadable

## Key Policies
- Default: root account has full access
- Custom: define who can administer and use the key
- Required for cross-account key usage

## Envelope Encryption
1. KMS generates a Data Encryption Key (DEK)
2. DEK encrypts your data (client-side)
3. Encrypted DEK stored alongside encrypted data
4. KMS only decrypts the DEK when needed

> Used when data > 4KB (KMS direct encryption limited to 4KB)

## Key Rotation
- **Automatic rotation**: yearly for customer managed keys (optional, enable it)
- **Manual rotation**: create new key, update alias
- AWS managed keys rotate every 3 years automatically

## Multi-Region Keys
- Replicate keys across regions
- Same key material in multiple regions
- Use case: encrypt in one region, decrypt in another (DynamoDB Global Tables, Aurora Global)

## S3 Encryption Options
- **SSE-S3**: AWS manages keys (default)
- **SSE-KMS**: KMS manages keys (audit trail via CloudTrail)
- **SSE-C**: customer provides keys (AWS never stores them)
- **Client-side encryption**: encrypt before uploading

## Exam Tips
- KMS keys are regional (except multi-region keys)
- For cross-account: key policy + IAM policy both needed
- SSE-KMS has a KMS API call quota (can throttle high-throughput)
- Use `GenerateDataKey` for envelope encryption
- CloudTrail logs all KMS API calls

## SAA Decision Addendum

### Choose This When
- **AWS managed key**: simplest service-integrated encryption with minimal administration.
- **Customer managed key**: you need key policy control, rotation control, aliases, grants, or audit separation.
- **Multi-Region key**: the same logical key material is needed for client-side or cross-Region cryptographic workflows.
- **CloudHSM/custom key store**: dedicated HSM control or strict compliance is explicitly required.
- **Envelope encryption**: data is larger than what KMS should encrypt directly.

### Exam Traps
- KMS keys are Regional unless you explicitly create multi-Region keys.
- Key rotation does not re-encrypt old data immediately; old backing keys remain available for decrypt.
- A full S3 admin still needs KMS permission to decrypt SSE-KMS objects.
- Disabling or scheduling deletion of a key can make encrypted data unrecoverable.

---

Related: [[CloudHSM]] | [[Secrets Manager]] | [[Domain 2 - Resilient Architectures/S3]]
#aws #kms #encryption #security #domain1

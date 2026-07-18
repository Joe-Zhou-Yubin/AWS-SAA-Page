# AWS CloudHSM

## Overview
Dedicated hardware security module (HSM) in the AWS Cloud. You manage your own encryption keys entirely.

## Key Differences from KMS

| Feature | KMS | CloudHSM |
|---------|-----|----------|
| Management | Shared (multi-tenant) | Dedicated (single-tenant) |
| Key control | AWS manages HSM | You manage HSM |
| Integration | Native AWS service integration | Custom application integration |
| Compliance | FIPS 140-2 Level 3 | FIPS 140-2 Level 3 |
| Availability | AWS manages HA | You deploy across AZs |
| Price | Per key + API calls | Per HSM instance/hour |

## Architecture
- Deployed in your VPC
- Clustered across AZs for HA
- AWS manages hardware, you manage keys and crypto operations
- Irretrievable if you lose credentials — AWS cannot recover keys

## Use Cases
- Regulatory compliance requiring single-tenant HSM
- SSL/TLS offloading
- Custom key stores for KMS (KMS backed by CloudHSM)
- Oracle TDE (Transparent Data Encryption)

## Exam Tips
- Choose CloudHSM when you need dedicated hardware or must manage keys yourself
- CloudHSM client runs in your VPC
- Supports both symmetric and asymmetric keys
- If question mentions "FIPS 140-2 Level 3" and "dedicated", think CloudHSM
- Can be used as custom key store for KMS

---

Related: [[KMS]] | [[Secrets Manager]]
#aws #cloudhsm #encryption #security #domain1

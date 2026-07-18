# Amazon Macie

## Overview
Fully managed data privacy and security service that uses ML to discover, classify, and protect sensitive data in S3.

## Key Features
- Discovers PII (names, addresses, credit cards, SSNs)
- Scans S3 buckets for sensitive data
- Provides alerts via findings
- Integrates with EventBridge for automated remediation
- Supports custom data identifiers (regex patterns)
- Multi-account support via Organizations

## How It Works
1. Enable Macie → select S3 buckets
2. Macie analyzes objects using ML + pattern matching
3. Generates findings for sensitive data discovered
4. Findings → EventBridge → SNS/Lambda for alerting/remediation

## Exam Tips
- Macie = sensitive data discovery in S3
- Think "PII detection" → Macie
- Automated scanning on schedule or one-time
- Use EventBridge integration for automated responses

---

Related: [[GuardDuty]] | [[Inspector]] | [[Security Hub]]
#aws #macie #data-privacy #security #domain1

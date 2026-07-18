# Amazon CloudFront

## Overview
Global Content Delivery Network (CDN). Caches content at edge locations for low-latency delivery.

## Origins
- **S3 bucket** — distribute static files, use OAC (Origin Access Control)
- **Custom Origin (HTTP)** — ALB, EC2, any HTTP backend
- **Origin Groups** — primary + failover origin for HA

## Key Features
- 400+ edge locations globally
- DDoS protection (integrated with Shield)
- SSL/TLS termination (ACM certificates in us-east-1)
- Field-level encryption
- Real-time logs (Kinesis Data Streams)
- Standard logs (S3)

## Cache Behaviors
- Path pattern matching (/images/*, /api/*)
- TTL configuration (default, min, max)
- Cache based on: headers, query strings, cookies
- Origin request policy (what to forward to origin)

## Cache Invalidation
- Invalidate specific paths (e.g., /index.html, /images/*)
- Costs per invalidation request
- Alternative: use versioned file names (app-v2.js)

## CloudFront Functions vs Lambda@Edge

| Feature | CloudFront Functions | Lambda@Edge |
|---------|---------------------|-------------|
| Runtime | JavaScript | Node.js, Python |
| Execution | Viewer request/response only | All 4 events |
| Duration | < 1 ms | Up to 30s |
| Scale | Millions of req/sec | Thousands |
| Use case | URL rewrites, headers, simple redirects | Complex logic, API calls |

## Signed URLs & Cookies
- **Signed URLs** — restrict access to individual files
- **Signed Cookies** — restrict access to multiple files
- Use CloudFront key pairs or trusted key groups
- For S3: CloudFront signed URL ≠ S3 pre-signed URL

## Geo Restriction
- Allowlist or blocklist countries
- Uses third-party GeoIP database

## Exam Tips
- CloudFront + S3 OAC = secure static website delivery
- Use us-east-1 for ACM certificates with CloudFront
- CloudFront Functions for lightweight edge logic
- Lambda@Edge for complex transformations
- Origin Groups for origin failover
- Caching reduces load and cost on origins

## SAA Decision Addendum

### Choose This When
- **CloudFront**: HTTP/HTTPS content needs global caching, TLS termination, signed URLs/cookies, or origin offload.
- **Origin Access Control**: S3 origin must stay private while CloudFront serves objects.
- **Origin group**: CloudFront should fail over from primary to secondary origin.
- **CloudFront Functions**: lightweight viewer request/response changes at very high scale.
- **Lambda@Edge**: more complex edge logic or origin request/response customization is needed.

### Exam Traps
- ACM certificates for CloudFront must be created or imported in us-east-1.
- Invalidations work, but versioned filenames are often cheaper and cleaner for frequent releases.
- CloudFront signed URLs are different from S3 pre-signed URLs.
- CloudFront is optimized for content delivery; Global Accelerator is usually better for TCP/UDP app acceleration.

---

Related: [[Domain 1 - Secure Architectures/WAF & Shield]] | [[Domain 1 - Secure Architectures/Certificate Manager]]
#aws #cloudfront #cdn #networking #domain3

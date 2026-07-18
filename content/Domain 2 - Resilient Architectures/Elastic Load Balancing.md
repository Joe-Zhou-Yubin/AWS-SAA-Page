# Elastic Load Balancing (ELB)

## Overview
Distributes incoming traffic across multiple targets (EC2, containers, IPs, Lambda).

## Load Balancer Types

### Application Load Balancer (ALB) — Layer 7
- HTTP/HTTPS traffic
- Path-based & host-based routing
- Supports WebSocket, HTTP/2
- Target groups: EC2, ECS, Lambda, IP addresses
- Fixed hostname (xxx.region.elb.amazonaws.com)
- Sticky sessions (cookie-based)
- Native integration with Cognito, WAF

### Network Load Balancer (NLB) — Layer 4
- TCP/UDP/TLS traffic
- Ultra-high performance (~millions of requests/sec)
- Static IP per AZ (or Elastic IP)
- Preserves source IP
- Target groups: EC2, IP addresses, ALB
- Low latency (~100ms vs ~400ms for ALB)

### Gateway Load Balancer (GLB) — Layer 3
- Transparent network gateway + load balancer
- GENEVE protocol (port 6081)
- Use case: third-party appliances (firewalls, IDS/IPS)
- Target groups: EC2, IP addresses

## Key Features
- **Cross-zone load balancing** — distributes evenly across all AZs
  - ALB: always on (free)
  - NLB/GLB: disabled by default (charges apply)
- **Health checks** — /health endpoint, threshold, interval
- **SSL/TLS termination** — decrypt at LB, re-encrypt optionally
- **Connection draining (deregistration delay)** — time to complete in-flight requests

## Sticky Sessions
- ALB: application-based or duration-based cookies
- NLB: based on source IP (no cookies)
- Use case: stateful applications

## Exam Tips
- ALB for HTTP/HTTPS routing decisions
- NLB for extreme performance, static IPs, non-HTTP protocols
- GLB for inline security appliances
- ALB + Auto Scaling = most common HA pattern
- NLB can front an ALB (for static IP + Layer 7 features)
- ALB supports SNI (multiple SSL certs)

---

Related: [[EC2 Auto Scaling]] | [[Domain 1 - Secure Architectures/Certificate Manager]]
#aws #elb #networking #domain2

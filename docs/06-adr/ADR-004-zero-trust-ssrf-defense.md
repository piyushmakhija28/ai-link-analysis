# ADR-004: Multi-Stage Zero-Trust SSRF Defense Pipeline

## Status
Accepted

## Context
Allowing an API to accept user-supplied URLs and fetch content on the server creates a severe Server-Side Request Forgery (SSRF) vulnerability. An attacker can target:
- Cloud instance metadata services (AWS/GCP/Azure `169.254.169.254` to steal IAM credentials).
- Internal loopback services (`127.0.0.1`, `localhost` on internal ports 8000, 6379, 5432).
- Internal private VPC subnets (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`).
- DNS rebinding attacks (where a public domain resolves to a private IP).

## Decision
We implement a **7-Stage Zero-Trust SSRF Defense Pipeline**:
1. **Scheme Validation:** Strictly allow `http` and `https`; reject `file://`, `gopher://`, `ftp://`.
2. **Hostname String Check:** Reject loopback strings (`localhost`, `127.0.0.1`, `::1`).
3. **DNS Pre-Flight Resolution:** Resolve target domain using `socket.getaddrinfo()` to inspect all IPv4 and IPv6 target IPs.
4. **IP Subnet Blacklisting:** Check all resolved IPs against `ipaddress.ip_network` blacklist:
   - Loopback (`127.0.0.0/8`, `::1`)
   - Private RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`)
   - Link-Local RFC 3927 (`169.254.0.0/16`, `fe80::/10`)
   - Carrier-Grade NAT (`100.64.0.0/10`)
   - Multicast/Broadcast (`224.0.0.0/4`, `255.255.255.255/32`)
5. **Port Whitelisting:** Strictly permit ports `80` and `443`.
6. **DNS Rebinding Guard:** Ensure all resolved socket addresses pass checks before connecting.
7. **Streaming Size Limit:** Cap responses at 5MB.

## Consequences
- **Positive:**
  - Comprehensive defense against OWASP API Security Top 10 SSRF vulnerabilities.
  - Cloud metadata credentials completely inaccessible to attackers.
- **Negative:**
  - Adds ~10-20ms of DNS pre-flight resolution overhead per un-cached request.

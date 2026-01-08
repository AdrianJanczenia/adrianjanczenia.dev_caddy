# Adrian Janczenia - Caddy

This repository contains the configuration for **Caddy**, which serves as the edge server and reverse proxy for the portfolio microservices ecosystem.

## Role in the Ecosystem

Caddy is the first point of contact for all external traffic. Its primary responsibilities include:

- **Reverse Proxy**: Forwarding incoming requests to the `frontend-service` container.
- **SSL Termination**: Automatically provisioning and renewing TLS certificates via Let's Encrypt.
- **Security Hardening**: Injecting essential security headers to protect against common web vulnerabilities.
- **Static Asset Caching**: Optimizing delivery of CSS, JS, and image files through aggressive cache control.
- **Traffic Encoding**: Compressing responses using modern algorithms like Zstandard and Gzip to reduce bandwidth usage.

## Configuration Details

### Security Policy
The configuration implements a strict Content Security Policy (CSP) and other protective headers:
- **HSTS**: Enforces HTTPS for one year.
- **X-Frame-Options**: Prevents clickjacking by denying framing.
- **CSP**: Restricts source domains for scripts, styles, and fonts to ensure integrity.

### Static Asset Handling
Specialized block for static files (`.ico`, `.css`, `.js`, `.svg`, etc.) sets a `Cache-Control` header with a 60-day expiration period to improve performance for returning visitors.

## Environment Variables

The `Caddyfile` utilizes the following environment variables for flexibility:

| Variable | Description |
|----------|-------------|
| DOMAIN_NAME | The primary domain name (e.g., adrianjanczenia.dev). |

## Deployment

The proxy is containerized using the official Caddy Alpine image for a minimal footprint.

### Build Image
docker build -t caddy-proxy .

### Log Management
Access logs are automatically rolled when they reach 10MB, keeping the last 5 rotations to prevent disk exhaustion.

---
Adrian Janczenia
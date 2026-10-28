# Caddy configuration

This repository packages the portfolio's Caddy configuration. The `Caddyfile` listens on port 80, redirects `adrianjanczenia.dev` to `https://www.adrianjanczenia.dev`, and forwards requests for `www.adrianjanczenia.dev` to `frontend-service:8080`. Other host names receive a 404 response. That upstream name must resolve in the container network where Caddy runs.

## Response handling

- Compresses responses with Zstandard or gzip where supported.
- Adds `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, and `Strict-Transport-Security: max-age=31536000`. It supplies `Referrer-Policy: strict-origin-when-cross-origin` only if the upstream did not set one.
- Sets `Cache-Control: public, max-age=300` for the static extensions matched in the `Caddyfile`, including locally hosted WOFF2 fonts. The short lifetime supports updates to assets that keep the same URL.
- Rotates access logs at 10 MB and keeps five old files. The log filter replaces the `token` query parameter value and omits the `Referer` request header.

The `Dockerfile` packages the configuration with Caddy. To check a local change with a Caddy installation, run:

```sh
caddy validate --config Caddyfile --adapter caddyfile
```

# Caddy reverse proxy

This repository packages the portfolio's Caddy configuration. The `Caddyfile` listens on port 80, redirects `adrianjanczenia.dev` to `https://www.adrianjanczenia.dev`, and forwards requests for `www.adrianjanczenia.dev` to `frontend-service:8080`. Other host names receive a 404 response. That upstream name must resolve in the container network where Caddy runs.

## Response handling

- Compresses responses with Zstandard or gzip where supported.
- Adds `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, and `Strict-Transport-Security: max-age=31536000`. It supplies `Referrer-Policy: strict-origin-when-cross-origin` only if the upstream did not set one.
- Sets `Cache-Control: public, max-age=300` for the static extensions matched in the `Caddyfile`, including locally hosted WOFF2 fonts. The short lifetime supports updates to assets that keep the same URL.
- Rotates access logs at midnight and noon according to the container clock, or when they reach 10 MB. It keeps at most 50 rolled files and removes them at the next rotation once they are older than six days. With twice-daily rotation and cleanup, this limits individual log entries to about seven days while Caddy is running. The log filter replaces the `token` query parameter value and omits the `Referer` request header.

## Run and verify

The `Dockerfile` copies the `Caddyfile` into `caddy:2.11.7-alpine`, which supports time-based log rotation. It expects a Docker network where `frontend-service` resolves to the running front-end container on port 8080. The site block listens on **HTTP port 80**; TLS must be handled before traffic reaches this container. It does not request or manage a certificate.

Validate the configuration with a local Caddy installation:

```sh
caddy validate --config Caddyfile --adapter caddyfile
```

To build the image, run `docker build -t portfolio-caddy .` from this directory. Run it on the same network as the front end, publish its port 80 as appropriate for that environment, and supply a writable `/var/log/caddy` if access logs must survive container replacement. With the proxy reachable on local port 8080, the host routing can be checked without DNS changes:

```sh
curl -i -H 'Host: www.adrianjanczenia.dev' http://127.0.0.1:8080/
```

That request needs a running front end. The bare domain returns a permanent redirect to the HTTPS `www` URL; other host names return 404.

Changes to Caddy's file log options require a Caddy process restart; reloading the Caddyfile alone is insufficient. The production Compose file in `adrianjanczenia.dev_tools` references a tagged Caddy image without a `build` section, so that image must be built separately and made available to the deployment host before the Caddy container is recreated.

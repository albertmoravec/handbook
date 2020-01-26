# Traefik (v2)

## Setting up HTTP to HTTPS redirect

For the redirect to work a global middleware needs to be set up.

It's important to set `traefik.enable=true` on the traefik container itself if you're enabling traefik per-container, otherwise configuration flags are ignored.

```
image: traefik:v2.0
labels:
  # Global HTTPS redirect
  - traefik.enable=true
  - traefik.http.routers.https-redirect.entrypoints=web
  - traefik.http.routers.https-redirect.rule=HostRegexp(`{any:.*}`)
  - traefik.http.routers.https-redirect.middlewares=https-redirect
  - traefik.http.middlewares.https-redirect.redirectscheme.scheme=https
  - traefik.http.middlewares.https-redirect.redirectscheme.port=443
  - traefik.http.middlewares.https-redirect.redirectscheme.permanent=true
```

## Setting up Let's Encrypt


**Traefik configuration:**  
The commented-out `caServer` option points to ACME staging environment which doesn't rate-limit in case of misconfiguration.
```
image: traefik:v2.0
command:
  - --entrypoints.websecure.address=:443
  - --certificatesresolvers.le.acme.email=albert.moravec@keenmate.com
  - --certificatesresolvers.le.acme.storage=/storage/acme.json
  - --certificatesresolvers.le.acme.tlschallenge=true
# - --certificatesResolvers.le.acme.caServer=https://acme-staging-v02.api.letsencrypt.org/directory
```

**Container configuration:**
```
labels:
  - traefik.http.routers.into-phoenix.tls=true
  - traefik.http.routers.into-phoenix.tls.certresolver=le
  - traefik.http.routers.into-phoenix.entrypoints=websecure
```
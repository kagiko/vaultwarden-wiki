Dockerfile (Caddy Builder)

```nginx
FROM caddy:builder AS builder
RUN xcaddy build --with github.com/caddy-dns/cloudflare

FROM caddy:latest
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

build command

```bash
docker build -t [YOUR-NAME]/caddycfdns .
```

Caddyfile (as reverse proxy)
```nginx
https://[YOUR-DOMAIN]:443 {

  tls {
        dns cloudflare [API-KEY]
  }

  encode gzip

  header / {
       # Enable HTTP Strict Transport Security (HSTS)
       Strict-Transport-Security "max-age=31536000;"
       # Enable cross-site filter (XSS) and tell browser to block detected attacks
       X-XSS-Protection "0"
       # Disallow the site to be rendered within a frame (clickjacking protection)
       X-Frame-Options "DENY"
       # Prevent search engines from indexing (optional)
       X-Robots-Tag "noindex, nofollow"
       # Disallow sniffing of X-Content-Type-Options
       X-Content-Type-Options "nosniff"
       # Server name removing
       -Server
       # Remove X-Powered-By though this shouldn't be an issue, better opsec to remove
       -X-Powered-By
       # Remove Last-Modified because etag is the same and is as effective
       -Last-Modified
   }
  # Proxy to Rocket
  reverse_proxy vaultwarden:80 {
       # Send the true remote IP to Rocket, so that vaultwarden can put this in the
       # log, so that fail2ban can ban the correct IP.
       header_up X-Real-IP {remote_host}
  }
}
```

docker-compose.yml

```nginx
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server
    restart: always
    volumes:
      - $PWD/vw-data:/data
    environment:
      SIGNUPS_ALLOWED: 'false'   # set to false to disable signups
      DOMAIN: 'https://[DOMAIN]'
      SMTP_HOST: '[MAIL-SERVER]'
      SMTP_FROM: '[E-MAIL]'
      SMTP_PORT: '587'
      SMTP_SSL: 'true'
      SMTP_USERNAME: '[E-MAIL]'
      SMTP_PASSWORD: '[SMTP-PASS]'
#      ADMIN_TOKEN: '[RAND. GENERATE]'
#      YUBICO_CLIENT_ID: '[OPTIONAL]'
#      YUBICO_SECRET_KEY: '[OPTIONAL]'

  caddy:
    image: [YOUR-NAME]/caddycfdns
    restart: always
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
      - caddy_log:/logs
    ports:
      - [PRIVATE-IP]:443:443
    environment:
      ACME_AGREE: 'true'
      CLOUDFLARE_EMAIL: '[YOUR-EMAIL]'
      CLOUDFLARE_API_TOKEN: '[YOUR-TOKEN]'
      DOMAIN: '[DOMAIN]'

volumes:
  caddy_data:
  caddy_config:
  caddy_log:
```
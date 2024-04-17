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

  # For the remaining config, see https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples
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
      SMTP_SECURITY: 'starttls'
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
Docker Compose is a tool that allows the definition and configuration of multi-container applications. In our case, we want both the Bitwarden_RS server and a proxy to redirect the WebSocket requests to the correct place.

This guide is based on [#126 (comment)](https://github.com/dani-garcia/bitwarden_rs/issues/126#issuecomment-417872681).

Create a `docker-compose.yml` file based on this:
```yml
# docker-compose.yml
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server
    restart: always
    volumes:
      - ./bw-data:/data
    environment:
      WEBSOCKET_ENABLED: 'true' # Required to use websockets
      SIGNUPS_ALLOWED: 'true'   # set to false to disable signups

  caddy:
    image: abiosoft/caddy
    restart: always
    volumes:
      - ./Caddyfile:/etc/Caddyfile:ro
      - caddycerts:/root/.caddy
    ports:
      - 80:80 # needed for Let's Encrypt
      - 443:443
    environment:
      ACME_AGREE: 'true'              # agree to Let's Encrypt Subscriber Agreement
      DOMAIN: 'bitwarden.example.org' # CHANGE THIS! Used for Auto Let's Encrypt SSL
      EMAIL: 'bitwarden@example.org'  # CHANGE THIS! Optional, provided to Let's Encrypt

volumes:
  caddycerts:
```

and the corresponding `Caddyfile` (does not need to be modified):
```nginx
# Caddyfile
{$DOMAIN} {
    tls {$EMAIL}
    gzip

    header / {
        # Enable HTTP Strict Transport Security (HSTS)
        Strict-Transport-Security "max-age=31536000;"
        # Enable cross-site filter (XSS) and tell browser to block detected attacks
        X-XSS-Protection "1; mode=block"
        # Disallow the site to be rendered within a frame (clickjacking protection)
        X-Frame-Options "DENY"
        # Prevent search engines from indexing (optional)
        #X-Robots-Tag "none"
    }

    # The negotiation endpoint is also proxied to Rocket
    proxy /notifications/hub/negotiate bitwarden:80 {
        transparent
    }

    # Notifications redirected to the websockets server
    proxy /notifications/hub bitwarden:3012 {
        websocket
    }

    # Proxy the Root directory to Rocket
    proxy / bitwarden:80 {
        transparent
    }
}
```

Run
```bash
docker-compose up -d
```
to create & start the containers. It creates a private network between the two containers for the reverse proxy, only caddy is exposed to the outside.

```bash
docker-compose down
```
stops and destroys the containers.

If there's no need for websocket notifications, you can run Bitwarden_rs alone. Here's my example. Actually I'm running Bitwarden_rs on my Raspberry Pi and I'm using bitwardenrs/server:raspberry image. If you want to do the same, remember to change it to the example.
```yml
# docker-compose.yml
version: '3'

services:
 bitwarden:
  image: bitwardenrs/server
  restart: always
  volumes:
      - ./bw-data:/data
      - ./ssl:/ssl
  ports:
    - 443:80
  environment:
   ROCKET_TLS: '{certs = "/ssl/fullchain.pem", key = "/ssl/key.pem"}'
   LOG_FILE: '/data/bitwarden.log'
   SIGNUPS_ALLOWED: 'true'
```

Even the server is running at the home network behind the NAT, I wanted to have Let's Encrypt's certificate. I followed this guide https://github.com/Neilpang/acme.sh/wiki/DNS-alias-mode. First set domain cname. And with CloudFlare export CF_Key and CF_Email or CF_Token and CF_Account_ID. https://github.com/Neilpang/acme.sh/wiki/dnsapi Then issue a cert. Finally install cert. `acme.sh --install-cert -d home.example.com --key-file /home/pi/ssl/key.pem --fullchain-file /home/pi/ssl/fullchain.pem`
Or simply use `acme.sh --issue -d  home.example.com --challenge-alias otherdomain.com --dns dns_cf --key-file /home/pi/ssl/key.pem --fullchain-file /home/pi/ssl/fullchain.pem`
My domain's A record points to the binded IP on the last line of docker-compose.yml and there are no complaints about certificate.

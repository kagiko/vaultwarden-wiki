[Docker Compose](https://docs.docker.com/compose/) is a tool that allows the definition and configuration of multi-container applications. In our case, we want both the bitwarden_rs server and a proxy to redirect the WebSocket requests to the correct place.

This example assumes that you have [installed](https://docs.docker.com/compose/install/) Docker Compose, that you have a domain name (e.g., `bitwarden.example.com`) for your bitwarden_rs instance, and that it will be publicly accessible.

Start by making a new directory and changing into it. Next, create the `docker-compose.yml` below, making sure to substitute appropriate values for the `DOMAIN` and `EMAIL` variables.

```yaml
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server:latest
    container_name: bitwarden
    restart: always
    environment:
      - WEBSOCKET_ENABLED=true  # Enable WebSocket notifications.
    volumes:
      - ./bw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80  # Needed for the ACME HTTP-01 challenge.
      - 443:443
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      - DOMAIN=bitwarden.example.com  # Your domain.
      - EMAIL=admin@example.com       # The email address to use for ACME registration.
      - LOG_FILE=/data/access.log
```

In the same directory, create the `Caddyfile` below. (This file does not need to be modified.)
```
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # Use the ACME HTTP-01 challenge to get a cert for the configured domain.
  tls {$EMAIL}

  # This setting may have compatibility issues with some browsers
  # (e.g., attachment downloading on Firefox). Try disabling this
  # if you encounter issues.
  encode gzip

  # Notifications redirected to the WebSocket server
  reverse_proxy /notifications/hub bitwarden:3012

  # Proxy everything else to Rocket
  reverse_proxy bitwarden:80 {
       # Send the true remote IP to Rocket, so that bitwarden_rs can put this in the
       # log, so that fail2ban can ban the correct IP.
       header_up X-Real-IP {remote_host}
  }
}
```

Run
```bash
docker-compose up -d
```
to create and start the containers. A private network for the services in this `docker-compose.yml` file will be created automatically, with only Caddy being publicly exposed.

```bash
docker-compose down
```
stops and destroys the containers.

A similar Caddy-based example for Synology is available [here](https://github.com/sosandroid/docker-bitwarden_rs-caddy-synology).

If there's no need for websocket notifications, you can run Bitwarden_rs alone. Here's my example. Actually I'm running Bitwarden_rs on my Raspberry Pi and I'm using bitwardenrs/server image. If you want to do the same, remember to change it to the example.
```yml
# docker-compose.yml
version: '3'

services:
 bitwarden:
  image: bitwardenrs/server:latest
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

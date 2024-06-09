[Docker Compose](https://docs.docker.com/compose/) is a tool that allows the definition and configuration of multi-container applications. In our case, we want both the vaultwarden server and a proxy to redirect the WebSocket requests to the correct place.

## Minimal template for no reverse proxy / a reverse proxy configured by yourself (caddy example below)

This example assumes that you have [installed](https://docs.docker.com/compose/install/) Docker Compose. This configuration can be used either for local servers that are not open to the 'outside world', or as a template for a [reverse proxy](https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples).

Start by creating a new directory at your preferred location and changing into it. Next, create the `compose.yml` (or `docker-compose.yml` for legacy versions)

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      # DOMAIN: "https://vaultwarden.example.com"  # required when using a reverse proxy; your domain; vaultwarden needs to know it's https to work properly with attachments
      SIGNUPS_ALLOWED: "true" # Deactivate this with "false" after you have created your account so that no strangers can register
    volumes:
      - ./vw-data:/data # the path before the : can be changed
    ports:
      - 11001:80 # you can replace the 11001 with your preferred port
```

to create and run the container, run:
```bash
docker compose up -d && docker compose logs -f
```

to update and run the container, run:
```bash
docker compose pull && docker compose up -d && docker compose logs -f
```

## Caddy with HTTP challenge

This example assumes that you have [installed](https://docs.docker.com/compose/install/) Docker Compose, that you have a domain name (e.g., `vaultwarden.example.com`) for your vaultwarden instance, and that it will be publicly accessible.

> [!NOTE]
> Docker Compose might be run as `docker-compose <command> ...` (with a dash) or `docker compose <command> ...` (with a space), depending on how you have installed Docker Compose. `docker-compose` is the original syntax, when Docker Compose was distributed as a standalone executable. You can still choose to do a [standalone](https://docs.docker.com/compose/install/other/#install-compose-standalone) installation, in which case you would continue to use this syntax. However, Docker currently recommends installing Docker Compose as a Docker plugin, where `compose` becomes a subcommand of `docker`, making the syntax `docker compose <command> ...`.

Start by making a new directory and changing into it. Next, create the `compose.yaml` (or `docker-compose.yml` for legacy versions) below, making sure to substitute appropriate values for the `DOMAIN` and `EMAIL` variables.

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      DOMAIN: "https://vaultwarden.example.com"  # Your domain; vaultwarden needs to know it's https to work properly with attachments
      SIGNUPS_ALLOWED: "true"
    volumes:
      - /vw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80  # Needed for the ACME HTTP-01 challenge.
      - 443:443
      - 443:443/udp # Needed for HTTP/3.
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      DOMAIN: "https://vaultwarden.example.com"  # Your domain.
      EMAIL: "admin@example.com"                 # The email address to use for ACME registration.
      LOG_FILE: "/data/access.log"
```

In the same directory, create the `Caddyfile` below. (This file does not need to be modified.)
```
{$DOMAIN} {
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
  encode zstd gzip

  # Proxy everything Rocket
  reverse_proxy vaultwarden:80 {
       # Send the true remote IP to Rocket, so that vaultwarden can put this in the
       # log, so that fail2ban can ban the correct IP.
       header_up X-Real-IP {remote_host}
  }
}
```

Run
```bash
docker compose up -d  # or `docker-compose up -d` if using standalone Docker Compose
```
to create and start the containers. A private network for the services in this `compose.yaml` (or `docker-compose.yml` for legacy versions) file will be created automatically, with only Caddy being publicly exposed.

```bash
docker compose down  # or `docker-compose down` if using standalone Docker Compose
```
stops and destroys the containers.

A similar Caddy-based example for Synology is available [here](https://github.com/sosandroid/docker-bitwarden_rs-caddy-synology).

## Caddy with DNS challenge

This example is the same as the previous one, but for the case where you don't want your instance to be publicly accessible (i.e., you can access it only from your local network). This example uses Duck DNS as the DNS provider. Refer to [[Running a private vaultwarden instance with Let's Encrypt certs|Running-a-private-vaultwarden-instance-with-Let's-Encrypt-certs]] for more background, and details on how to set up Duck DNS.

Start by making a new directory and changing into it. Next, create the `compose.yaml` (or `docker-compose.yml` for legacy versions) below, making sure to substitute appropriate values for the `DOMAIN` and `EMAIL` variables.

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      DOMAIN: "https://vaultwarden.example.com"  # Your domain; vaultwarden needs to know it's https to work properly with attachments
    volumes:
      - /vw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80
      - 443:443
      - 443:443/udp # Needed for HTTP/3.
    volumes:
      - ./caddy:/usr/bin/caddy  # Your custom build of Caddy.
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      DOMAIN: "https://vaultwarden.example.com"  # Your domain.
      EMAIL: "admin@example.com"                 # The email address to use for ACME registration.
      DUCKDNS_TOKEN: "<token>"                   # Your Duck DNS token.
      LOG_FILE: "/data/access.log"
```

The stock Caddy builds (including the one in the Docker image) don't include the DNS challenge modules, so next you'll need to [get a custom Caddy build](https://github.com/dani-garcia/vaultwarden/wiki/Running-a-private-vaultwarden-instance-with-Let%27s-Encrypt-certs#getting-a-custom-caddy-build). Rename the custom build as `caddy` and move it under the same directory as `compose.yaml` (or `docker-compose.yml` for legacy versions). Make sure the `caddy` file is executable (e.g., `chmod a+x caddy`). The `compose.yaml` (or `docker-compose.yml` for legacy versions) file above bind-mounts the custom build into the `caddy:2` container, replacing the stock build.

In the same directory, create the `Caddyfile` below. (This file does not need to be modified.)
```
{$DOMAIN} {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # Use the ACME DNS-01 challenge to get a cert for the configured domain.
  tls {
    dns duckdns {$DUCKDNS_TOKEN}
  }

  # This setting may have compatibility issues with some browsers
  # (e.g., attachment downloading on Firefox). Try disabling this
  # if you encounter issues.
  encode zstd gzip

  # Proxy everything to Rocket
  reverse_proxy vaultwarden:80
}
```

As with the HTTP challenge example, run
```bash
docker compose up -d  # or `docker-compose up -d` if using standalone Docker Compose
```
to create and start the containers.
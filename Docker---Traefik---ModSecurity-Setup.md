Setting up ModSecurity will proxy all requests to Vaultwarden through a Web Application Firewall (WAF). This might help mitigate unknown vulnerabilities in Vaultwarden by filtering suspicious requests (like injection attempts).

# Pre-Reqs
- Setup with Docker + Traefik 2.0 as reverse proxy
- Fail2Ban properly set up ([see this tutorial](https://github.com/dani-garcia/vaultwarden/wiki/Fail2Ban-Setup#debian--ubuntu--raspberry-pi-os))
- This is only tested on Debian (but should work on similar systems like Ubuntu or Raspbian)

# Installation

bash
```bash
touch /opt/docker/waf-rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf && touch /opt/docker/waf-rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
```

`/opt/docker/docker-compose.yml`
```yml
services:
  traefik:
    image: traefik:latest
    container_name: traefik
    command:
      - --providers.docker=true
      - --providers.docker.exposedByDefault=false
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      - --certificatesresolvers.myresolver.acme.tlschallenge=true
      - --certificatesresolvers.myresolver.acme.email=you@domain.tld
      - --certificatesresolvers.myresolver.acme.storage=acme.json
      - --certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /opt/docker/le:/letsencrypt

  waf:
    image: owasp/modsecurity-crs:apache
    container_name: waf
    environment:
      PARANOIA: 1
      ANOMALY_INBOUND: 10
      ANOMALY_OUTBOUND: 5
      PROXY: 1
      REMOTEIP_INT_PROXY: "172.20.0.1/16"
      BACKEND: "http://vaultwarden:80"
      BACKEND_WS: "ws://vaultwarden:80/notifications/hub"
      ERRORLOG: "/var/log/waf/waf.log"
    volumes:
     - /opt/docker/waf:/var/log/waf
     - /opt/docker/waf-rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf:/etc/modsecurity.d/owasp-crs/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
     - /opt/docker/waf-rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf:/etc/modsecurity.d/owasp-crs/rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
    labels:
      - traefik.enable=true
      - traefik.http.middlewares.redirect-https.redirectScheme.scheme=https
      - traefik.http.middlewares.redirect-https.redirectScheme.permanent=true
      - traefik.http.routers.vw-ui-https.rule=Host(`sub.domain.tld`)
      - traefik.http.routers.vw-ui-https.entrypoints=websecure
      - traefik.http.routers.vw-ui-https.tls=true
      - traefik.http.routers.vw-ui-https.service=vw-ui
      - traefik.http.routers.vw-ui-http.rule=Host(`sub.domain.tld`)
      - traefik.http.routers.vw-ui-http.entrypoints=web
      - traefik.http.routers.vw-ui-http.middlewares=redirect-https
      - traefik.http.routers.vw-ui-http.service=vw-ui
      - traefik.http.services.vw-ui.loadbalancer.server.port=80
      - traefik.http.routers.vw-websocket-https.rule=Host(`sub.domain.tld`) && Path(`/notifications/hub`)
      - traefik.http.routers.vw-websocket-https.entrypoints=websecure
      - traefik.http.routers.vw-websocket-https.tls=true
      - traefik.http.routers.vw-websocket-https.service=vw-websocket
      - traefik.http.routers.vw-websocket-http.rule=Host(`sub.domain.tld`) && Path(`/notifications/hub`)
      - traefik.http.routers.vw-websocket-http.entrypoints=web
      - traefik.http.routers.vw-websocket-http.middlewares=redirect-https
      - traefik.http.routers.vw-websocket-http.service=vw-websocket
      - traefik.http.services.vw-websocket.loadbalancer.server.port=3012

  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      WEBSOCKET_ENABLED: "true"
      SENDS_ALLOWED: "true"
      PASSWORD_ITERATIONS: 500000
      SIGNUPS_ALLOWED: "true"
      SIGNUPS_VERIFY: "true"
      SIGNUPS_DOMAINS_WHITELIST: "yourdomain.tld"
      ADMIN_TOKEN: "some random string" #generate with openssl rand
      DOMAIN: "domain host name"
      SMTP_HOST: "smtp server"
      SMTP_FROM: "sender email e.g: you@domain.tld"
      SMTP_FROM_NAME: "sender name"
      SMTP_SECURITY: "starttls"
      SMTP_PORT: 587
      SMTP_USERNAME: "smtp username"
      SMTP_PASSWORD: "smtp password"
      SMTP_TIMEOUT: 15
      LOG_FILE: "/data/vaultwarden.log"
      LOG_LEVEL: "warn"
      EXTENDED_LOGGING: "true"
      TZ: "your time zone"
    volumes:
      - /opt/docker/vaultwarden:/data

networks:
  default:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 172.20.0.1/16
```

/etc/fail2ban/filter.d/waf.conf
```yml
[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*\[client <ADDR>\] ModSecurity: Access denied with code 403 .*$
ignoreregex =
```

/etc/fail2ban/jail.d/waf.conf
```yml
[waf]
enabled = true
port = 80,443
filter = waf
action = iptables-allports[name=waf, chain=FORWARD]
logpath = /opt/docker/waf/waf.log
maxretry = 1
bantime = 14400
findtime = 14400
```

# Note
Integrating Fail2Ban with ModSecurity will slow / deter further exploitation / exploration by adversaries. This is setup to ban on the first ModSecurity intervention. 

In order to increase the aggressivity of ModSecurity increase `PARANOIA` ([read more here](https://coreruleset.org/20211028/working-with-paranoia-levels/)) and or decrease `ANOMALY_INBOUND` ([read more](https://coreruleset.org/docs/concepts/anomaly_scoring/)).

Prepare yourself to tune ModSecurity HEAVILY with PARANOIA >2 in order to get the UI barely working without disabling many rules.

The following files will enable you to tune ModSecurity ([tutorial](https://coreruleset.org/docs/concepts/false_positives_tuning/))
```bash
/opt/docker/waf-rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
/opt/docker/waf-rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
```

☁️ Food for thought ☁️ 

If your data is so sensitive that you are considering `PARANOIA` > 1, then consider hosting Vaultwarden not on a public endpoint and limit access to the host itself via a firewall and grant user access only via a VPN connection. Keep in mind that this does not mitigate insider threats which are often underestimated, so keep that in mind!
In this document, `<SERVER>` refers to the IP or domain where bitwarden_rs is accessible from. If both the proxy and bitwarden_rs are running in the same system, simply use `localhost`.
The ports proxied by default are `80` for the web server and `3012` for the WebSocket server. The proxies are configured to listen in port `443` with HTTPS enabled, which is recommended.

When using a proxy, it's preferrable to configure HTTPS at the proxy level and not at the application level, this way the WebSockets connection is also secured.

## Caddy

```nginx
localhost:443 {
    # The negotiation endpoint is also proxied to Rocket
    proxy /notifications/hub/negotiate <SERVER>:80 {
        transparent
    }
    
    # Notifications redirected to the websockets server
    proxy /notifications/hub <SERVER>:3012 {
        websocket
    }
    
    # Proxy the Root directory to Rocket
    proxy / <SERVER>:80 {
        transparent
    }

    tls ${SSLCERTIFICATE} ${SSLKEY}
    # or 'tls self_signed' to generate a self-signed certificate
}
```
Caddy can also automatically enable HTTPS in some circumstances, check the [docs](https://caddyserver.com/docs/automatic-https).

## Nginx (by shauder)
```nginx
server {
  listen 443 ssl http2;
  server_name vault.*;
  
  # Specify SSL config if using a shared one.
  #include conf.d/ssl/ssl.conf;
  
  # Allow large attachments
  client_max_body_size 128M;

  location / {
    proxy_pass http://<SERVER>:80;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
  
  location /notifications/hub {
    proxy_pass http://<SERVER>:3012;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
  
  location /notifications/hub/negotiate {
    proxy_pass http://<SERVER>:80;
  }

  # Optionally add extra authentication besides the AUTH_TOKEN
  # If you don't want this, leave this part out
  location /admin {
    # See: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
    auth_basic "Private";
    auth_basic_user_file /path/to/htpasswd_file;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_pass http://<SERVER>:80;
  }

}
```

## Apache (by fbartels)
```apache
<VirtualHost *:443>
    SSLEngine on
    ServerName bitwarden.$hostname.$domainname

    SSLCertificateFile ${SSLCERTIFICATE}
    SSLCertificateKeyFile ${SSLKEY}
    SSLCACertificateFile ${SSLCA}
    ${SSLCHAIN}

    ErrorLog \${APACHE_LOG_DIR}/bitwarden-error.log
    CustomLog \${APACHE_LOG_DIR}/bitwarden-access.log combined

    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /notifications/hub(.*) ws://<SERVER>:3012/$1 [P,L]
    ProxyPass / http://<SERVER>:80/

    ProxyPreserveHost On
    ProxyRequests Off
    RequestHeader set X-Real-IP %{REMOTE_ADDR}s
</VirtualHost>
```

## Traefik v1 (docker-compose example)
```yaml
labels:
    - traefik.enable=true
    - traefik.docker.network=traefik
    - traefik.web.frontend.rule=Host:bitwarden.domain.tld
    - traefik.web.port=80
    - traefik.hub.frontend.rule=Host:bitwarden.domain.tld;Path:/notifications/hub
    - traefik.hub.port=3012
    - traefik.hub.protocol=ws
```

## Traefik v2 (docker-compose example by hwwilliams)
#### Traefik v1 labels migrated to Traefik v2
```yaml
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.routers.bitwarden-ui.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-ui.service=bitwarden-ui
  - traefik.http.services.bitwarden-ui.loadbalancer.server.port=80
  - traefik.http.routers.bitwarden-websocket.rule=Host(`bitwarden.domain.tld`) && Path(`/notifications/hub`)
  - traefik.http.routers.bitwarden-websocket.service=bitwarden-websocket
  - traefik.http.services.bitwarden-websocket.loadbalancer.server.port=3012
```

#### Migrated labels plus HTTP to HTTPS redirect
These labels assume that the entrypoints defined in Traefik for port 80 and 443 are 'web' and 'websecure' respectively.

These labels also assume you already have a default certificates resolver defined in Traefik.
```yaml
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.middlewares.redirect-https.redirectScheme.scheme=https
  - traefik.http.middlewares.redirect-https.redirectScheme.permanent=true
  - traefik.http.routers.bitwarden-ui-https.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-ui-https.entrypoints=websecure
  - traefik.http.routers.bitwarden-ui-https.tls=true
  - traefik.http.routers.bitwarden-ui-https.service=bitwarden-ui
  - traefik.http.routers.bitwarden-ui-http.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-ui-http.entrypoints=web
  - traefik.http.routers.bitwarden-ui-http.middlewares=redirect-https
  - traefik.http.routers.bitwarden-ui-http.service=bitwarden-ui
  - traefik.http.services.bitwarden-ui.loadbalancer.server.port=80
  - traefik.http.routers.bitwarden-websocket-https.rule=Host(`bitwarden.domain.tld`) && Path(`/notifications/hub`)
  - traefik.http.routers.bitwarden-websocket-https.entrypoints=websecure
  - traefik.http.routers.bitwarden-websocket-https.tls=true
  - traefik.http.routers.bitwarden-websocket-https.service=bitwarden-websocket
  - traefik.http.routers.bitwarden-websocket-http.rule=Host(`bitwarden.domain.tld`) && Path(`/notifications/hub`)
  - traefik.http.routers.bitwarden-websocket-http.entrypoints=web
  - traefik.http.routers.bitwarden-websocket-http.middlewares=redirect-https
  - traefik.http.routers.bitwarden-websocket-http.service=bitwarden-websocket
  - traefik.http.services.bitwarden-websocket.loadbalancer.server.port=3012
```
In this document, `<SERVER>` refers to the IP or domain where bitwarden_rs is accessible from. If both the proxy and bitwarden_rs are running in the same system, simply use `localhost`.
The ports proxied by default are `80` for the web server and `3012` for the WebSocket server. The proxies are configured to listen in port `443` with HTTPS enabled, which is recommended.

When using a proxy, it's preferrable to configure HTTPS at the proxy level and not at the application level, this way the WebSockets connection is also secured.

<details>
<summary>Caddy 1.x</summary><br/>

Caddy can also automatically enable HTTPS in some circumstances, check the [docs](https://caddyserver.com/v1/docs/automatic-https).
```nginx
:443 {
  tls ${SSLCERTIFICATE} ${SSLKEY}
  # or 'tls self_signed' to generate a self-signed certificate

  # This setting may have compatibility issues with some browsers
  # (e.g., attachment downloading on Firefox). Try disabling this
  # if you encounter issues.
  gzip

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
}
```
</details>

<details>
<summary>Caddy 2.x</summary><br/>

Caddy 2 can also automatically enable HTTPS in some circumstances, check the [docs](https://caddyserver.com/docs/automatic-https).
```nginx
# Caddyfile V2.0 config file
:80 {
  #Caddy on port 80 in container to bitwarden_rs private instance
  #Use it if Caddy behind another reverse proxy such as the one embedded on Synology
  log {
	output file {env.LOG_FILE}
	level INFO
	#roll_size 5MiB #Not working on Caddy V2.0.0 Beta20 https://caddyserver.com/docs/caddyfile/directives/log#log
	#roll_keep 2 #Not working on Caddy V2.0.0 Beta20 https://caddyserver.com/docs/caddyfile/directives/log#log
  }

  # This setting may have compatibility issues with some browsers
  # (e.g., attachment downloading on Firefox). Try disabling this
  # if you encounter issues.
  encode gzip

  header {
       # Enable cross-site filter (XSS) and tell browser to block detected attacks
       X-XSS-Protection "1; mode=block"
       # Disallow the site to be rendered within a frame (clickjacking protection)
       X-Frame-Options "DENY"
       # Prevent search engines from indexing (optional)
       X-Robots-Tag "none"
       # Server name removing
       -Server
   }

  # The negotiation endpoint is also proxied to Rocket
  reverse_proxy /notifications/hub/negotiate <SERVER>:80

  # Notifications redirected to the websockets server
  reverse_proxy /notifications/hub <SERVER>:3012

  # Proxy the Root directory to Rocket
  reverse_proxy <SERVER>:80
}

#{env.DOMAIN}:443 {
#  #Caddy on port 443 in container to bitwarden_rs private instance 
#  #Use it if Caddy exposed to the net 
#
#  log {
#	output file {env.LOG_FILE}
#	level INFO
#   #roll_size 5MiB #Not working on Caddy V2.0.0 Beta20 https://caddyserver.com/docs/caddyfile/directives/log#log
#   #rool_keep 30 #Not working on Caddy V2.0.0 Beta20 https://caddyserver.com/docs/caddyfile/directives/log#log
#  }
#
#  # Uncomment only one of the 2 lines. Depending if you provide your own cert or request one from Let's Encrypt
#  tls {env.SSLCERTIFICATE} {env.SSLKEY}
#  tls {env.EMAIL}
#
#  encode gzip
#
#  header / {
#       # Enable HTTP Strict Transport Security (HSTS)
#       Strict-Transport-Security "max-age=31536000;"
#       # Enable cross-site filter (XSS) and tell browser to block detected attacks
#       X-XSS-Protection "1; mode=block"
#       # Disallow the site to be rendered within a frame (clickjacking protection)
#       X-Frame-Options "DENY"
#       # Prevent search engines from indexing (optional)
#       X-Robots-Tag "none"
#       # Server name removing
#       -Server
#   }
#  # The negotiation endpoint is also proxied to Rocket
#  reverse_proxy /notifications/hub/negotiate <SERVER>:80
#
#  # Notifications redirected to the websockets server
#  reverse_proxy /notifications/hub <SERVER>:3012
#
#  # Proxy the Root directory to Rocket
#  reverse_proxy <SERVER>:80
#}
```
</details>

<details>
<summary>Nginx (by shauder)</summary><br/>

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
</details>

<details>
<summary>Nginx (by ypid)</summary><br/>

Ansible inventory example that uses DebOps to configure nginx as a reverse proxy for bitwarden_rs. I choose to go with the PSK in the URL for additional security to not expose the API to everyone on the Internet because the client apps do not support client certificates yet (I tested it). Note: Using subpath/PSK requires to patch the source code and recompile, ref: https://github.com/dani-garcia/bitwarden_rs/issues/241#issuecomment-436376497. /admin is untested. For general discussion about subpath hosting for security refer to: https://github.com/debops/debops/issues/1233

```YAML
bitwarden__fqdn: 'vault.example.org'

nginx__upstreams:

  - name: 'bitwarden'
    type: 'default'
    enabled: True
    server: 'localhost:8000'

nginx__servers:

  - name: '{{ bitwarden__fqdn }}'
    filename: 'debops.bitwarden'
    by_role: 'debops.bitwarden'
    favicon: False
    root: '/usr/share/bitwarden_rs/web-vault'

    location_list:

      - pattern: '/'
        options: |-
          deny all;

      - pattern: '= /ekkP9wtJ_psk_changeme_Hr9CCTud'
        options: |-
          return 307 $scheme://$host$request_uri/;

      ## All the security HTTP headers would then need to be set by nginx as well.
      # - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/'
      #   options: |-
      #     alias /usr/share/bitwarden_rs/web-vault/;

      - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/'
        options: |-
          proxy_set_header Host              $host;
          # proxy_set_header X-Real-IP         $remote_addr;
          # proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port  443;

          proxy_pass http://bitwarden;

      ## Do not use the icons features as long as it reveals all domains from
      ## our credentials to the server.
      - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/icons/'
        options: |-
          access_log off;
          log_not_found off;
          deny all;
```
</details>

<details>
<summary>Nginx (NixOS)(by tklitschi)</summary><br/>

Example NixSO nginx config. For more Information about NixOS Deployement see [depluyement Wiki page](https://github.com/dani-garcia/bitwarden_rs/wiki/Deployment-examples).


```nix
{ config, ... }:
{
  security.acme.acceptTerms = true;
  security.acme.email = "me@example.com";
  security.acme.certs = {

    "bw.example.com" = {
      group = "bitwarden_rs";
      keyType = "rsa2048";
      allowKeysForGroup = true;
    };
  };

  services.nginx = {
    enable = true;

    recommendedGzipSettings = true;
    recommendedOptimisation = true;
    recommendedProxySettings = true;
    recommendedTlsSettings = true;

    virtualHosts = {
      "bw.example.com" = {
        forceSSL = true;
        enableACME = true;
        locations."/" = {
          proxyPass = "http://localhost:8812"; #changed the default rocket port due to some conflict
          proxyWebsockets = true;
        };
        locations."/notifications/hub" = {
          proxyPass = "http://localhost:3012";
          proxyWebsockets = true;
        };
        locations."/notifications/hub/negotiate" = {
          proxyPass = "http://localhost:8812";
          proxyWebsockets = true;
        };
      };
    };
  };
}

```
</details>
<details>
<summary>Apache (by fbartels)</summary><br/>

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
</details>

<details>
<summary>Apache in a sub-location (by ss89)</summary><br/>
Ensure you have the websocket proxy module loaded somewhere in your apache config.
It can look something like: 

```
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so`
```

```apache
<VirtualHost *:443>
    SSLEngine on
    ServerName $hostname.$domainname

    SSLCertificateFile ${SSLCERTIFICATE}
    SSLCertificateKeyFile ${SSLKEY}
    SSLCACertificateFile ${SSLCA}
    ${SSLCHAIN}

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined

    <Location /bitwarden> #adjust here if necessary
        RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule /notifications/hub(.*) ws://<SERVER>:3012/$1 [P,L]
        ProxyPass http://<SERVER>:80/

        ProxyPreserveHost On
        RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    </Location>
</VirtualHost>
```
</details>

<details>
<summary>Traefik v1 (docker-compose example)</summary><br/>

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
</details>

<details>
<summary>Traefik v2 (docker-compose example by hwwilliams)</summary><br/>

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
</details>
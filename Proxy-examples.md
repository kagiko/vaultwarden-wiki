In this document, `<SERVER>` refers to the IP or domain where you access bitwarden_rs. If both the reverse proxy and bitwarden_rs are running on the same system, simply use `localhost`.

By default, bitwarden_rs listens on port 80 for web (REST API) traffic and on port 3012 for WebSocket traffic (if [[WebSocket notifications|Enabling-WebSocket-notifications]] are enabled). The reverse proxy should be configured to terminate SSL/TLS connections (preferably on port 443, the standard port for HTTPS). The reverse proxy then passes incoming client requests to bitwarden_rs on port 80 or 3012 as appropriate, and upon receiving a response from bitwarden_rs, passes that response back to the client.

Note that when you put bitwarden_rs behind a reverse proxy, the connections between the reverse proxy and bitwarden_rs are typically assumed to be going through a secure private network, and thus do not need to be encrypted. The examples below assume you are running in this configuration, in which case you should not enable the HTTPS functionality built into bitwarden_rs (i.e., you should not set the `ROCKET_TLS` environment variable). If you do, connections will fail since the reverse proxy is using HTTP to connect to bitwarden_rs, but you're configuring bitwarden_rs to expect HTTPS.

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

Caddy 2 can automatically enable HTTPS in some circumstances, check the [docs](https://caddyserver.com/docs/automatic-https).

In the Caddyfile syntax, `{$VAR}` denotes the value of the environment variable `VAR`.
If you prefer, you can also directly specify a value instead of substituting an env var value.

```
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # Uncomment this if you want to get a cert via ACME (Let's Encrypt or ZeroSSL).
  # tls {$EMAIL}

  # Or uncomment this if you're providing your own cert. You would also use this option
  # if you're running behind Cloudflare.
  # tls {$SSL_CERT_PATH} {$SSL_KEY_PATH}

  # This setting may have compatibility issues with some browsers
  # (e.g., attachment downloading on Firefox). Try disabling this
  # if you encounter issues.
  encode gzip

  header / {
       # Enable HTTP Strict Transport Security (HSTS)
       Strict-Transport-Security "max-age=31536000;"
       # Enable cross-site filter (XSS) and tell browser to block detected attacks
       X-XSS-Protection "1; mode=block"
       # Disallow the site to be rendered within a frame (clickjacking protection)
       X-Frame-Options "DENY"
       # Prevent search engines from indexing (optional)
       X-Robots-Tag "none"
       # Server name removing
       -Server
   }

  # Notifications redirected to the websockets server
  reverse_proxy /notifications/hub <SERVER>:3012

  # Proxy everything else to Rocket
  reverse_proxy <SERVER>:80 {
       # Send the true remote IP to Rocket, so that bitwarden_rs can put this in the
       # log, so that fail2ban can ban the correct IP.
       header_up X-Real-IP {remote_host}
  }
}
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

  # Optionally add extra authentication besides the ADMIN_TOKEN
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
<summary>Nginx with sub-path (by BlackDex)</summary><br/>

In this example bitwarden_rs will be available via https://bitwarden.example.tld/vault/<br/>
If you want to use any other sub-path, like `bitwarden` or `secret-vault` you should change `/vault/` in the example below to match.<br/>
<br/>
For this to work you need to configure your `DOMAIN` variable to match so it should look like:

```ini
; Add the sub-path! Else this will not work!
DOMAIN=https://bitwarden.example.tld/vault/
```

```nginx
# Define the server IP and ports here.
upstream bitwardenrs-default { server 127.0.0.1:8080; }
upstream bitwardenrs-ws { server 127.0.0.1:3012; }

# Redirect HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name bitwardenrs.example.tld;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name bitwardenrs.example.tld;

    # Specify SSL Config when needed
    #ssl_certificate /path/to/certificate/letsencrypt/live/bitwardenrs.example.tld/fullchain.pem;
    #ssl_certificate_key /path/to/certificate/letsencrypt/live/bitwardenrs.example.tld/privkey.pem;
    #ssl_trusted_certificate /path/to/certificate/letsencrypt/live/bitwardenrs.example.tld/fullchain.pem;

    client_max_body_size 128M;

    ## Using a Sub Path Config
    # Path to the root of your installation
    location /vault/ {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://bitwardenrs-default;
    }

    location /vault/notifications/hub/negotiate {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://bitwardenrs-default;
    }

    location /vault/notifications/hub {
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $http_connection;
      proxy_set_header X-Real-IP $remote_addr;

      proxy_pass http://bitwardenrs-ws;
    }

    # Optionally add extra authentication besides the ADMIN_TOKEN
    # If you don't want this, leave this part out
    location ^~ /vault/admin {
      # See: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
      auth_basic "Private";
      auth_basic_user_file /path/to/htpasswd_file;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://bitwardenrs-default;
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

Example NixSO nginx config. For more Information about NixOS Deployment see [Deployment Wiki page](https://github.com/dani-garcia/bitwarden_rs/wiki/Deployment-examples).


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

Remember to enable `mod_proxy_wstunnel` and `mod_proxy_http`, for example with: `a2enmod proxy_wstunnel` and `a2enmod proxy_http`.
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

On some OS's you can use a2enmod, for example with: `a2enmod proxy_wstunnel` and `a2enmod proxy_http`.

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

<details>
<summary>HAproxy (by BlackDex)</summary><br/>

Add these lines to your haproxy configuration. 

```haproxy
frontend bitwarden_rs
    bind 0.0.0.0:80
    option forwardfor header X-Real-IP
    http-request set-header X-Real-IP %[src]
    default_backend bitwarden_rs_http
    use_backend bitwarden_rs_ws if { path_beg /notifications/hub } !{ path_beg /notifications/hub/negotiate }

backend bitwarden_rs_http
    # Enable compression if you want
    # compression algo gzip
    # compression type text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript
    server bwrshttp 0.0.0.0:8080

backend bitwarden_rs_ws
    server bwrsws 0.0.0.0:3012
```
</details>


<details>
<summary>HAproxy (by <a href="https://github.com/williamdes" target="_blank">@williamdes</a>)</summary><br/>

Add these lines to your HAproxy configuration. 

```haproxy
backend static-success-default
  mode http
  errorfile 503 /usr/local/etc/haproxy/static/index.static.default.html
  errorfile 200 /usr/local/etc/haproxy/static/index.static.default.html

frontend http-in
    bind *:80
    bind *:443 ssl crt /acme.sh/domain.tld/domain.tld.pem
    option forwardfor header X-Real-IP
    http-request set-header X-Real-IP %[src]
    default_backend static-success-default

    # Define hosts
    acl host_bitwarden_domain_tld hdr_dom(Host) -i bitwarden.domain.tld

    ## figure out which one to use
    use_backend bitwarden_rs_http if host_bitwarden_domain_tld !{ path_beg /notifications/hub } or { path_beg /notifications/hub/negotiate }
    use_backend bitwarden_rs_ws if host_bitwarden_domain_tld { path_beg /notifications/hub } !{ path_beg /notifications/hub/negotiate }

backend bitwarden_rs_http
    # Enable compression if you want
    # compression algo gzip
    # compression type text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript
    # You can use the container hostname if you are using haproxy with docker-compose
    server bwrs_http 0.0.0.0:8080

backend bitwarden_rs_ws
    # You can use the container hostname if you are using haproxy with docker-compose
    server bwrs_ws 0.0.0.0:3012
```
</details>
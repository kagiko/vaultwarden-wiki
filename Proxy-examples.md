In this document, `<SERVER>` refers to the IP or domain where you access Vaultwarden. If both the reverse proxy and Vaultwarden are running on the same system, simply use `localhost`.

By default, Vaultwarden listens on port 80 for web (REST API) traffic and [[WebSocket traffic|Enabling-WebSocket-notifications]].
The reverse proxy should be configured to terminate SSL/TLS connections (preferably on port 443, the standard port for HTTPS). The reverse proxy then passes incoming client requests to Vaultwarden on port 80 or the port on which you configured Vaultwarden to listen on, and upon receiving a response from Vaultwarden, passes that response back to the client.

Note that when you put Vaultwarden behind a reverse proxy, the connections between the reverse proxy and Vaultwarden are typically assumed to be going through a secure private network, and thus do not need to be encrypted. The examples below assume you are running in this configuration, in which case you should not enable the HTTPS functionality built into Vaultwarden (i.e., you should not set the `ROCKET_TLS` environment variable). If you do, connections will fail since the reverse proxy is using HTTP to connect to Vaultwarden, but you're configuring Vaultwarden to expect HTTPS.

It's common to use [Docker Compose](https://docs.docker.com/compose/) to link containerized services together (e.g., Vaultwarden and a reverse proxy). See [[Using Docker Compose|Using-Docker-Compose]] for an example of this.

Secure TLS protocol and cipher configurations for webservers can be generated using Mozilla's [SSL Configuration Generator](https://ssl-config.mozilla.org/). All supported browsers and the Mobile apps are known to work with the "Modern" configuration.

<details>
<summary>Caddy 2.x</summary><br/>

Caddy 2 will automatically enable HTTPS in most circumstances, check the [docs](https://caddyserver.com/docs/automatic-https#activation).

In the Caddyfile syntax, `{$VAR}` denotes the value of the environment variable `VAR`.
If you prefer, you can also directly specify a value instead of substituting an env var value.

```

# Uncomment this in addition with the import admin_redir statement allow access to the admin interface only from local networks
# (admin_redir) {
#        @admin {
#                path /admin*
#                not remote_ip private_ranges
#        }
#        redir @admin /
# }

{$DOMAIN} {
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
  encode zstd gzip

  # Uncomment to improve security (WARNING: only use if you understand the implications!)
  # If you want to use FIDO2 WebAuthn, set X-Frame-Options to "SAMEORIGIN" or the Browser will block those requests
  # header / {
  #	# Enable HTTP Strict Transport Security (HSTS)
  #	Strict-Transport-Security "max-age=31536000;"
  #	# Disable cross-site filter (XSS)
  #	X-XSS-Protection "0"
  #	# Disallow the site to be rendered within a frame (clickjacking protection)
  #	X-Frame-Options "DENY"
  #	# Prevent search engines from indexing (optional)
  #	X-Robots-Tag "noindex, nofollow"
  #	# Disallow sniffing of X-Content-Type-Options
  #	X-Content-Type-Options "nosniff"
  #	# Server name removing
  #	-Server
  #	# Remove X-Powered-By though this shouldn't be an issue, better opsec to remove
  #	-X-Powered-By
  #	# Remove Last-Modified because etag is the same and is as effective
  #	-Last-Modified
  # }

  # Uncomment to allow access to the admin interface only from local networks
  # import admin_redir

  # Proxy everything to Rocket
  # if located at a sub-path the reverse_proxy line will look like:
  #   reverse_proxy /subpath/* <SERVER>:80
  reverse_proxy <SERVER>:80 {
       # Send the true remote IP to Rocket, so that Vaultwarden can put this in the
       # log, so that fail2ban can ban the correct IP.
       header_up X-Real-IP {remote_host}
       # If you use Cloudflare proxying, replace remote_host with http.request.header.Cf-Connecting-Ip
       # See https://developers.cloudflare.com/support/troubleshooting/restoring-visitor-ips/restoring-original-visitor-ips/
       # and https://caddy.community/t/forward-auth-copy-headers-value-not-replaced/16998/4
  }
}
```
</details>

<details>
<summary>lighttpd with sub-path (by FlakyPi)</summary><br/>

In this example Vaultwarden will be available via https://vaultwarden.example.tld/vault/<br/>
If you want to use any other sub-path, like `bitwarden` or `secret-vault` you should change `vault` in the example below to match.<br/>

```lighttpd
server.modules += (
"mod_openssl"
)

$SERVER["socket"] == ":443" {  
    ssl.engine   = "enable"   
    ssl.pemfile  = "/etc/letsencrypt/live/vaultwarden.example.tld/fullchain.pem"
    ssl.privkey  = "/etc/letsencrypt/live/vaultwarden.example.tld/privkey.pem"
}

# Redirect HTTP requests (port 80) to HTTPS (port 443)
$SERVER["socket"] == ":80" {  
        $HTTP["host"] =~ "vaultwarden.example.tld" {  
         url.redirect = ( "^/(.*)" => "https://vaultwarden.example.tld/$1" )  
          server.name                 = "vaultwarden.example.tld"   
        }  
}

server.modules += ( "mod_proxy" )

$HTTP["host"] == "vaultwarden.example.tld" {
    $HTTP["url"] =~ "/vault" {
       proxy.server  = ( "" => ("vaultwarden" => ( "host" => "<SERVER>", "port" => 8080 )))
       proxy.forwarded = ( "for" => 1 )
       proxy.header = (
           "https-remap" => "enable",
           "upgrade" => "enable",
           "connect" => "enable"
       )
    }
}
```
You'll have to set `IP_HEADER` to `X-Forwarded-For` instead of `X-Real-IP` in the Vaultwarden environment.

</details>

<details>
<summary>Nginx (by <a href="https://github.com/BlackDex" target="_blank">@BlackDex</a>)</summary><br/>

```nginx
# The `upstream` directives ensure that you have a http/1.1 connection
# This enables the keepalive option and better performance
#
# Define the server IP and ports here.
upstream vaultwarden-default {
  zone vaultwarden-default 64k;
  server 127.0.0.1:8080;
  keepalive 2;
}

# Needed to support websocket connections
# See: https://nginx.org/en/docs/http/websocket.html
# Instead of "close" as stated in the above link we send an empty value.
# Else all keepalive connections will not work.
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      "";
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name vaultwarden.example.tld;

    if ($host = vaultwarden.example.tld) {
        return 301 https://$host$request_uri;
    }
    return 404;
}

server {
    # For older versions of nginx appened http2 to the listen line after ssl and remove `http2 on`
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name vaultwarden.example.tld;

    # Specify SSL Config when needed
    #ssl_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;
    #ssl_certificate_key /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/privkey.pem;
    #ssl_trusted_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;

    client_max_body_size 525M;

    location / {
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://vaultwarden-default;
    }

    # Optionally add extra authentication besides the ADMIN_TOKEN
    # Remove the comments below `#` and create the htpasswd_file to have it active
    #
    #location /admin {
    #  # See: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
    #  auth_basic "Private";
    #  auth_basic_user_file /path/to/htpasswd_file;
    #
    #  proxy_http_version 1.1;
    #  proxy_set_header Upgrade $http_upgrade;
    #  proxy_set_header Connection $connection_upgrade;
    #
    #  proxy_set_header Host $host;
    #  proxy_set_header X-Real-IP $remote_addr;
    #  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #  proxy_set_header X-Forwarded-Proto $scheme;
    #
    #  proxy_pass http://vaultwarden-default;
    #}
}
```

If you run into 504 Gateway Timeout problems, tell nginx to wait longer for Vaultwarden by adding longer timeouts to the `server {` section, for example:

```nginx
  proxy_connect_timeout       777;
  proxy_send_timeout          777;
  proxy_read_timeout          777;
  send_timeout                777;
```

</details>

<details>
<summary>Nginx with sub-path (by <a href="https://github.com/BlackDex" target="_blank">@BlackDex</a>)</summary><br/>

In this example Vaultwarden will be available via https://bitwarden.example.tld/vault/<br/>
If you want to use any other sub-path, like `bitwarden` or `secret-vault` you should change `/vault/` in the example below to match.<br/>
<br/>
For this to work you need to configure your `DOMAIN` variable to match so it should look like:

```ini
; Add the sub-path! Else this will not work!
DOMAIN=https://vaultwarden.example.tld/vault/
```

```nginx
# The `upstream` directives ensure that you have a http/1.1 connection
# This enables the keepalive option and better performance
#
# Define the server IP and ports here.
upstream vaultwarden-default {
  zone vaultwarden-default 64k;
  server 127.0.0.1:8080;
  keepalive 2;
}

# Needed to support websocket connections
# See: https://nginx.org/en/docs/http/websocket.html
# Instead of "close" as stated in the above link we send an empty value.
# Else all keepalive connections will not work.
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      "";
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name vaultwarden.example.tld;

    if ($host = vaultwarden.example.tld) {
        return 301 https://$host$request_uri;
    }
    return 404;
}

server {
    # For older versions of nginx appened `http2` to the listen line after ssl and remove `http2 on;`
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name vaultwarden.example.tld;

    # Specify SSL Config when needed
    #ssl_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;
    #ssl_certificate_key /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/privkey.pem;
    #ssl_trusted_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;

    client_max_body_size 525M;

    ## Using a Sub Path Config
    # Path to the root of your installation
    # Be sure to DO ADD a trailing /, else you will experience issues 
    # But only for this location, all other locations should NOT add this.
    location /vault/ {
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://vaultwarden-default;
    }

    # Optionally add extra authentication besides the ADMIN_TOKEN
    # Remove the comments below `#` and create the htpasswd_file to have it active
    #
    # DO NOT add a trailing /, else you will experience issues
    #location /vault/admin {
    #  # See: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
    #  auth_basic "Private";
    #  auth_basic_user_file /path/to/htpasswd_file;
    #
    #  proxy_http_version 1.1;
    #  proxy_set_header Upgrade $http_upgrade;
    #  proxy_set_header Connection $connection_upgrade;
    #
    #  proxy_set_header Host $host;
    #  proxy_set_header X-Real-IP $remote_addr;
    #  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #  proxy_set_header X-Forwarded-Proto $scheme;
    #
    #  proxy_pass http://vaultwarden-default;
    #}
}
```
</details>

<details>
<summary>Nginx (NixOS) (by tklitschi, samdoshi)</summary><br/>

Example NixOS nginx config. For more Information about NixOS Deployment see [Deployment Wiki page](https://github.com/dani-garcia/vaultwarden/wiki/Deployment-examples).


```nix
{ config, ... }:
{
  security.acme = {
    defaults = {
      acceptTerms = true;
      email = "me@example.com";
    };
    certs."vaultwarden.example.com".group = "vaultwarden";
  };

  services.nginx = {
    enable = true;

    recommendedGzipSettings = true;
    recommendedOptimisation = true;
    recommendedProxySettings = true;
    recommendedTlsSettings = true;

    virtualHosts = {
      "vaultwarden.example.com" = {
        enableACME = true;
        forceSSL = true;
        locations."/" = {
          proxyPass = "http://localhost:8080";
          proxyWebsockets = true;
        };
      };
    };
  };
}

```
</details>

<details>
<summary>Nginx with proxy_protocol in front (by dionysius)</summary><br/>

In this example there is a downstream proxy communicating in [proxy_protocol in front of this nginx](https://docs.nginx.com/nginx/admin-guide/load-balancer/using-proxy-protocol/) (E.g. a [LXD proxy device with proxy_protocol enabled](https://linuxcontainers.org/lxd/docs/master/reference/devices_proxy/)). Nginx needs to correctly consume the protocol and headers to forward need to be set from the those. Lines marked with `# <---` have different contents than BlackDex's example.

For reference this LXD downstream proxy device configuration:
```yaml
devices:
  http:
    connect: tcp:[::1]:80
    listen: tcp:[::]:80
    proxy_protocol: "true"
    type: proxy
  https:
    connect: tcp:[::1]:443
    listen: tcp:[::]:443
    proxy_protocol: "true"
    type: proxy
```

```nginx
# proxy_protocol related:

set_real_ip_from ::1; # which downstream proxy to trust, enter address of your proxy in front
real_ip_header proxy_protocol; # optional, if you want nginx to override remote_addr with info from proxy_protocol. depends on which variables you use regarding remote addr in log template and in server or stream blocks.

# below based on BlackDex's example:

# The `upstream` directives ensure that you have a http/1.1 connection
# This enables the keepalive option and better performance
#
# Define the server IP and ports here.
upstream vaultwarden-default {
  zone vaultwarden-default 64k;
  server 127.0.0.1:8080;
  keepalive 2;
}

# Needed to support websocket connections
# See: https://nginx.org/en/docs/http/websocket.html
# Instead of "close" as stated in the above link we send an empty value.
# Else all keepalive connections will not work.
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      "";
}

# Redirect HTTP to HTTPS
server {
    if ($host = vaultwarden.example.tld) {
        return 301 https://$host$request_uri;
    }

    listen 80 proxy_protocol; # <---
    listen [::]:80 proxy_protocol; # <---
    server_name vaultwarden.example.tld;
    return 404;
}

server {
    listen 443 ssl proxy_protocol; # <---
    listen [::]:443 ssl proxy_protocol; # <---
    http2 on;
    server_name vaultwarden.example.tld;

    # Specify SSL Config when needed
    #ssl_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;
    #ssl_certificate_key /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/privkey.pem;
    #ssl_trusted_certificate /path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem;

    client_max_body_size 525M;

    ## Using a Sub Path Config
    # Path to the root of your installation
    # Be sure to DO ADD a trailing /, else you will experience issues 
    # But only for this location, all other locations should NOT add this.
    location /vault/ {
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://vaultwarden-default;
    }
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

    ProxyPass / http://<SERVER>:80/ upgrade=websocket

    ProxyPreserveHost On
    ProxyRequests Off
    RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    # Add this line if your url attributes are reported back as http://... :
    # RequestHeader add X-Forwarded-Proto https
</VirtualHost>
```
</details>

<details>
<summary>Apache in a sub-location (by <a href=https://github.com/agentdr8 target=_blank>@agentdr8</a>)</summary><br/>
Modify your docker start-up to include the sub-location.

```
; Add the sub-location! Else this will not work!
DOMAIN=https://$hostname.$domainname/$sublocation/
```

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

    <Location /$sublocation/> #adjust here if necessary
        RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule /notifications/hub(.*) ws://<SERVER>:<SERVER_PORT>/$sublocation/notifications/hub/$1 [P,L]
        ProxyPass http://<SERVER>:<SERVER_PORT>/$sublocation

        ProxyPreserveHost Off
        RequestHeader set X-Real-IP %{REMOTE_ADDR}s
        RequestHeader setifempty Connection "Upgrade"
        RequestHeader setifempty Upgrade "websocket"
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
```
</details>

<details>
<summary>Traefik v2 (docker-compose example by hwwilliams, gzfrozen)</summary><br/>

#### Traefik v1 labels migrated to Traefik v2
```yaml
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.routers.bitwarden.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden.service=bitwarden
  - traefik.http.services.bitwarden.loadbalancer.server.port=80
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
  - traefik.http.routers.bitwarden-https.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-https.entrypoints=websecure
  - traefik.http.routers.bitwarden-https.tls=true
  - traefik.http.routers.bitwarden-https.service=bitwarden
  - traefik.http.routers.bitwarden-http.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-http.entrypoints=web
  - traefik.http.routers.bitwarden-http.middlewares=redirect-https
  - traefik.http.routers.bitwarden-http.service=bitwarden
  - traefik.http.services.bitwarden.loadbalancer.server.port=80
```
</details>

<details>
<summary>HAproxy (by <a href="https://github.com/BlackDex" target="_blank">@BlackDex</a>)</summary><br/>

Add these lines to your haproxy configuration. 

```haproxy
frontend vaultwarden
    bind 0.0.0.0:80
    option forwardfor header X-Real-IP
    http-request set-header X-Real-IP %[src]
    default_backend vaultwarden_http

backend vaultwarden_http
    # Enable compression if you want
    # compression algo gzip
    # compression type text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript
    server vwhttp 0.0.0.0:8080 alpn http/1.1
```
</details>
<details>
<summary>HAproxy Kubernetes Ingress(by <a href="https://github.com/devinslick" target="_blank">@BlackDex</a>)</summary><br/>

Controller installation details can be found here: https://www.haproxy.com/documentation/kubernetes-ingress/community/installation/on-prem/
Note that the CF-Connecting-IP header is only required if you use cloudflare 

Add the following resource definition:

```haproxy-kubernetes-ingress

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vaultwarden
  namespace: default
  annotations:
    haproxy.org/forwarded-for: "true"
    haproxy.org/compression-algo: "gzip"
    haproxy.org/compression-type: "text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript"
    haproxy.org/http2-enabled: "true"
spec:
  ingressClassName: haproxy
  tls:
  - hosts:
    - vaultwarden.example.tld
  rules:
  - host: vaultwarden.example.tld
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vaultwarden-http
            port:
              number: 80
```
</details>
<details>
<summary>Istio k8s (by <a href="https://github.com/asenyaev" target="_blank">@asenyaev</a>)</summary><br/>

```gateway+vs
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: vaultwarden-gateway
  namespace: vaultwarden
spec:
  selector:
    istio: ingressgateway-internal # use Istio default gateway implementation
  servers:
  - hosts:
    - vw.k8s.prod
    port:
      number: 80
      name: http
      protocol: HTTP
    tls:
      httpsRedirect: true
  - hosts:
    - vw.k8s.prod
    port:
      name: https-443
      number: 443
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: vw-k8s-prod-tls
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: vaultwarden-vs
  namespace: vaultwarden
spec:
  hosts:
  - vw.k8s.prod
  gateways:
  - vaultwarden-gateway
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        port:
          number: 80
        host: vaultwarden
```
</details>

<details>
<summary>CloudFlare Tunnel (by <a href="https://github.com/calvin-li-developer" target="_blank">@calvin-li-developer</a>)</summary><br/>

`docker-compose.yml`:

```yml
version: '3'

services:
  vaultwarden:
    container_name: vaultwarden
    image: vaultwarden/server:latest
    restart: unless-stopped
    environment:
      DOMAIN: "https://vaultwarden.example.com"  # Your domain; vaultwarden needs to know it's https to work properly with attachments
    volumes:
      - ./vw-data:/data
    networks:
      - vaultwarden-network

  cloudflared:
    image: cloudflare/cloudflared:2024.1.2
    container_name: vaultwarden-cloudflared
    restart: unless-stopped
    read_only: true
    volumes:
      - ./cloudflared-config:/root/.cloudflared/
    command: [ "tunnel", "run", "${TUNNEL_ID}" ]
    user: root
    depends_on:
      - vaultwarden
    networks:
      - vaultwarden-network
networks:
  vaultwarden-network:
    name: vaultwarden-network
    external: false
```
Contents in `cloudflared-config` folder:
```sh
config.yml  aaaaa-bbbb-cccc-dddd-eeeeeeeee.json
```
Please use [this](https://thedxt.ca/2022/10/cloudflare-tunnel-with-docker/) guide to figure out the contents/values below for your cloudflare account.
<br>Note: `aaaaa-bbbb-cccc-dddd-eeeeeeeee` is just a random tunnelID please use a real ID.

`config.yml`:
```
tunnel: aaaaa-bbbb-cccc-dddd-eeeeeeeee
credentials-file: /root/.cloudflared/aaaaa-bbbb-cccc-dddd-eeeeeeeee.json

originRequest:
  noHappyEyeballs: true
  disableChunkedEncoding: true
  noTLSVerify: true

ingress:
  - hostname: vault.example.com # change to your domain
    service: http_status:404
    path: admin
  - hostname: vault.example.com # change to your domain
    service: http://vaultwarden
  - service: http_status:404
```
`aaaaa-bbbb-cccc-dddd-eeeeeeeee.json`:
```
{"AccountTag":"changeme","TunnelSecret":"changeme","TunnelID":"aaaaa-bbbb-cccc-dddd-eeeeeeeee"}
```

</details>

<details>
<summary>Pound</summary><br/>

```
Alive		15

ListenHTTP
	Address 127.0.0.1
	Port    80
	xHTTP 3
	HeadRemove "X-Forwarded-For"
	Service
		Host "vaultwarden.example.tld"
		Redirect 301 "https://vaultwarden.example.tld"
	End
End

ListenHTTPS
	Address 127.0.0.1
	Port    443
	Cert    "/path/to/certificate/letsencrypt/live/vaultwarden.example.tld/fullchain.pem"
	xHTTP 3
	AddHeader "Front-End-Https: on"
	RewriteLocation 0
	HeadRemove "X-Forwarded-Proto"
	AddHeader "X-Forwarded-Proto: https"
End

Service
	Host "vaultwarden.example.tld"
	BackEnd
		Address <SERVER>
		Port    80
	End
End
```

</details>
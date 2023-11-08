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
  encode gzip

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
       # If you use Cloudlfare proxying, replace remote_host with http.request.header.Cf-Connecting-Ip
       # See https://developers.cloudflare.com/support/troubleshooting/restoring-visitor-ips/restoring-original-visitor-ips/
       # and https://caddy.community/t/forward-auth-copy-headers-value-not-replaced/16998/4
  }
}
```
</details>

<details>
<summary>lighttpd (by forkbomb9)</summary><br/>

```lighttpd
server.modules += ( "mod_proxy" )

$HTTP["host"] == "vault.example.net" {
    $HTTP["url"] == "/notifications/hub" {
       # WebSocket proxy
       proxy.server  = ( "" => ("vaultwarden" => ( "host" => "<SERVER>", "port" => 3012 )))
       proxy.forwarded = ( "for" => 1 )
       proxy.header = (
           "https-remap" => "enable",
           "upgrade" => "enable",
           "connect" => "enable"
       )
    } else {
       proxy.server  = ( "" => ("vaultwarden" => ( "host" => "<SERVER>", "port" => 4567 )))
       proxy.forwarded = ( "for" => 1 )
       proxy.header = ( "https-remap" => "enable" )
    }
}
```

You'll have to set `IP_HEADER` to `X-Forwarded-For` instead of `X-Real-IP` in the Vaultwarden environment.

</details>

<details>
<summary>Nginx - v1.29.0+ (by <a href="https://github.com/BlackDex" target="_blank">@BlackDex</a>)</summary><br/>

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
<summary>Nginx with sub-path - v1.29.0+ (by <a href="https://github.com/BlackDex" target="_blank">@BlackDex</a>)</summary><br/>

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
<summary>Nginx configured by Ansible/DebOps (by ypid)</summary><br/>

Ansible inventory example that uses [DebOps](https://debops.org) to configure Nginx as a reverse proxy for Vaultwarden. I choose to go with the PSK in the URL for additional security to not expose the API to everyone on the Internet because the client apps do not support client certificates yet (I tested it). Refer to [[Hardening Guide – hiding under a subdir|Hardening-Guide#hiding-under-a-subdir]].

```YAML
vaultwarden__fqdn: 'vault.example.org'
vaultwarden__http_psk_subpath_enabled: True
vaultwarden__http_psk_subpath: '{{ lookup("password", secret + "/vaultwarden/" +
                                     inventory_hostname + "/config/subpath chars=ascii_letters,digits length=23")
                                   if vaultwarden__http_psk_subpath_enabled | bool
                                   else "" }}'

nginx__upstreams:

  - name: 'vaultwarden-default'
    type: 'default'
    enabled: True
    server: 'localhost:8000'

  - name: 'vaultwarden-ws'
    type: 'default'
    enabled: True
    server: 'localhost:3012'

nginx__servers:

  - name: '{{ vaultwarden__fqdn }}'
    filename: 'debops.vaultwarden'
    by_role: 'debops.vaultwarden'
    favicon: False
    # root: '/usr/share/vaultwarden/web-vault'

    location_list:

      - pattern: '/'
        options: |-
          deny all;

      - pattern: '= /{{ vaultwarden__http_psk_subpath }}'
        options: |-
          return 307 $scheme://$host$request_uri/;

      ## All the security HTTP headers would then need to be set by nginx as well.
      # - pattern: '/{{ vaultwarden__http_psk_subpath }}/'
      #   options: |-
      #     alias /usr/share/vaultwarden/web-vault/;

      - pattern: '/{{ vaultwarden__http_psk_subpath }}/'
        options: |-
          proxy_set_header Host              $host;
          proxy_set_header X-Real-IP         $remote_addr;
          proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port  443;

          proxy_pass http://vaultwarden-default;

      - pattern: '/{{ vaultwarden__http_psk_subpath }}/notifications/hub/negotiate'
        options: |-
          proxy_set_header Host              $host;
          proxy_set_header X-Real-IP         $remote_addr;
          proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port  443;

          proxy_pass http://vaultwarden-default;

      - pattern: '/{{ vaultwarden__http_psk_subpath }}/notifications/hub'
        options: |-
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection $connection_upgrade;

          proxy_set_header Host              $host;
          proxy_set_header X-Real-IP         $remote_addr;
          proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port  443;

          proxy_pass http://vaultwarden-ws;

      # Do not use the icons features as long as it reveals all domains from
      # our credentials to the server.
      - pattern: '/{{ vaultwarden__http_psk_subpath }}/icons/'
        options: |-
          access_log off;
          log_not_found off;
          deny all;
```
</details>

<details>
<summary>Nginx (NixOS)(by tklitschi)</summary><br/>

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
        };
        locations."/notifications/hub" = {
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
<summary>Nginx with proxy_protocol in front - v1.29.0+ (by dionysius)</summary><br/>

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
<summary>Apache in a sub-location (by ss89)</summary><br/>
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
        RewriteRule /notifications/hub(.*) ws://<SERVER>:3012/$1 [P,L]
        ProxyPass http://<SERVER>:80/$sublocation/

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
```
</details>

<details>
<summary>Traefik v2 - v1.29.0+ (docker-compose example by hwwilliams, gzfrozen)</summary><br/>

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
<summary>HAproxy - v1.29.0+ (by <a href="https://github.com/BlackDex" target="_blank">@BlackDex</a>)</summary><br/>

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
<summary>HAproxy - (before v1.29.0) (by <a href="https://github.com/williamdes" target="_blank">@williamdes</a>)</summary><br/>

Add these lines to your HAproxy configuration. 

```haproxy
backend static-success-default
  mode http
  errorfile 503 /usr/local/etc/haproxy/static/index.static.default.html
  errorfile 200 /usr/local/etc/haproxy/static/index.static.default.html

frontend http-in
    bind *:443 ssl crt /acme.sh/domain.tld/domain.tld.pem alpn h2,http/1.1
    option forwardfor header X-Real-IP
    http-request set-header X-Real-IP %[src]
    default_backend static-success-default

    # Define hosts
    acl host_bitwarden_domain_tld hdr(Host) -i bitwarden.domain.tld

    ## figure out which one to use
    use_backend vaultwarden_http if host_bitwarden_domain_tld

backend vaultwarden_http
    # Enable compression if you want
    # compression algo gzip
    # compression type text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript
    # You can use the container hostname if you are using haproxy with docker-compose
    server vw_http 0.0.0.0:8080 alpn http/1.1
```
</details>

<details>
<summary>HAproxy inside PfSense (by <a href="https://github.com/RichardMawdsley" target="_blank">@RichardMawdsley</a>)</summary><br/>

Being a GUI setup, details\instructions below for you to add where required. 
 * Assumes you already have basic HTTP>HTTPS Redirection setup [Basic Setup](https://blog.devita.co/pfsense-to-proxy-traffic-for-websites-using-pfsense/)


## Backend Creation
Backend 1:
```
Mode	Name	                   Forwardto	     Address	     Port	 Encrypt(SSL)	SSL checks	Weight	Actions
active 	Vaultwarden                Address+Port:     IPADDRESSHERE   80          no             no
```
Backend 2:
```
Mode	Name	                   Forwardto	     Address	     Port	 Encrypt(SSL)	SSL checks	Weight	Actions
active 	Vaultwarden-Notifications  Address+Port:     IPADDRESSHERE   3012        no             no
```

## Frontend Creation - 1 - Domain
**ACCESS CONTROL LIST**
``` 	
ACL00
Host matches:
no
no
FQDN.com     -  NOTE:  This needs to be your root domain.  
 	
ACL00
Path starts with:
no
yes
/big-ass-randomized-test-that-really-no-one-is-ever-going-to-type-DONT-USE-THIS-LINE-THOUGH-make-your-own-up

ACL01
Host matches:
no
no
VAULTWARDEN.MYDOMAIN.COM

ACL01
Host matches:
no
no
EXAMPLE-OTHER-SUB-DOMAIN-1.MYDOMAIN.COM

ACL01
Host matches:
no
no
EXAMPLE-OTHER-SUB-DOMAIN-2.MYDOMAIN.COM
```

**ACTIONS - 1 - Domain**
``` 	
http-request allow
See below
ACL01

http-request deny
See below
ACL00
```


## Frontend Creation - 2 - VaultWarden
**ACCESS CONTROL LIST**
``` 	
ACL1
Path starts with:
no
yes
/notifications/hub  
 	
ACL2
Path starts with:
no
no
/notifications/hub/negotiate  
 	
ACL3
Path starts with:
no
no
/notifications/hub  
 	
ACL4
Path starts with:
no
yes
/notifications/hub/negotiate

ACL5
Path starts with:
no
no
/admin
```

**ACTIONS - 2 - VaultWarden**
``` 	
Use Backend
See below
ACL1  
backend: VaultWarden
 	
Use Backend
See below
ACL2  
backend: VaultWarden
 	
Use Backend
See below
ACL3  
backend: VaultWarden-Notifications
 	
Use Backend
See below
ACL4
backend: VaultWarden-Notifications

http-request deny
See below
ACL5
```

**Updates**
```
Updated above 30/07 - I realized after the first config that because ACL1-4 have 'Not' in, they were matching anything to their actions.  So BlahBlahMcGee.FQDN.com was passing through.  This was not intended, so ACL5 has been added above which resolves this, it also removes the need for the default backend.
Updated again 30/07 - ^ Yeah that didn't work.  This all stems because HaProxy doesn't allow for 'AND' in ACL's. Sigh.  Now with the above, you cofigure a front end for you root domain.  This has a deny for itself, and anything not specified.  So if you have multiple other subdomains you're passing through, you need to add them here all under ACL01.  Now everything works as it should!
```

**Important Notes**
```
1) You must keep the Domain FrontEnd up to date with any other sub domains on an allow list
2) On the Domain FrontEnd, ACL01 must be top of the Actions table - or atleast above ACL00
3) Duplicate Use of ACL names is intentional. No I havent typoed them.  ACL00, ACL01 etc
```

**OPTIONAL**
```
ACL5 above denies access to the /admin portal.  I'm not particularly fond of the admin portal not having any form of 2FA and only a password.  Thus when I'm not using it, I just deny access.  If I need it, unblock, do the required job and reblock.
```

Complete! - Go test!

This in turn will add the equivalent of below to your config (note this is an extract for example). 

	acl			ACL00	var(txn.txnhost) -m str -i VAULTWARDEN.MYDOMAIN.COM
	acl			ACL00	var(txn.txnpath) -m beg -i /big-ass-randomised-test-that-really-no-one-is-ever-going-to-type-DONT-USE-THIS-LINE-THOUGH-make-your-own-up
	acl			ACL01	var(txn.txnhost) -m str -i EXAMPLE-OTHER-SUB-DOMAIN-1.MYDOMAIN.COM
	acl			ACL01	var(txn.txnhost) -m str -i EXAMPLE-OTHER-SUB-DOMAIN-2.MYDOMAIN.COM
	acl			ACL1	var(txn.txnpath) -m beg -i /notifications/hub
	acl			ACL2	var(txn.txnpath) -m beg -i /notifications/hub/negotiate
	acl			ACL3	var(txn.txnpath) -m beg -i /notifications/hub
	acl			ACL4	var(txn.txnpath) -m beg -i /notifications/hub/negotiate
	acl			ACL5	var(txn.txnpath) -m beg -i /admin

	http-request allow  if  ACL01 
	http-request deny   if  !ACL00 
	http-request deny   if  !ACL5 
	http-request deny   if  ACL5 
	use_backend VaultWarden_ipvANY  if  !ACL1 
	use_backend VaultWarden_ipvANY  if  ACL2 
	use_backend VaultWarden-Notifications_ipvANY  if  ACL3 
	use_backend VaultWarden-Notifications_ipvANY  if  !ACL4 

To test, if you navigate in a browser to /notifications/hub then you should get a page saying "WebSocket Protocol Error: Unable to parse WebSocket key.".. that means its working! - all other sub pages should get a Rocket error.
</details>

<details>
<summary>Istio k8s (by <a href="https://github.com/dpoke" target="_blank">@dpoke</a>)</summary><br/>

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
        exact: /notifications/hub
    route:
    - destination:
        port:
          number: 3012
        host: vaultwarden-ws
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
<summary>relayd on openbsd (by olliestrickland)</summary><br/>

This is a tested and working (websockets included) - /etc/relayd.conf - on openbsd 7.2 using vaultwarden from ports - https://openports.se/security/vaultwarden

This configuration depends on proper setup of tls - I used https://man.openbsd.org/acme-client
```
table <vaultwarden-default-host> { localhost }
table <vaultwarden-websocket-host> { localhost }

# protocol definition for vaultwarden with tls

http protocol vaultwarden-https {
        # add a header vaultwarden needs
        match request header append "X-Real-IP" value "$REMOTE_ADDR"

        # add a few headers vaultwarden may not need
        match request header append "Host" value "$HOST"
        match request header append "X-Forwarded-For" value "$REMOTE_ADDR"
        match request header append "X-Forwarded-By" value "$SERVER_ADDR:$SERVER_PORT"

        # most general rule - forward connections to vaultwarden rocket
        match request path "/*" forward to <vaultwarden-default-host>

        # forward the path used for websocket to the vaultwarden websocket port
        match request path "/notifications/hub" forward to <vaultwarden-websocket-host>

        # save most specific path for last - this path should not forward to the websocket server
        match request path "/notifications/hub/negotiate" forward to <vaultwarden-default-host>

        # various TCP options
        tcp { nodelay, sack, backlog 128 }

        # tls config
        tls keypair bitwarden.example.tld
        tls { no tlsv1.0, ciphers HIGH }

        # allow websockets - this is nice it handles the connection upgrade, no need for manual header edits
        http websockets
}

# relay definition for vaultwarden - forward inbound 443 tls on the egress interface to rocket on default port 8000 and websocket on 3012

relay vaultwarden-https-relay {
        listen on egress port 443 tls
        protocol vaultwarden-https
        forward to <vaultwarden-default-host> port 8000
        forward to <vaultwarden-websocket-host> port 3012
}
```
</details>

<details>
<summary>CloudFlare (before v1.29.0) (by <a href="https://github.com/williamdes" target="_blank">@williamdes</a>)</summary><br/>

Follow the screenshot to create a new rule.
Example dashboard URL to find the settings: `https://dash.cloudflare.com/xxxxxx/example.org/rules/origin-rules/new`

![Rules](https://github.com/dani-garcia/vaultwarden/assets/7784660/e27d9152-219b-4b6a-bf96-dcfce30ebd73)
</details>

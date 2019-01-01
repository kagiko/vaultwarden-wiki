**IMPORTANT**: It's heavily recommended to activate HTTPS before enabling this feature, to avoid posible MITM attacks.

This page allows a server administrator to view all the registered users and to delete them. It also allows inviting new users, even when registration is disabled.

To enable the admin page, you need to set an authentication token. This token can be anything, but it's recommended to use a long, randomly generated string of characters, for example running `openssl rand -base64 48`.

To set the token, use the `ADMIN_TOKEN` variable:

```sh
docker run -d --name bitwarden \
  -e ADMIN_TOKEN=Vy2VyYTTsKPv8W5aEOWUbB/Bt3DEKePbHmI4m9VcemUMS2rEviDowNAFqYi1xjmp \
  -v /bw-data/:/data/ \
  -p 80:80 \
  mprasil/bitwarden:latest
```

After this, the page will be available in the `/admin` subdomain.
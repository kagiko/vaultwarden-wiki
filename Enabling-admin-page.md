**IMPORTANT**: It's heavily recommended to activate HTTPS before enabling this feature, to avoid possible MITM attacks.

This page allows a server administrator to view all the registered users and to delete them. It also allows inviting new users, even when registration is disabled.

To enable the admin page, you need to set an authentication token. This token can be anything, but it's recommended to use a long, randomly generated string of characters, for example running `openssl rand -base64 48`. **Keep this token secret, this is password to access admin area of your server!**

To set the token, use the `ADMIN_TOKEN` variable:

```sh
docker run -d --name bitwarden \
  -e ADMIN_TOKEN=some_random_token_as_per_above_explanation \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

After this, the page will be available in the `/admin` subdomain.
You can configure bitwarden_rs to send emails via a SMTP agent:

```sh
docker run -d --name bitwarden \
  -e SMTP_HOST=<smtp.domain.tld> \
  -e SMTP_FROM=<bitwarden@domain.tld> \
  -e SMTP_PORT=587 \
  -e SMTP_SSL=true \
  -e SMTP_USERNAME=<username> \
  -e SMTP_PASSWORD=<password> \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

When `SMTP_SSL` is set to `true`(this is the default), only TLSv1.1 and TLSv1.2 protocols will be accepted and `SMTP_PORT` will default to `587`. If set to `false`, `SMTP_PORT` will default to `25` and the connection won't be encrypted. This can be very insecure, use this setting only if you know what you're doing. To run SMTP in explicit mode, set `SMTP_EXPLICIT_TLS` to `true`.

Note that if SMTP and invitations are enabled, invitations will be sent to new users via email. You must set the `DOMAIN` configuration option with the base URL of your bitwarden_rs instance for the invite link to be generated correctly:

```sh
docker run -d --name bitwarden \
...
-e DOMAIN=https://vault.example.com \
...
```

User invitation links are valid for 5 days, after which a new invitation will need to be sent.
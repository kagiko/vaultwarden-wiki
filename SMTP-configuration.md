You can configure vaultwarden to send emails via a SMTP agent:

```sh
docker run -d --name vaultwarden \
  -e SMTP_HOST=<smtp.domain.tld> \
  -e SMTP_FROM=<vaultwarden@domain.tld> \
  -e SMTP_PORT=587 \
  -e SMTP_SSL=true \
  -e SMTP_USERNAME=<username> \
  -e SMTP_PASSWORD=<password> \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

When `SMTP_SSL` is set to `true`(this is the default), only TLSv1.1 and TLSv1.2 protocols will be accepted and `SMTP_PORT` will default to `587`. If set to `false`, `SMTP_PORT` will default to `25` and the opportunistic encryption will be tried (no encryption attempted with code prior to 3/12/2020). This can be very insecure, use this setting only if you know what you're doing. To run SMTP in explicit mode, set `SMTP_EXPLICIT_TLS` to `true` (SMTP_SSL has to be set to 'true', too). If you can send emails without logging in, you can simply not set `SMTP_USERNAME` and `SMTP_PASSWORD`.

Note that if SMTP and invitations are enabled, invitations will be sent to new users via email. You must set the `DOMAIN` configuration option with the base URL of your vaultwarden instance for the invite link to be generated correctly:

```sh
docker run -d --name vaultwarden \
...
-e DOMAIN=https://vault.example.com \
...
```

User invitation links are valid for 5 days, after which a new invitation will need to be sent.

## SMTP servers

Properly configuring an SMTP server/relay isn't trivial. The mailer library that vaultwarden uses also isn't the easiest to troubleshoot. So unless you're particularly interested in setting this up yourself, it's probably easier to use an external service.

Here are a few services with a free tier that allows sending 100-200 emails per day (which is plenty for most use cases):

* [SendGrid](https://sendgrid.com/)
* [MailJet](https://www.mailjet.com/)
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

When `SMTP_SSL` is set to `true`(this is the default), only TLSv1.1 and TLSv1.2 protocols will be accepted and `SMTP_PORT` will default to `587`. If set to `false`, `SMTP_PORT` will default to `25` and the opportunistic encryption will be tried (no encryption attempted with code prior to 3/12/2020). This can be very insecure, use this setting only if you know what you're doing. To run SMTP in explicit mode, set `SMTP_EXPLICIT_TLS` to `true`. If you can send emails without logging in, you can simply not set `SMTP_USERNAME` and `SMTP_PASSWORD`.

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

## Here some sane defaults for well known services

### General

Mail servers listen on port 25 mostly only to accept mail from other mail servers, and only for mail which they are the final location.<br>
Also a lot of internet providers block outgoing port 25 to prevent spamming.<br>
Most mail servers where you need to login to use either port 587, or port 465.<br>
Port 587 is called the submission port, and can most of the time only be when using a username and password. Port 587 starts off unencrypted and upgrades to a TLS encrypted connection during the communication between client and server.<br>
Port 465 is SSL encrypted from the start and no plain text communication is done at all via this port.<br>
<br>

Some general settings per port.
* for mail servers that use port 465
  ```ini
  SMTP_PORT=465
  SMTP_SSL=false
  SMTP_EXPLICIT_TLS=true
  ```
* for mail servers that use port 587 (or sometimes 25)
  ```ini
  SMTP_PORT=587
  SMTP_SSL=true
  SMTP_EXPLICIT_TLS=false
  ```
* for mail servers that do not support encryption at all.
  ```ini
  SMTP_PORT=25
  SMTP_SSL=false
  SMTP_EXPLICIT_TLS=false
  ```

### Google/Gmail
```ini
  # Domains: gmail.com, googlemail.com
  SMTP_HOST=smtp.gmail.com
  SMTP_PORT=465
  SMTP_SSL=false
  SMTP_EXPLICIT_TLS=true
  SMTP_USERNAME=<mail-address>
  SMTP_PASSWORD=<less-secure-app-password>
```
Also see: https://web.archive.org/web/20210925161633/https://webewizard.com/2019/09/17/Using-Lettre-With-Gmail/

### Hotmail/Outlook/Office365
```ini
  # Domains: hotmail.com, outlook.com, office365.com
  SMTP_HOST=smtp-mail.outlook.com
  SMTP_PORT=587
  SMTP_SSL=true
  SMTP_EXPLICIT_TLS=false
  SMTP_USERNAME=<mail-address>
  SMTP_PASSWORD=<password>
```

### Sendgrid
Encode your api-key using base64 and use that output as your password.
```bash
echo -n '<YOUR_API_KEY>' | openssl base64
```
```ini
  SMTP_HOST=smtp.sendgrid.net
  SMTP_PORT=587
  SMTP_SSL=true
  SMTP_EXPLICIT_TLS=false
  SMTP_USERNAME=apikey
  SMTP_PASSWORD=<base64-encoded-api-key>
```

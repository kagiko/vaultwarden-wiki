You can configure vaultwarden to send emails via a SMTP agent:

```sh
docker run -d --name vaultwarden \
  -e SMTP_HOST=<smtp.domain.tld> \
  -e SMTP_FROM=<vaultwarden@domain.tld> \
  -e SMTP_PORT=587 \
  -e SMTP_SECURITY=starttls \
  -e SMTP_USERNAME=<username> \
  -e SMTP_PASSWORD=<password> \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

From v1.25.0, environment variable for SMTP SSL/TLS configuration has been updated to `SMTP_SECURITY` (which was mislabelled, see bug #851).<br>
When `SMTP_SECURITY` is set to `starttls`(this is the default), only TLSv1.1 and TLSv1.2 protocols will be accepted and `SMTP_PORT` will default to `587`. If set to `off`, `SMTP_PORT` will default to `25` and the opportunistic encryption will be tried (no encryption attempted with code prior to 3/12/2020). This can be very insecure, use this setting only if you know what you're doing. To run SMTP in implicit (forced TLS) mode, set `SMTP_SECURITY` to `force_tls`. If you can send emails without logging in, you can simply not set `SMTP_USERNAME` and `SMTP_PASSWORD`.

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

Here are a few services with a free tier that supports most use cases:

* [SendGrid](https://sendgrid.com/) (100 emails per day)
* [MailJet](https://www.mailjet.com/) (200 emails per day)
* [SendinBlue](https://www.sendinblue.com) (300 emails per day)
* [SMTP2GO](https://www.smtp2go.com/) (1000 emails per month)

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
  SMTP_SECURITY=force_tls
  ```
* for mail servers that use port 587 (or sometimes 25)
  ```ini
  SMTP_PORT=587
  SMTP_SECURITY=starttls
  ```
* for mail servers that do not support encryption at all.
  ```ini
  SMTP_PORT=25
  SMTP_SECURITY=off
  ```

### HELO Hostname

By default the machine's hostname is used as the hostname in the HELO command. To overwrite this, you can set `HELO_NAME` in the configuration.

### Google/Gmail
You need to generate a App Password for Vaultwarden to work with Gmail.<br>
Follow the steps here: https://support.google.com/accounts/answer/185833?hl=en&ref_topic=7189145 (unavailable since 5/30/2022)<br>
In the end you well be shown a password (with spaces in between which are not there, it is just for easy type-over), us this password.<br>
FullSSL:
```ini
  # Domains: gmail.com, googlemail.com
  SMTP_HOST=smtp.gmail.com
  SMTP_PORT=465
  SMTP_SECURITY=force_tls
  SMTP_USERNAME=<mail-address>
  SMTP_PASSWORD=<less-secure-app-password>
```
StartTLS:
```ini
  # Domains: gmail.com, googlemail.com
  SMTP_HOST=smtp.gmail.com
  SMTP_PORT=587
  SMTP_SECURITY=starttls
  SMTP_USERNAME=<mail-address>
  SMTP_PASSWORD=<less-secure-app-password>
```
Also see: https://web.archive.org/web/20210925161633/https://webewizard.com/2019/09/17/Using-Lettre-With-Gmail/

### Hotmail/Outlook/Office365
```ini
  # Domains: hotmail.com, outlook.com, office365.com
  SMTP_HOST=smtp-mail.outlook.com
  SMTP_PORT=587
  SMTP_SECURITY=starttls
  SMTP_FROM=<mail-address>
  SMTP_USERNAME=<mail-address>
  SMTP_PASSWORD=<password>
  SMTP_AUTH_MECHANISM="Login"
```

### Sendgrid
Replace `<full-api-key>` with the generated API-Key from SendGrid which starts with `SG.`<br>
Also make sure the API-Key has full `Mail Send` rights, else you can't login with this key.<br>
StartTLS:
```ini
  SMTP_HOST=smtp.sendgrid.net
  SMTP_PORT=587
  SMTP_SECURITY=starttls
  SMTP_USERNAME=apikey
  SMTP_PASSWORD=<full-api-key>
  SMTP_AUTH_MECHANISM="Login"
```

Full SSL:
```ini
  SMTP_HOST=smtp.sendgrid.net
  SMTP_PORT=465
  SMTP_SECURITY=force_tls
  SMTP_USERNAME=apikey
  SMTP_PASSWORD=<full-api-key>
  SMTP_AUTH_MECHANISM="Login"
```

## Passwords with special characters

If you want to use some special characters within your password, it could be that you need to escape some of these characters to not confuse the environment variable parsers.<br>
For example a `\` or `'` or `"` can be used, but sometimes they need to be escaped so that they are actually used.
It is probably best, if you use special characters, to always use single quotes around the password.<br>
Lets take the following password as an example: `~^",a.%\,'}b&@|/c!1(#}`<br>
Here are a few characters which could break the environment variable parses like, `\`, `'` and `"`.
A single `\` is normally used to escape other characters, so if you want to use a single `\`, you need to type `\\`.<br>
Also, the quotes `'` and `"` could cause some issues, so lets enclose this password within single quotes and escape the special characters.<br>
To have the password above to work we need to type `'~^",a.%\\,\'}b&@|/c!1(#}'`, here you see that we escaped both the `\` and the `'` characters and used single quotes to surround the whole password.
So: `~^",a.%\,'}b&@|/c!1(#}` becomes `'~^",a.%\\,\'}b&@|/c!1(#}'`

## Troubleshooting

It often happens that people are having issues with connecting to there SMTP server from Vaultwarden.<br>
Most of the time it's a wrong configuration or the ISP/Hosting blocking the ports or IP's.<br>

Some basic steps to check if you can access the SMTP server can be done by running the following commands on the host where you are running Vaultwarden, either using Docker or as a standalone binary.

> [!NOTE]
> Replace `smtp.google.com` and `587`, `465` or `25` with the host and port matching your SMTP server.

The output of these commands should be `0`, if it returns anything else but `0`, it means there is an issue connecting to the server.

```bash
# First check if you can us this check at all by checking HTTPS access to google.com
timeout 5 bash -c 'cat < /dev/null > /dev/tcp/www.google.com/443'; echo $?

# Check for 587 SMTP Submission Port
timeout 5 bash -c 'cat < /dev/null > /dev/tcp/smtp.gmail.com/587'; echo $?

# Check for 465 SMTP SSL Port
timeout 5 bash -c 'cat < /dev/null > /dev/tcp/smtp.gmail.com/465'; echo $?

# Check for 25 Default SMTP Port
timeout 5 bash -c 'cat < /dev/null > /dev/tcp/smtp.gmail.com/25'; echo $?

# Or use a tool called nc (This tool is not always available on the host or within the container)
nc -vz smtp.gmail.com 587
```

To check this from within the docker container, run the following on the docker host first to login into the container.<br>
In the example bellow i assume the container name is `vaultwarden`, change this to the container name you used.<br>
After running the command bellow, run one of the commands above to check access from within the container also.

```bash
docker exec -it vaultwarden sh
```

## Using `sendmail` (without docker)

If you already have a working SMTP server (Postfix for ex.) running on your system and you install Vaultwarden without docker, a few extra steps are needed to allow the server to use your SMTP server through sendmail:
- in Vaultwarden config file (usually `/etc/vaultwarden.env`), set `USE_SENDMAIL=true`
- in the same file, set `SMTP_FROM=user@example.com` (replace with your own!) variable since it's also used by sendmail
- as `root` user (or using `sudo`), add `vaultwarden` user to `postdrop` group with `gpasswd -a vaultwarden postdrop`
- edit vaultwarden systemd service with `systemctl edit vaultwarden` and add this two lines in `[Service]` section:
```
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6 AF_LOCAL AF_NETLINK
ReadWritePaths=/var/lib/vaultwarden /var/log/vaultwarden.log /var/spool/postfix/maildrop
```

Finally don't forget to restart the service with `systemctl restart vaultwarden`.
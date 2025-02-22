Suppose you want to run an instance of vaultwarden that can only be accessed from your local network, but you want your instance to be HTTPS-enabled with certs signed by a widely-accepted CA instead of managing your own [private CA](https://github.com/dani-garcia/vaultwarden/wiki/Private-CA-and-self-signed-certs-that-work-with-Chrome) (to avoid the hassle of having to load private CA certs into all of your devices).

This article demonstrates how to create such a setup using the [Caddy](https://caddyserver.com/) web server, which has built-in ACME support for various DNS providers. We'll configure Caddy to obtain Let's Encrypt certs via the ACME [DNS challenge](https://letsencrypt.org/docs/challenge-types/#dns-01-challenge) -- using the more common HTTP challenge would be problematic here, since it would rely on Let's Encrypt servers being able to reach your internal web server.

> [!WARNING]
> This article covers the DNS challenge setup in more generic terms, but many users will probably find it easiest to use Docker Compose to integrate Caddy and Vaultwarden. See [Using Docker Compose](https://github.com/dani-garcia/vaultwarden/wiki/Using-Docker-Compose#caddy-with-dns-challenge) for an example specific to that.

Two DNS providers are covered:

* [Duck DNS](https://www.duckdns.org/) -- This gives you a subdomain under `duckdns.org` (e.g., `my-vw.duckdns.org`). This option is simplest if you don't already own a domain.
* [Cloudflare](https://www.cloudflare.com/) -- This lets you put your vaultwarden instance under a domain you own or control. Note that Cloudflare can be used as just a DNS provider (i.e., without the proxying functionality that Cloudflare is best known for). If you don't currently own a domain, you may be able to get a free one at [Freenom](https://www.freenom.com/).

It's certainly possible to create a similar setup using other combinations of web server, [ACME client](https://letsencrypt.org/docs/client-options/), and DNS provider, but you'll have to work out the differences in details.

## Getting a custom Caddy build

DNS challenge support is not built into Caddy by default, as most people don't use this challenge method, and it requires a custom implementation for each DNS provider.

The easiest way to get a version of Caddy with the necessary DNS challenge modules is via https://caddyserver.com/download. Select your platform, check the box for `github.com/caddy-dns/cloudflare` (for Cloudflare support) and/or `github.com/caddy-dns/duckdns` (for Duck DNS support), and then click `Download`.

If you prefer to build from source, you can use [`xcaddy`](https://caddyserver.com/docs/build#xcaddy). For example, to create a build that includes both Cloudflare and Duck DNS support:

    xcaddy build --with github.com/caddy-dns/cloudflare --with github.com/caddy-dns/duckdns

Move the `caddy` binary to `/usr/local/bin/caddy` or some other appropriate directory in your path. Make the file executable. Optionally, run `sudo setcap cap_net_bind_service=+ep /usr/local/bin/caddy` to allow `caddy` to listen on privileged ports (< 1024) without running as root.

## Duck DNS setup

If you don't already have an account, create one at https://www.duckdns.org/. Create a subdomain for your vaultwarden instance (e.g., `my-vw.duckdns.org`) and set its IP to your vaultwarden host's private IP (e.g., `192.168.1.100`). Make note of your account's token (a string in [UUID](https://en.wikipedia.org/wiki/UUID) format). Caddy will need this token to solve the DNS challenge.

Create a file named `Caddyfile` (capital C and no file extention) in the same directory where the caddy executable is located with the following content and replace the `localhost:` port by the one used by your vaultwarden in its `ROCKET_PORT=` directive (vaultwarden's default rocket_port is 8001):

```
{$DOMAIN}:443 {
    tls {
        dns duckdns {$DUCKDNS_TOKEN}
    }
    reverse_proxy localhost:8001
}
```

Create a file named `caddy.env` with the following content (replacing each value as appropriate):
```
DOMAIN=my-vw.duckdns.org
DUCKDNS_TOKEN=00112233-4455-6677-8899-aabbccddeeff
```

Change to the directory where caddy is located and start `caddy` for the first time by running
```
caddy run --envfile caddy.env
```

The first start of caddy for your Duck DNS domain (e.g. my-vw.duckns.org) takes a few seconds to solve the DNS challenge and obtain the HTTPS certificates. Caddy usually stores them in `/root/.local/share/caddy` and caddy's configuration is automatically saved to `/root/.config/caddy`.

Start `vaultwarden` - if not already the case - by running
```
export ROCKET_PORT=8001

./vaultwarden
```

> [!NOTE]
> It does not matter if vaultwarden has been running already or not before you set up caddy. 

You should now be able to reach your vaultwarden instance at https://my-vw.duckdns.org. If not, check the output of caddy.

You can stop caddy by [STRG]-[C]. In the future start caddy in the background by running
```
caddy start --envfile caddy.env
```

**Important:** If necessary, in some routers (e.g. FritzBox) or DNS resolvers (e.g. unbound) an exception must be set for the domain (e.g., `my-vw.duckdns.org`) due to DNS rebind protection.

## Cloudflare setup

If you don't already have an account, create one at https://www.cloudflare.com/; you'll also have to go to your domain registrar to set your nameservers to the ones assigned to you by Cloudflare. Create a subdomain for your vaultwarden instance (e.g., `vw.example.com`), setting its IP to your vaultwarden host's private IP (e.g., `192.168.1.100`). For example:

![A record config](https://i.imgur.com/BBvy4Yj.png)

Create an API token for the DNS challenge (for more background, see https://github.com/libdns/cloudflare/blob/master/README.md):

1. In the upper right, click the person icon and navigate to `My Profile`, and then select the `API Tokens` tab.
1. Click the `Create Token` button, and then `Use template` on `Edit zone DNS`.
1. Edit the `Token name` field if you prefer a more descriptive name.
1. Under `Permissions`, the `Zone / DNS / Edit` permission should already be populated. Add another permission: `Zone / Zone / Read`.
1. Under `Zone Resources`, set `Include / Specific zone / example.com` (replacing `example.com` with your domain).
1. Under `TTL`, set an End Date for when your token will become inactive. You might want to choose one far in the future.
1. Create the token and copy the token value.

Your token list should look like:

![API token config](https://i.imgur.com/FoOv9Ww.png)

Create a file named `Caddyfile` with the following content:
```
{$DOMAIN}:443 {
    tls {
        dns cloudflare {$CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy localhost:8080
}
```

Create a file named `caddy.env` with the following content (replacing each value as appropriate):
```
DOMAIN=vw.example.com
CLOUDFLARE_API_TOKEN=<your-api-token>
```

Start `caddy` by running
```
caddy run --envfile caddy.env
```

Start `vaultwarden` by running
```
export ROCKET_PORT=8080

./vaultwarden
```

You should now be able to reach your vaultwarden instance at https://vw.example.com.

**Important:** If necessary, in some routers (e.g. FritzBox) an exception must be set for the domain (e.g., `vw.example.com`) due to DNS rebind protection.

## Getting certs using the `lego` CLI

In the DuckDNS example above, Caddy used the `lego` library to get certs via DNS challenge.
`lego` also has a CLI that you can use to get certs directly, e.g. if you want to use a reverse proxy other than Caddy.
(Note: This example uses `lego`, but there are other standalone ACME clients that support the DNS challenge method (see the [DNS Challenge](#dns-challenge) section.) 

Here's an example of how to do this:

1. Download a pre-built `lego` binary for your system from https://github.com/go-acme/lego/releases. Extract the contents to some directory, say, `/usr/local/lego`.
2. From that directory, run `DUCKDNS_TOKEN=<token> ./lego -a --dns duckdns -d my-vw.duckdns.org -m me@example.com run`,
   substituting appropriate values for the token, domain, and email address. This registers you with Let's Encrypt and
   fetches a certificate for your domain.
3. Set up a weekly cron job to run `DUCKDNS_TOKEN=<token> ./lego --dns duckdns -d my-vw.duckdns.org -m me@example.com renew`.
   This renews your certificate as it nears expiration.

> [!NOTE]
> `lego` requests ECC/ECDSA certs by default. If you are using the [[Rocket HTTPS server|Enabling-HTTPS#via-rocket]] built into vaultwarden, you will need to request RSA certs instead. In the `lego` commands above, add the option `--key-type rsa2048`.

In this example, the generated outputs you need to configure your reverse proxy with are:

* `/usr/local/lego/.lego/certificates/my-vw.duckdns.org.crt` (certificate)
* `/usr/local/lego/.lego/certificates/my-vw.duckdns.org.key` (private key)

## Troubleshooting

### DNS issues

If you get a DNS resolution error for your subdomain (e.g., `DNS_PROBE_FINISHED_NXDOMAIN` or `ERR_NAME_NOT_RESOLVED`), your DNS resolver is probably blocking resolution because:

1. It blocks dynamic DNS services for security reasons.
2. It blocks domains that resolve to private (RFC 1918) IP addresses to prevent [DNS rebinding](https://en.wikipedia.org/wiki/DNS_rebinding) attacks, or for some other reason.

In either case, you might try using another DNS resolver, such as Google's `8.8.8.8` or Cloudflare's `1.1.1.1`. In the second case, if you're running behind a local DNS server like dnsmasq or Unbound, you may be able to configure it to either disable DNS rebind protection entirely, or allow certain domains to return private addresses. Regarding Unbound you can do so by adding the following directive to its configuration file (replacing the domain by your own Duck DNS domain):
```
private-domain: "my-vw.duckdns.org"
```

Afterwards restart unbound by `unbound-control reload` or `systemctl restart unbound` to make it load its new configuration.

Additionally, make sure to turn off a previous HTTPS setup you might had set up for vaultwarden, in particular a private CA with your own (self-signed) certificates via Rocket TLS because this interferes with your new Let's Encrypt-protected domain. Simply do so by commenting out (# sign) the ROCKET_TLS directive in your vaultwarden's environment file:

```
# ROCKET_TLS={certs="./cert.pem",key="./privkey.pem"}
```

### Vaultwarden login issues

Do not miss to update vaultwardens's environment file to reflect the domain change:
```
DOMAIN=https://my-vw.duckdns.org
```

## References

### DNS Challenge

* https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438
* https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148

### Caddy Cloudflare module

* https://github.com/caddy-dns/cloudflare
* https://go-acme.github.io/lego/dns/cloudflare/

### Caddy Duck DNS module

* https://github.com/caddy-dns/duckdns
* https://go-acme.github.io/lego/dns/duckdns/

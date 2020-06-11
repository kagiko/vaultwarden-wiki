# Application configuration

The subsections below cover hardening related to bitwarden_rs itself.

## Disable registration and (optionally) invitations

By default, bitwarden_rs allows any anonymous user to register new accounts on the server without first being invited. This is necessary to create your first user on the server, but it's recommended that you disable it in the admin panel (if the admin panel is enabled) or [[with the environment variable|Disable-registration-of-new-users]] to prevent attackers from creating accounts on your bitwarden_rs server.

bitwarden_rs also allows registered users to invite other new users to create accounts on the server and join their organizations. This does not pose an immediate risk (as long as you trust your users), but it can be disabled in the admin panel or [[with the environment variable|Disable-invitations]].

## Disable password hint display
bitwarden_rs displays password hints on the login page to accommodate small/local deployments that do not have SMTP configured, which could be abused by an attacker to facilitate password-guessing attacks against users on the server. This can be disabled in the admin panel by unchecking the `Show password hints` option or [[with the environment variable|Password-hint-display]].

# HTTPS / TLS configuration

The subsections below cover hardening related to HTTPS / TLS.

## Strict SNI

[SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) is how a web browser requests that an HTTPS server provide the SSL/TLS certificate for a particular site (e.g., `bitwarden.example.com`). Suppose `bitwarden.example.com` has the IP address `1.2.3.4`. Ideally, you want your instance to only be accessible via https://bitwarden.example.com, and not https://1.2.3.4. This is because IP addresses are continually scanned for various reasons, and your bitwarden_rs instance becomes a more obvious target if it can be detected this way. For example, a simple [Shodan search](https://www.shodan.io/search?query=bitwarden) reveals a number of Bitwarden instances that are accessible by IP address.

## Reverse proxying

In general, you should avoid enabling HTTPS via bitwarden_rs's built-in [[Rocket TLS support|Enabling-HTTPS]], especially if your instance is publicly accessible. Rocket itself lists the following [warning](https://rocket.rs/v0.4/guide/configuration/#configuring-tls):

> Rocket's built-in TLS is not considered ready for production use. It is intended for development use only.

For example, Rocket TLS doesn't support [strict SNI](#Strict-SNI) or ECC certs (only RSA).

See [[Proxy example|Proxy-examples]] for some sample reverse proxy configurations.

# Miscellaneous

## Brute-force mitigation
When two-factor-authentication is not in use, it is (in theory) possible to brute-force user passwords and thus gain access to their account. One, relatively easy, way to mitigate this, is setting up fail2ban which blocks ipadresses after too many  failed login attempts. However: Care should be taken when using this behind multiple reverse-proxies (such as cloudflare).
See: [[Fail2Ban Setup|Fail2Ban Setup]]
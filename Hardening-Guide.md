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

# Docker configuration

The subsections below cover hardening related to Docker.

## Run as a non-root user

The bitwarden_rs Docker image is configured to run the container process as the `root` user by default. This allows bitwarden_rs to read/write any data [bind-mounted](https://docs.docker.com/storage/bind-mounts/) into the container without permission issues, even if that data is owned by another user (e.g., your user account on the Docker host). 
The default configuration provides a good balance of security and usability -- running as root within an unprivileged Docker container provides a reasonable level of isolation on its own, while also making setup easier for users who aren't necessarily well-versed in how to manage ownership/permissions on Linux. However, as a general policy, it's better security-wise to run processes with the minimum privileges required; this is somewhat less of a concern with programs written in a memory-safe language like Rust, but note that bitwarden_rs does also use some library code written in C (SQLite, OpenSSL, MySQL, PostgreSQL, etc.).

To run the container process (bitwarden_rs) as a non-root user (uid/gid 1000) in Docker:

    docker run -u 1000:1000 -e ROCKET_PORT=8080 -p <host-port>:8080 \
           [...other args...] \
           bitwardenrs/server:latest

The default user in many Linux distros has uid/gid 1000 (run the `id` command to verify), so this is a good value to use if you prefer to be able to easily access your bitwarden_rs data without changing to another user, but you can adjust the uid/gid as needed. Note that you'll most likely need to specify a numeric uid/gid, because the bitwarden_rs container doesn't share the same mapping of user/group names to uid/gid (e.g., compare the `/etc/passwd` and `/etc/group` files in the container to the ones on the Docker host).

 `ROCKET_PORT` defaults to 80, which is a [privileged port](https://www.w3.org/Daemon/User/Installation/PrivilegedPorts.html); it needs to be 1024 or higher when running as a non-root user, or else you'll get a permission denied error when bitwarden_rs attempts to bind and listen for connections on that port.

To do the same in `docker-compose`:

    services:
      bitwarden:
        image: bitwardenrs/server:latest
        container_name: bitwarden
        user: 1000:1000
        environment:
          - ROCKET_PORT=8080

        ... other configuration ...

Since `ROCKET_PORT` is being changed here, make sure to also update your reverse proxy config to proxy bitwarden_rs traffic to port 8080 (or whatever higher port you chose) instead of 80.

## Mounting data into the container

Generally, only data that bitwarden_rs needs to operate properly should be mounted into the bitwarden_rs container (typically, this is just your data directory, and maybe a directory containing SSL/TLS certs and private keys). For example, don't mount your entire home directory, `/var/run/docker.sock`, etc. unless you have a specific reason and know what you're doing.

Also, if you don't expect bitwarden_rs to modify the data you're mounting in (e.g., certs), then [mount it read-only](https://docs.docker.com/storage/bind-mounts/#use-a-read-only-bind-mount) by adding `:ro` to the volume specification (for example, `docker run -v /home/username/bitwarden-ssl:/ssl:ro`).

# Miscellaneous

## Brute-force mitigation

When two-factor-authentication is not in use, it is (in theory) possible to brute-force user passwords and thus gain access to their account. One, relatively easy, way to mitigate this, is setting up fail2ban which blocks ipadresses after too many  failed login attempts. However: Care should be taken when using this behind multiple reverse-proxies (such as cloudflare).
See: [[Fail2Ban Setup|Fail2Ban Setup]]

## Hiding under a subdir

Traditionally, a Bitwarden instance resides at the root of a subdomain (i.e., `bitwarden.example.com`, and not `bitwarden.example.com/some/path`). The upstream Bitwarden server currently only supports subdomain roots, while bitwarden_rs adds support for [[alternate base directories|Using-an-alternate-base-dir]]. For some users, this is useful simply because they only have access to one subdomain and want to run multiple services under different directories. In such cases, they typically choose something obvious like `mysubdomain.example.com/bitwarden`. However, you can also use this to provide an extra layer of protection by putting bitwarden_rs under something like `mysubdomain.example.com/bitwarden/<mysecretstring>`, where `<mysecretstring>` effectively acts as a password. Some may argue that this is [security through obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity), but it's actually [defense in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)) -- the secrecy of the subdir is just an extra layer of security, and not intended to be the primary means of security (which is still the strength of a user's master password).
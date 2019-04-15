LDAP integration is performed using a small service that queries LDAP and invites users to your Bitwarden instance. This service is uncreatively named [bitwarden_rs_ldap](https://github.com/ViViDboarder/bitwarden_rs_ldap).

It is not yet distributed as a binary, but there is an available Docker image [vividboarder/bitwarden_rs_ldap](https://hub.docker.com/r/vividboarder/bitwarden_rs_ldap).

Before deploying, you must [[enable your bitwarden_rs admin page|Enabling-admin-page]]. This is enables the API that the LDAP sync service will use to invite users. The `ADMIN_TOKEN` that you set will be used when configuring the LDAP sync service. You must also be sure to **not** disable the invitation capability. To verify this, double check that the environment variable `INVITATIONS_ALLOWED` is not set to `false`.

It is also recommended to [[enable email sending|SMTP-configuration]] from your Bitwarden instance so that your users will be notified they can make an account. If you do not, they will still be able to register if they use their LDAP email address, but you will have to inform them on your own.

With these steps done, you can configure and deploy the LDAP sync service. The most up-to-date instructions are to be found on the service [Readme](https://github.com/ViViDboarder/bitwarden_rs_ldap) itself, but it will involve creating the `config.toml` file with the connection info for your bitwarden_rs instance, your LDAP instance, as well as the LDAP query you'd like to use to find users.
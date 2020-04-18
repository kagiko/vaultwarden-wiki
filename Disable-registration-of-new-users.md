By default, anyone who can access your instance can register for a new account. To disable this, set the `SIGNUPS_ALLOWED` env variable to `false`:

```sh
docker run -d --name bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

## Disabling organization invitations

Even when `SIGNUPS_ALLOWED=false`, an existing user who is an organization owner or admin can still invite new users. If you want to disable this as well, see [[Disable invitations|disable-invitations]].

## Restricting registrations to certain email domains

You can restrict registration to email addresses from certain domains by setting `SIGNUPS_DOMAINS_WHITELIST` accordingly. For example:

* `SIGNUPS_DOMAINS_WHITELIST=example.com` (single domain)
* `SIGNUPS_DOMAINS_WHITELIST=example.com,example.net,example.org` (multiple domains)

If `SIGNUPS_DOMAINS_WHITELIST` is set, then the value of `SIGNUPS_ALLOWED` is ignored. Also, see [#728](https://github.com/dani-garcia/bitwarden_rs/pull/728) for caveats -- in particular, the emails are currently not checked, meaning that anyone could still register, by providing a fake email address that has the proper domain.

## Invitations via the admin page

The bitwarden_rs admin can invite anyone via the [[admin page|Enabling-admin-page]], regardless of any of the restrictions above.
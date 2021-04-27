By default, anyone who can access your instance can register for a new account. To disable this, set the `SIGNUPS_ALLOWED` env variable to `false`:

```sh
docker run -d --name bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /bw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

Note that when `SIGNUPS_ALLOWED=false`, the `Create Account` button will still be shown in the web vault UI, but actually attempting to create an account will result in an error message. Upstream Bitwarden isn't designed to allow disabling signups, so this can't be worked around easily.

## Disabling organization invitations

Even when `SIGNUPS_ALLOWED=false`, an existing user who is an organization owner or admin can still invite new users. If you want to disable this as well, see [[Disable invitations|disable-invitations]].

## Restricting registrations to certain email domains

You can restrict registration to email addresses from certain domains by setting `SIGNUPS_DOMAINS_WHITELIST` accordingly. For example:

* `SIGNUPS_DOMAINS_WHITELIST=example.com` (single domain)
* `SIGNUPS_DOMAINS_WHITELIST=example.com,example.net,example.org` (multiple domains)

:warning: If `SIGNUPS_DOMAINS_WHITELIST` is set, then the value of `SIGNUPS_ALLOWED` is ignored.

You may also want to set `SIGNUPS_VERIFY=true`, which would require email verification before a newly-registered user can successfully log in. This would prevent someone from registering with a fake email address that has the proper domain.

## Invitations via the admin page

The vaultwarden admin can invite anyone via the [[admin page|Enabling-admin-page]], regardless of any of the restrictions above.
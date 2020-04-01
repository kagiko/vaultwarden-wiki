By default new users can register, if you want to disable that, set the `SIGNUPS_ALLOWED` env variable to `false`:

```sh
docker run -d --name bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```
Note: While users can't register on their own, they can still be invited by already registered users. See [[Disable invitations|disable-invitations]] if you also want to disable that.

You can also disable registration except for email addresses from certain domains. For example:

* `SIGNUPS_DOMAINS_WHITELIST=example.com` (single domain)
* `SIGNUPS_DOMAINS_WHITELIST=example.com,example.net,example.org` (multiple domains)

You still need to set `SIGNUPS_ALLOWED=false`. Also, see [#728](https://github.com/dani-garcia/bitwarden_rs/pull/728) for caveats -- in particular, the emails are currently not checked, meaning that anyone could still register, by providing a fake email address that has the proper domain.
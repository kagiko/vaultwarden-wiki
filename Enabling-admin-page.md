**IMPORTANT**: It's heavily recommended to activate HTTPS before enabling this feature, to avoid possible MITM attacks.

This page allows a server administrator to view all the registered users and to delete them. It also allows inviting new users, even when registration is disabled.

To enable the admin page, you need to set an authentication token. This token can be anything, but it's recommended to use a long, randomly generated string of characters, for example running `openssl rand -base64 48`. **Keep this token secret, this is the password to access the admin area of your server!**

To set the token, use the `ADMIN_TOKEN` variable:

```sh
docker run -d --name bitwarden \
  -e ADMIN_TOKEN=some_random_token_as_per_above_explanation \
  -v /bw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

After this, the page will be available in the `/admin` subdirectory.

The first time you save a setting in the admin page, `config.json` will be generated in your `DATA_FOLDER`. Values in this file will take precedence over the corresponding environment variable.

Note that config changes in the admin page do not take effect until you click the `Save` button. For example, if you are testing SMTP settings, and you change the `SMTP Auth mechanism` setting and then click `Send test email` to test the change, this won't work as expected -- since you didn't click `Save`, the `SMTP Auth mechanism` change won't have taken effect.

**Note:** After changing the `ADMIN_TOKEN`, the currently logged in admins will still be able to use their old login token for [up to 20 minutes](https://github.com/dani-garcia/vaultwarden/blob/master/src/api/admin.rs#L87).
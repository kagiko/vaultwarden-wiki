

## Disable registration and (optionally) invitations
By default, bitwarden_rs allows any anonymous user to register new accounts on the server without first being invited. This is necessary to create your first user on the server, but it's recommended that you disable it in the admin panel (if the admin panel is enabled) or with the `SIGNUPS_ALLOWED=false` environment variable to prevent attackers from creating accounts on your bitwarden_rs server.

bitwarden_rs also allows registered users to invite other new users to create accounts on the server and join their organizations. This does not pose an immediate risk (as long as you trust your users), but it can be disabled in the admin panel or with the `INVITATIONS_ALLOWED=false` environment variable.

## Enable HTTPS

## Disable password hint display
bitwarden_rs displays password hints on the login page to accommodate small/local deployments that do not have SMTP configured, which could be abused by an attacker to facilitate password-guessing attacks against users on the server. This can be disabled in the admin panel by unchecking the `Show password hints` option or by starting the server with the `SHOW_PASSWORD_HINT=false` environment variable.

## Disable icon fetching

## SMTP hardening

## Brute-force mitigation
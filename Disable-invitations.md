Even when registration is disabled, organization administrators or owners can invite users to join organization. After they are invited, they can register with the invited email even if `SIGNUPS_ALLOWED` is actually set to `false`. You can disable this functionality completely by setting `INVITATIONS_ALLOWED` env variable to `false`:

```sh
docker run -d --name bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -e INVITATIONS_ALLOWED=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```
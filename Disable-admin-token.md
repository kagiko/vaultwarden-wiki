> [!IMPORTANT]
> Your administration page will be accessible to anyone

If you have another method you would like to use for authentication to the `/admin` page then you can set the `DISABLE_ADMIN_TOKEN` variable to true. This will disable the built in `ADMIN_TOKEN` used for authentication while also enabling the admin panel. Anyone with access to the URL will be able to access the admin panel. You will need to take extra steps to secure it. This includes externally and locally.

```bash
docker run -d --name bitwarden \
  -e DISABLE_ADMIN_TOKEN=true \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```
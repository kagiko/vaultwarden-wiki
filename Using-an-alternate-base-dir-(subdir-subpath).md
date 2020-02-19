Traditionally, Bitwarden is limited to residing at the root of a subdomain, e.g. `https://bitwarden.example.com`.

This limitation originates in the backend and web vault, which haven't been designed to accommodate alternate base dirs (see [bitwarden/server#277](/bitwarden/server/issues/277)). The mobile/desktop apps and browser extensions actually have no issues using a base URL with a path.

In bitwarden_rs, with the changes in [PR#868](../pull/868), you can configure the backend server to work properly with an alternate base dir. With a bit more work, it's also possible to modify the web vault to work properly, resulting in a fully functional installation.

## Configuring the backend server

Simply configure your domain URL to include the base dir. For example, suppose you want to access your installation at `https://bitwarden.example.com/secret-path`.

1. Stop bitwarden_rs.
2. If you normally configure bitwarden_rs using the admin page, edit your `config.json` to look as follows:
    ```javascript
    {
      "domain": "https://bitwarden.example.com/secret-path",
      // ... other values ...
    }
    ```
3. If you normally configure bitwarden_rs via environment variables, update your config files/scripts to set the `DOMAIN` environment variable to the base URL. For example:
   ```sh
   docker run -e DOMAIN="https://bitwarden.example.com/secret-path" ...
   ```
4. Restart bitwarden_rs.
5. You should now be able to access the web vault at `https://bitwarden.example.com/secret-path/` (note the trailing slash). For reasons not entirely clear, you may run into issues if you use `https://bitwarden.example.com/secret-path` (without the trailing slash).
6. Configure your apps or browser extensions to use `https://bitwarden.example.com/secret-path`. If you add a trailing slash, the apps and extensions will automatically remove it before saving.

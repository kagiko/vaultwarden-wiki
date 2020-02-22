Traditionally, Bitwarden is limited to residing at the root of a subdomain, e.g. `https://bitwarden.example.com`.

This limitation originates in the backend and web vault, which haven't been designed to accommodate alternate base dirs (see [bitwarden/server#277](/bitwarden/server/issues/277)). The mobile/desktop apps and browser extensions actually have no issues using a base URL with a path.

In bitwarden_rs, with the changes in [PR#868](https://github.com/dani-garcia/bitwarden_rs/pull/868) (backend) and [PR#11](https://github.com/dani-garcia/bw_web_builds/pull/11) (web vault), you can configure a fully functional instance at an alternate base dir.

## Configuration

Simply configure your domain URL to include the base dir. For example, suppose you want to access your instance at `https://bitwarden.example.com/base-dir`. (Note that you can also use multiple levels of directories, like `https://bitwarden.example.com/multi/level/base/dir`if you want.)

1. Stop bitwarden_rs.
2. If you normally configure bitwarden_rs using the admin page, edit your `config.json` to look as follows:
    ```javascript
    {
      "domain": "https://bitwarden.example.com/base-dir",
      // ... other values ...
    }
    ```
3. If you normally configure bitwarden_rs via environment variables, update your config files/scripts to set the `DOMAIN` environment variable to the base URL. For example:
   ```sh
   docker run -e DOMAIN="https://bitwarden.example.com/base-dir" ...
   ```
4. Restart bitwarden_rs.
5. You should now be able to access the web vault at `https://bitwarden.example.com/base-dir/` (note the trailing slash). For reasons not entirely clear, you'll probably run into issues if you use `https://bitwarden.example.com/base-dir` (without the trailing slash).
6. Configure your apps or browser extensions to use `https://bitwarden.example.com/base-dir`. If you add a trailing slash, the apps and extensions will automatically remove it before saving.
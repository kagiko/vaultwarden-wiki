Traditionally, Bitwarden is limited to residing at the root of a subdomain, e.g. `https://bitwarden.example.com`.

This limitation originates in the backend and web vault, which haven't been designed to accommodate alternate base dirs (see [bitwarden/server#277](/bitwarden/server/issues/277)). The mobile/desktop apps and browser extensions actually have no issues using a base URL with a path.

In vaultwarden, with the changes in [PR#868](https://github.com/dani-garcia/vaultwarden/pull/868) (backend) and [PR#11](https://github.com/dani-garcia/bw_web_builds/pull/11) (web vault), you can configure a fully functional instance at an alternate base dir.

## Configuration

Simply configure your domain URL to include the base dir. For example, suppose you want to access your instance at `https://bitwarden.example.com/base-dir`. (Note that you can also use multiple levels of directories, like `https://bitwarden.example.com/multi/level/base/dir`if you want.)

1. Stop vaultwarden.
2. If you normally configure vaultwarden using the admin page, edit your `config.json` to look as follows:
    ```javascript
    {
      "domain": "https://bitwarden.example.com/base-dir",
      // ... other values ...
    }
    ```
3. If you normally configure vaultwarden via environment variables, update your config files/scripts to set the `DOMAIN` environment variable to the base URL. For example:
   ```sh
   docker run -e DOMAIN="https://bitwarden.example.com/base-dir" ...
   ```
4. Restart vaultwarden.
5. You should now be able to access the web vault at `https://bitwarden.example.com/base-dir/` (note the trailing slash). For reasons not entirely clear, you'll probably run into issues if you use `https://bitwarden.example.com/base-dir` (without the trailing slash).
6. Configure your apps or browser extensions to use `https://bitwarden.example.com/base-dir`. If you add a trailing slash, the apps and extensions will automatically remove it before saving.
7. Note over **5**. The trailing slash `/` issue could be solved by appending `/` after the route location string. For example, in nginx.

    ```
    location /my-base-path {
      # This config would cause `/` issue
    }
    
    location /my-base-path-2/ {
      # This config works perfectly
    }
    ```

## Reverse proxying

If you are putting vaultwarden behind a reverse proxy, make sure your proxy is configured to pass the request path through to vaultwarden, since the vaultwarden API routes are set up to expect the base dir. So if a request for `https://bitwarden.example.com/base-dir/api/sync` hits your reverse proxy, which then proxies to your vaultwarden listening on `localhost:8080`, the request must go to `http://localhost:8080/base-dir/api/sync`, not `http://localhost:8080/api/sync`.
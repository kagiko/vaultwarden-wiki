Traditionally, Bitwarden is limited to residing at the root of a subdomain, e.g. `https://bitwarden.example.com`.

This limitation originates in the backend and web vault, which haven't been designed to accommodate alternate base dirs (see [bitwarden/server#277](/bitwarden/server/issues/277)). The mobile/desktop apps and browser extensions actually have no issues using a base URL with a path.

In bitwarden_rs, with the changes in [PR#868](../pull/868), you can configure the backend server to work properly with an alternate base dir. With a bit more work, it's also possible to modify the web vault to work properly, resulting in a fully functional installation.

## Configuring the backend server

Simply configure your domain URL to include the base dir. For example, suppose you want to access your installation at `https://bitwarden.example.com/secret-dir`.

1. Stop bitwarden_rs.
2. If you normally configure bitwarden_rs using the admin page, edit your `config.json` to look as follows:
    ```javascript
    {
      "domain": "https://bitwarden.example.com/secret-dir",
      // ... other values ...
    }
    ```
3. If you normally configure bitwarden_rs via environment variables, update your config files/scripts to set the `DOMAIN` environment variable to the base URL. For example:
   ```sh
   docker run -e DOMAIN="https://bitwarden.example.com/secret-dir" ...
   ```
4. Restart bitwarden_rs.
5. You should now be able to access the web vault (assuming it has been modified appropriately; see the next section) at `https://bitwarden.example.com/secret-dir/` (note the trailing slash). For reasons not entirely clear, you may run into issues if you use `https://bitwarden.example.com/secret-dir` (without the trailing slash).
6. Configure your apps or browser extensions to use `https://bitwarden.example.com/secret-dir`. If you add a trailing slash, the apps and extensions will automatically remove it before saving.

## Modifying the web vault

The issue with the web vault is there's no simple way to configure it for a specific base URL. Instead, the code generally just assumes the web vault URL is given by `window.location.origin`, which always represents the root of the subdomain. This is true of both the upstream web vault and the patched version used in bitwarden_rs:

* https://github.com/bitwarden/web/blob/f7f7040/src/app/services/services.module.ts#L137-L144
* https://github.com/dani-garcia/bw_web_builds/blob/5c9de1a/patches/v2.11.0.patch#L17-L29

Here are some approaches you could take to modify the web vault to work at a different base dir.

### The hard and clean way

Modify the upstream code and/or bitwarden_rs patches and rebuild the web vault. (Someone else can document this if they're interested.)

### The quick and dirty way

1. Enter a shell in the bitwarden_rs container: `docker exec -it <container-name> /bin/sh`
2. Patch the web vault: `sed -i "s|window\.location\.origin|window.location.origin+'/secret-dir'|g" /web-vault/app/main*.js` (of course, replace `/secret-dir` with your actual base dir)

Pros:
* It works just fine for normal purposes.
* This approach could be easily automated on container start.

Cons:
* It's a brittle solution, although it's probably not too likely `window.location.origin` would be used for anything else.
* It will probably break the [source map](https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/), but this won't matter unless you're doing development or need to troubleshoot with a developer.
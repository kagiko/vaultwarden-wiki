## How to configure Vaultwarden

There are basically three different ways to configure Vaultwarden:

1. setting up environment variables,
2. using an `ENV_FILE` and
3. via a `config.json` (which can be generated and managed via the [[admin page|Enabling-admin-page]]).

You can find a documented list of most configuration options in the [**`.env.template`**](https://github.com/dani-garcia/vaultwarden/blob/main/.env.template) file. Typically the commented values will indicate the default values but that is not a guarantee. In case it's not, the source of truth will be [**`src/config.rs`**](https://github.com/dani-garcia/vaultwarden/blob/main/src/config.rs).

If you enable the [[admin page|Enabling-admin-page]], you can also see the configuration options with the configured values (and if you are using `config.json` an indication whether the value has been changed from the initial value).

> ⚠️ **NOTE:** Be aware that the `config.json` file is _**NOT**_ the recommended way to configure your settings!
> Either use environment variables which you can configure in several ways for your container environment (Docker, Docker-Compose, K8s etc..)
> Or, when using a standalone binary (not distributed by Vaultwarden itself) use a `.env` file located in your current working directory.

If you rely on a [third-party package](https://github.com/dani-garcia/vaultwarden/wiki/Third-party-packages) you will have to check the provided documentation (e.g. [installation notice](https://gitlab.archlinux.org/archlinux/packaging/packages/vaultwarden/-/blob/main/vaultwarden.install?ref_type=heads) of Arch Linux' `vaultwarden` package), as the downstream maintainers usually make some assumptions for their package.

### Using environment variables

The recommended way to configure Vaultwarden is via environment variables. Depending on how you run Vaultwarden (e.g. directly, in a containerized environment, via systemd, etc.), there are different ways to set the environment variables, so familiarize yourself with your platform and method of installation.

Most environment variables that can be set are found in the `.env.template` file. You can also use that file as a basis for an environment file for your container environment (e.g. via an [`env_file`](https://docs.docker.com/compose/environment-variables/set-environment-variables/#use-the-env_file-attribute) attribute) or with a systemd service (c.f. [`EnvironmentFile=`](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#EnvironmentFile=)) – just don't confuse this file with [the `ENV_FILE` method](#using-an-env_file) below!

> :information_source:  Be aware that there might be some subtle differences between the different platforms for how an environment file is interpreted (in regards to variable expansion or whether you can or should use quotation marks around the values, etc).

You also need to make sure that you set the variable in the <a id="correct-environment">correct environment</a>. If you use a containerized environment the `vaultwarden` process will be running isolated from the host platform. This is especially relevant if you use a container management platform that you can set environment variables for (e.g. when using `docker-compose`). Because typically those environment variables can then be used in the creation of a container but they will not be passed to down into the running container.

> :warning: **NOTE:** A container configured like this with environment variables needs to be recreated if you change a value because the values are bound to the container. So unless the value is [read from a (changed) file](#loading-individual-values-from-files) a restart will not do anything.

### Using an `ENV_FILE`

Vaultwarden can also directly read the configuration options from an environment file itself, which is especially useful when developing Vaultwarden.

By default Vaultwarden will try to read a file called `.env` from the current working directory (e.g. if you run `cargo run` from the root of the checked-out repository it should be in the same root directory).

> :information_source:  There is a difference between using an environment file to setup the runtime environment of the process (whether with something like docker or systemd) and using an env file in Vaultwarden. E.g. instead of passing an environment file [via `--env-file`](https://docs.docker.com/compose/environment-variables/set-environment-variables/#substitute-with---env-file) (which is read when the container is created) you could also mount the environment file to `/.env` when using a [container image](https://github.com/dani-garcia/vaultwarden/wiki/Which-container-image-to-use). (C.f. [the explanation](#correct-environment) above.)

The values that are set directly in the environment will take precedence over this method. That means the values can be overridden without changing the values in the `ENV_FILE` (which might be useful for debugging purposes, e.g. when you temporarily set `LOG_LEVEL=debug`).

### Loading individual values from files

Vaultwarden supports loading the values of the configuration options (both by environment variables or if set in the `ENV_FILE`) from the disk. You can achieve this by adding `_FILE` to the configuration option in question and setting the value to the path to a file containing the value.

This is useful if you want to use a feature like `docker secrets`, e.g. by setting `SMTP_PASSWORD_FILE=/run/secrets/smtp_password` it would load SMTP password from the file without making it available to the process or the container as an environment variable).

### Using the `/admin` page

To an extend, Vaultwarden can also be configured using a `config.json` file, which can be generated and edited over the `/admin` panel and is saved in the data folder.

:pray: While it's technically possible to create and edit the `config.json` file manually, **we strongly advise against it**. [JSON](https://www.json.org/) has a rather strict syntax and if you don't know what you are doing, this might become a nightmare to debug.

The settings in `config.json` will override any other configuration method and you will be warned on startup which settings are overwritten.

Since this generated `config.json` will include **all** editable options when saved, be aware that once you generate the configuration file via the `/admin` page, you cannot modify those options via any of the other methods (at least not without modifying or removing the `config.json` file).

> :warning: **NOTE:** The options in the section `Read-Only Config` **cannot** be modified via the `/admin` page because they require a server restart and **they will be ignored** if you add them manually in the `config.json`. Use the other methods described above to modify them. In most cases this means that you also need to recreate the container!

## Setting the domain URL

Make sure to set the `DOMAIN` environment variable (or `domain` in the config file) to the base URL of your Vaultwarden instance. If you don't, it is likely that some functionality might break mysteriously. Some examples:

* `https://bitwarden.example.com`
* `https://bitwarden.example.com:8443` (non-default port)
* `https://host.example.com/bitwarden/` ([[subdir hosting|Using-an-alternate-base-dir]] -- avoid URL-rewriting tricks whenever possible)

## Further information about different configuration options

- Invitation and Signup settings
  - [[Disable invitations|Disable-invitations]]
  - [[Disable registration of new users|Disable-registration-of-new-users]]
- Administration backend
  - [[Enabling admin page|Enabling-admin-page]]
  - [[Disable the admin token|Disable-admin-token]]
- [[SMTP configuration|SMTP-configuration]]
- Notification
  - [[Enabling WebSocket notifications|Enabling-WebSocket-notifications]]
  - [[Enabling Mobile Client push notification|Enabling-Mobile-Client-push-notification]]
- 2FA Settings
  - [[Enabling U2F and FIDO2 WebAuthn authentication|Enabling-U2F-(and-FIDO2-WebAuthn)-authentication]]
  - [[Enabling YubiKey OTP authentication|Enabling-Yubikey-OTP-authentication]]
- [[Logging|Logging]]
- [[Other configuration|Other-configuration]]
  - [[Changing persistent data location|Changing-persistent-data-location]]
  - [[Changing the API request size limit|Changing-the-API-request-size-limit]]
  - [[Changing the number of workers|Changing-the-number-of-workers]]
- [[Translating the email templates]]

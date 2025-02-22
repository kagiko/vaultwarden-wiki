Vaultwarden provides a single Docker image with unified support for SQLite, MySQL, and PostgreSQL database backends, as of version 1.17.0. Prior to that version, there were separate images for each database backend (see [Historical images](#historical-images)).

The `vaultwarden/server` image is also multi-arch, meaning it supports multiple CPU architectures under a single image name. Assuming you're running one of the supported architectures, simply pulling `vaultwarden/server` should automatically yield the appropriate arch-specific image for your environment. If you are using an ARMv6 board, such as Raspberry Pi 1 and Zero, you must be running Docker 20.10.0 or later for this to work (see [moby/moby#41017](https://github.com/moby/moby/issues/41017)).

The SQLite backend is the most widely used/tested, and recommended for most users unless there is a specific need to use a different database backend.

### Container Registries

Official build images are available on 3 different container registries.

- GitHub: [ghcr.io/dani-garcia/vaultwarden](https://github.com/dani-garcia/vaultwarden/pkgs/container/vaultwarden)
- Docker Hub: [docker.io/vaultwarden/server](https://hub.docker.com/r/vaultwarden/server)
- Quay: [quay.io/vaultwarden/server](https://quay.io/repository/vaultwarden/server)

### Image tags

The `vaultwarden/server` image has several tags, each of which represents some variant or property (e.g., specific version) of the image.

* `latest` -- Tracks the latest released version (i.e., tagged with a version number). This tag is recommended for most users, as it's generally the most stable.

* `testing` -- Tracks the latest commits to the source repository. This tag is recommended for users who want early access to the newest features, enhancements, or bug fixes. The testing version is generally pretty stable, but occasional issues are unavoidable.

* `x.y.z` (e.g., `1.30.0`) -- Represents a specific released version.

* `latest-alpine` -- this image is functionally the same as `latest`, but Alpine-based rather than Debian-based, resulting in a slimmer image and newer base applications. Therefore, `latest` vs. `latest-alpine` is mostly a matter of preference.

* `x.y.z-alpine` (e.g., `1.30.0-alpine`) -- Similar to `latest-alpine`, but represents a specific released version.

## Image updates

Occasionally, the upstream Bitwarden project (i.e., Bitwarden Inc.) makes backward-incompatible changes to the clients that require matching changes to the server implementation. Vaultwarden generally pushes out a new release promptly to handle these changes.

However, since upstream controls the release of the clients, and mobile apps and browser extensions typically auto-update on their own, it's important for Vaultwarden users to keep up-to-date with the latest Vaultwarden release. Otherwise, incompatible client and server versions can lead to sudden breakage or misbehavior.

The web vault is the only exception; as it's bundled with the Vaultwarden image, the web vault version is always properly matched to the Vaultwarden server version. If you only use the web vault as the client (unlikely), then you don't need to worry about these compatibility issues.

## Historical images

Prior to the addition of multi-database support in version 1.17.0, MySQL and PostgreSQL support was only included in separate database-specific images. You can still find these in Docker Hub, and they are still updated for now. However, the database-specific images will be removed in the future, so you should transition to using the unified `vaultwarden/server` image.

* [`bitwardenrs/server-mysql`](https://hub.docker.com/r/bitwardenrs/server-mysql) - Debian-based `vaultwarden` image that includes support for MySQL only (not SQLite or PostgreSQL).
* [`bitwardenrs/server-postgresql`](https://hub.docker.com/r/bitwardenrs/server-postgresql) - Debian-based `vaultwarden` image that includes support for PostgreSQL only (not SQLite or MySQL).

## Historical tags

Prior to the addition of multi-arch image support in version 1.16.0, all arch-specific images had individual arch-specific tags. As of 2021-01-14, these tags have been removed, since many users still ended up pulling these old tags due to following outdated tutorials or not reading the release notes.

* `raspberry` - armv7hf image that should run on Raspberry Pi 2 or newer and possibly on any other compatible boards. This image won't run on Raspberry Pi 1 or Raspberry Pi Zero as those use armv6 CPU.

* `armv6` - Armv6 image for Raspberry Pi 1 and Raspberry Pi Zero.

* `aarch64` - Aarch64 image, that should run on ARMv8 devices like Raspberry Pi 3 or possibly other ARMv8 based devices.

  **Note** that this will also require aarch64 distribution installed on your device, so for example if you use Raspbian on Raspberry Pi 3, you still need to use the `raspberry` tag as Raspbian is an `armv7hf` distribution.

## Reported compatibility table

Please add your details here, if you're running the image on a hardware that is not already in the table.
Note that some images mentioned here are no longer tagged like mentioned above.

| Hardware used        | OS           | Docker architecture reported    | Image used          | Status | Notes |
|----------------------|--------------|---------------------------------|---------------------|--------|-------|
| Regular 64bit server | Ubuntu 18.04 | x86_64                          | `vaultwarden/server` | OK     |       |
| Raspberry Pi Zero W  | Raspbian (4.14.98+) | linux/arm (armv6l)       | `vaultwarden/server:armv6` | OK |     |
| Raspberry Pi Zero W  | Raspbian (4.19.66+) | linux/arm (armv6l)       | `vaultwarden/server:latest` (Multiarch) | OK | Only when using the docker experimental feature 'docker pull --platform=linux/arm/v6'. Otherwise the wrong image will be selected (https://github.com/dani-garcia/vaultwarden/issues/1064) |
| Raspberry Pi 1 B     | Raspbian (4.19.97+) | linux/arm (armv6l)       | `vaultwarden/server:armv6` | OK |     |
| Raspberry Pi 3 B     | Raspbian (4.14.98-v7+) | linux/arm (armv7l)    | `vaultwarden/server:latest` | OK |     |
| Raspberry Pi 4    | Raspbian (4.19.118-v7l+) | linux/arm (armv7l)    | `vaultwarden/server:raspberry` | OK | 4go version, rev 1.1   |
| Raspberry Pi 5    | Raspberry Pi OS (Debian GNU/Linux 12) | linux/arm (arm64)    | `vaultwarden/server:latest` and `vaultwarden/server:testing-alpine` | OK | Tested on 02/16/2024 |
| Synology             | DSM (DSM 6.2.1-23824 Update 6) | Docker-x64-17.05.0-0367 | `vaultwarden/server:latest` | OK |
| Synology             | DSM (DSM 7.2-64570 Update 1) | Docker-20.10.23 | `vaultwarden/server:latest` | OK |
| Synology             | DSM (DSM 6.2.2-24922 Update 4) | Docker-x64-18.09.0-0506 | `vaultwarden/server:1.13.0-alpine` | OK |
| Regular 64bit server | Unraid 6.8.0 | 19.03.5                         | `vaultwarden/server:latest` | OK |
| QNAP TS-451DEU (Intel Celeron J4025) | QTS 5.0.0.1891 | x86_64                         | `vaultwarden/server:latest` | OK |     |
| Regular 64 bit server | OpenMediaVault 6 (Debian Bullseye) | x86_64 | `vaultwarden/server:alpine` | OK |  |
| Oracle Cloud Infrastructure (OCI) Ampere Altra | Ubuntu 24.04 LTS | linux/arm (arm64) | `vaultwarden/server:latest` | OK |  |

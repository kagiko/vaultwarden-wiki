For Docker and Podman users, `bitwarden_rs` provides three images, one for each supported database backend. Each image also has several available tags. The images and tags are described below. 

## Images

* `bitwardenrs/server` -- Debian-based image with SQLite support. This image is the most widely used/tested, and recommended for most users unless there is a specific need to use a different database backend. This image supports `amd64` (x86-64), `arm32v6`, `arm32v7`, and `arm64v8` architectures.

* `bitwardenrs/server-mysql` -- Debian-based image with MySQL support. This image currently supports the `amd64` architecture only, due to cross-compilation issues (see [#530](https://github.com/dani-garcia/bitwarden_rs/issues/530)).

* `bitwardenrs/server-postgresql` -- Debian-based image with PostgreSQL support. This image currently supports the `amd64` architecture only, due to cross-compilation issues (see [#530](https://github.com/dani-garcia/bitwarden_rs/issues/530)).

`bitwardenrs/server` (i.e., the SQLite image) is a [multi-arch](https://www.docker.com/blog/multi-arch-all-the-things/) image, meaning it supports multiple CPU architectures under a single image name. Assuming you're running one of the supported architectures, simply pulling `bitwardenrs/server` should automatically yield the appropriate arch-specific image for your environment, with the exception of Armv6 boards, such as Raspberry Pi 1 and Zero (see [moby/moby#41017](https://github.com/moby/moby/issues/41017)). Armv6 users must specify `arm32v6` in the image tag, e.g. `latest-arm32v6`.

Note that the MySQL and PostgreSQL images currently only support `amd64`, so they are not multi-arch. If someone steps up to address the cross-compilation issues, these images can be made multi-arch as well.

## Image tags

Each Docker image can have various tags, each of which represents some variant or property (e.g., specific version) of the image.

### Common tags

All three (SQLite, MySQL, and PostgreSQL) images have the following tags:

* `latest` -- Tracks the latest released version (i.e., tagged with a version number). This tag is recommended for most users, as it's generally the most stable.

* `testing` -- Tracks the latest commits to the source repository. This tag is recommended for users who want early access to the newest features or enhancements. The testing version is generally pretty stable, but occasional issues are unavoidable.

* `x.y.z` (e.g., `1.16.0`) -- Represents a specific released version.

### SQLite-specific tags

The SQLite image (`bitwardenrs/server`) has the following additional tags:

* `alpine` -- Functionally the same as `latest`, but Alpine-based rather than Debian-based, resulting in a slimmer image. `latest` vs. `alpine` is mostly a matter of preference, but note that the `alpine` tag currently supports the `amd64` architecture only.

* `x.y.z-alpine` (e.g., `1.16.0-alpine`) -- Similar to `alpine`, but represents a specific released version.

* `latest-arm32v6` -- Same as `latest`, but explicitly denotes the `arm32v6` image. This is currently required for users of Armv6 boards, such as Raspberry Pi 1 and Zero. Otherwise, Docker will attempt to pull the `arm32v7` image, which won't work (see [moby/moby#41017](https://github.com/moby/moby/issues/41017)).

* `testing-arm32v6` -- Same as `testing`, but explicitly denotes the `arm32v6` image.

* `x.y.z-arm32v6` (e.g., `1.16.0-arm32v6`) -- Similar to `latest-arm32v6`, but represents a specific released version.

## Image updates

Occasionally, the upstream Bitwarden project (i.e., Bitwarden Inc.) makes backward-incompatible changes to the clients that require matching changes to the server implementation. bitwarden_rs generally pushes out a new release promptly to handle these changes.

However, since upstream controls the release of the clients, and mobile apps and browser extensions typically auto-update on their own, it's important for bitwarden_rs users to keep up-to-date with the latest bitwarden_rs release. Otherwise, incompatible client and server versions can lead to sudden breakage or misbehavior.

The web vault is the only exception; as it's bundled with the bitwarden_rs image, the web vault version is always properly matched to the bitwarden_rs server version. If you only use the web vault as the client (unlikely), then you don't need to worry about these compatibility issues.

## Historical tags

Prior to the addition of multi-arch image support, all arch-specific images had individual arch-specific tags. You can still find these in Docker Hub for older versions of bitwarden_rs, but they shouldn't be used anymore:

* `raspberry` - Armv7hf image that should run on Raspberry Pi 2 or newer and possibly on any other compatible boards. This image won't run on Raspberry Pi 1 or Raspberry Pi Zero as those use armv6 CPU.

* `armv6` - Armv6 image for Raspberry Pi 1 and Raspberry Pi Zero.

* `aarch64` - Aarch64 image, that should run on ARMv8 devices like Raspberry Pi 3 or possibly other ARMv8 based devices.

  **Note** that this will also require aarch64 distribution installed on your device, so for example if you use Raspbian on Raspberry Pi 3, you still need to use the `raspberry` tag as Raspbian is an `armv7hf` distribution.

## Reported compatibility table

Please add your details here, if you're running the image on a hardware that is not already in the table.

| Hardware used        | OS           | Docker architecture reported    | Image used          | Status | Notes |
|----------------------|--------------|---------------------------------|---------------------|--------|-------|
| Regular 64bit server | Ubuntu 18.04 | x86_64                          | `bitwardenrs/server` | OK     |       |
| O-Droid HC2          | Armbian      | arm7l (arm32)                   | `registry.lollipopcloud.solutions/arm32v7/bitwarden` (see notes) | OK | Unofficial image built from upstream sources ; `bitwardenrs/server:raspberry` is the official equivalent image |
| Raspberry Pi Zero W  | Raspbian (4.14.98+) | linux/arm (armv6l)       | `bitwardenrs/server:armv6` | OK |     |
| Raspberry Pi Zero W  | Raspbian (4.19.66+) | linux/arm (armv6l)       | `bitwardenrs/server:latest` (Multiarch) | OK | Only when using the docker experimental feature 'docker pull --platform=linux/arm/v6'. Otherwise the wrong image will be selected (https://github.com/dani-garcia/bitwarden_rs/issues/1064) |
| Raspberry Pi 1 B     | Raspbian (4.19.97+) | linux/arm (armv6l)       | `bitwardenrs/server:armv6` | OK |     |
| Raspberry Pi 3 B     | Raspbian (4.14.98-v7+) | linux/arm (armv7l)    | `bitwardenrs/server:raspberry` | OK |     |
| Raspberry Pi 4    | Raspbian (4.19.118-v7l+) | linux/arm (armv7l)    | `bitwardenrs/server:raspberry` | OK | 4go version, rev 1.1   |
| Synology             | DSM (DSM 6.2.1-23824 Update 6) | Docker-x64-17.05.0-0367 | `bitwardenrs/server:latest` | OK |
| Synology             | DSM (DSM 6.2.2-24922 Update 4) | Docker-x64-18.09.0-0506 | `bitwardenrs/server:1.13.0-alpine` | OK |
| Regular 64bit server | Unraid 6.8.0 | 19.03.5                         | `bitwardenrs/server:latest` | OK |     |

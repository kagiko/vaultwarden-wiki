> :warning: This page is an index of third-party vaultwarden packages.<br>
> As these packages are not maintained or controlled by vaultwarden, they can lag behind official releases, sometimes significantly. If you rely on these packages, you might want to [enable watching](https://docs.github.com/en/github/managing-subscriptions-and-notifications-on-github/viewing-your-subscriptions#configuring-your-watch-settings-for-an-individual-repository) for new vaultwarden releases and let the maintainer know if the package isn't being kept up to date.

***

### Unofficial Packaging Status
| Server | Web-Vault |
|--------|-----------|
|[![Packaging status](https://repology.org/badge/vertical-allrepos/vaultwarden.svg)](https://repology.org/project/vaultwarden/versions)|[![Packaging status](https://repology.org/badge/vertical-allrepos/vaultwarden-web.svg)](https://repology.org/project/vaultwarden-web/versions)|

> :warning: Be aware that the latest vaultwarden release is not always forward-compatible with the latest web-vault version, so you might want to [use an older version](https://github.com/dani-garcia/bw_web_builds/releases) of vaultwarden-web to ensure compatibility.<br />


## Arch Linux

Available in the [official repositories](https://archlinux.org/packages/community/x86_64/vaultwarden/), along with the [web vault](https://archlinux.org/packages/community/any/vaultwarden-web/).

## Debian

A docker based toolchain can be used to build debian packages: https://github.com/greizgh/vaultwarden-debian
It bundles the server and the web vault.

Debian source with pure compilation toolchain (no docker): https://github.com/dionysius/vaultwarden-deb and https://github.com/dionysius/vaultwarden-web-vault-deb
It offers prebuild packages for latest Ubuntu LTS and Debian stable.

## DietPi (Highly optimised minimal Debian OS)

[DietPi](https://dietpi.com/) is a lightweight Debian-based distribution (image) for all kinds of devices like Raspberry Pi, Odroid, NanoPi and others. It offers a software script for installing various programs including Vaultwarden. That spares the user tinkering with installation commands.

For installing Vaultwarden on DietPi just type `dietpi-software install 183` on the command line. More information about the installation process and first access to Vaultwarden on DietPi can be found at https://dietpi.com/docs/software/cloud/#vaultwarden

## CentOS 8 / RHEL 8

A hacky package that uses SQLite. It doesn't have the vault (yet), and still has the old name in visible places.

https://github.com/alexpdp7/vaultwarden-rpm

## Fedora (current release, x86_64)

The vaultwarden package is built as a universal binary for SQLite, MySQL, and PostgreSQL. It also creates a `vaultwarden` user/group and a systemd service.

```
dnf config-manager --add-repo https://evermeet.cx/pub/repo/fedora/evermeet.repo
dnf install vaultwarden vaultwarden-webvault
```

## Nix (OS)

Vaultwarden is both packaged for mysql, sqlite, postgresql and for vault. There is also a NixOS module for declarative configuration (see `services.vaultwarden`)

## Cloudron

[Cloudron](https://cloudron.io) is a platform that helps you run web apps on your server. 
Using Cloudron, you can easily install Bitwarden_rs on a custom domain from the [App Library](https://cloudron.io/store/com.github.bitwardenrs.html)
The app package comes bundled with the upstream web vault and does not need any further configuration after installation to get started. The Cloudron team keeps track of releases and provides automatic updates.

The package code and the issue tracker can be found at https://git.cloudron.io/cloudron/vaultwarden-app
 
## Home Assistant

[Home Assistant](https://www.home-assistant.io/) is an open-source home automation platform. A bitwarden_rs community add-on is available at https://github.com/hassio-addons/addon-bitwarden.

## Build script for Ubuntu 20.04

Dinger1986 has created a script to install bitwarden_rs from source on Ubuntu 20.04, see
https://github.com/dinger1986/bitwardenrs_install_script

## FreeBSD

Available in the [FreeBSD ports tree](https://www.freshports.org/security/vaultwarden/) and as a binary package in the FreeBSD pkg repository: `pkg install vaultwarden`

A sample configuration file is provided at `/usr/local/etc/rc.conf.d/vaultwarden.sample`. Copy this file to `/usr/local/etc/rc.conf.d/vaultwarden` and edit its content to [configure vaultwarden](https://github.com/dani-garcia/vaultwarden/wiki/Configuration-overview#configuration-options). The `vaultwarden` service can then be launched as usual (`service(8)`, etc.).

## Syncloud

[Syncloud](https://syncloud.org) is a self-hosting platform to help people with even no device administration experience to get popular services running on their devices.

Bitwarden is available for installation in app store on the device and requires no configuration.

## RPM and DEB packages for most common distributions

openSUSE build service project with support for:

| RPM    |                                 |
|--------|---------------------------------|
| SUSE   | 15.4<br>15.5<br>Tumbleweed      |
| RHEL   | 7*<br>8                         |
| CentOS | 7*<br>8<br>8_Stream<br>9_Stream |
| Fedora | 36<br>37<br>38<br>Rawhide       |

_(* Only up to vaultwarden-1.28.0 because GCC-4.9 is not available. )_

| DEB    |                                  |
|--------|----------------------------------|
| Debian | 10<br>11<br>12<br>Testing        |
| Ubuntu | 18.04<br>20.04<br>22.04<br>23.04 |

You can either download the packages directly or use the available repositories.

[vaultwarden](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden)
[vaultwarden-webvault](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden-webvault)
[vaultwarden-webvault-dark](https://build.opensuse.org/package/show/home:Masgalor:Vaultwarden/vaultwarden-webvault-dark)

## RHEL 9 / CentOS 9

Packages built for EL9 from the Masgalor packages:

https://rpm.awx.wiki/vaultwarden/

New versions are automatically built every night

## Void Linux

Available in void-packages as [vaultwarden](https://github.com/void-linux/void-packages/tree/master/srcpkgs/vaultwarden): `xbps-install vaultwarden`  
The web vault ([vaultwarden-web](https://github.com/void-linux/void-packages/tree/master/srcpkgs/vaultwarden-web)) can be optionally installed as well: `xbps-install vaultwarden-web`

## Snapcraft

Available as a [snap](https://github.com/DownThePark/snapcraft-vaultwarden) through the [Snap Store](https://snapcraft.io/vaultwarden). The web-vault is also included and enabled by default.

Vaultwarden can be installed on the command line using: `snap install vaultwarden`

Its configuration file is located at: `/var/snap/vaultwarden/current/.env`
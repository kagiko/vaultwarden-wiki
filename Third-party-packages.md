This page is an index of third-party bitwarden_rs packages. As these packages are not maintained or controlled by bitwarden_rs, they can lag behind official releases, sometimes significantly. If you rely on these packages, you might want to [enable watching](https://docs.github.com/en/github/managing-subscriptions-and-notifications-on-github/viewing-your-subscriptions#configuring-your-watch-settings-for-an-individual-repository) for new bitwarden_rs releases and let the maintainer know if the package isn't being kept up to date.

## Arch Linux

Available in the [official repositories](https://www.archlinux.org/packages/community/x86_64/bitwarden_rs/), and the vault is available via the [AUR](https://aur.archlinux.org/pacakges/bitwarden_rs-vault).

## Debian

A docker based toolchain can be used to build debian packages: https://github.com/greizgh/bitwarden_rs-debian
It bundles the server and the web vault.


## CentOS7 / RHEL7

A RPM Repository is packaged by @MrMEEE here: https://copr.fedorainfracloud.org/coprs/mrmeee/bitwarden_rs/ ... This also includes the webinterface in an additional package. 

Installation instructions: https://github.com/MrMEEE/bitwarden_rs_rpm/blob/master/README.md

Any issues with the RPMs can be reported here: https://github.com/MrMEEE/bitwarden_rs_rpm/issues

## Nix (OS)

Bitwarden_rs is packacked in Nix with 4 packages (one for mysql, sqlite and postgresql and one for the vault). For NixOS there's a Module as well (services.bitwarden_rs) so bitwarden_rs can be configured in NixOS declarative way as well. 


## Cloudron

[Cloudron](https://cloudron.io) is a platform that helps you run web apps on your server. 
Using Cloudron, you can easily install Bitwarden_rs on a custom domain from the [App Library](https://cloudron.io/store/com.github.bitwardenrs.html)
The app package comes bundled with the upstream web vault and does not need any further configuration after installation to get started. The Cloudron team keeps track of releases and provides automatic updates.

The package code and the issue tracker can be found at https://git.cloudron.io/cloudron/bitwardenrs-app
 
## Home Assistant

[Home Assistant](https://www.home-assistant.io/) is an open-source home automation platform. A bitwarden_rs community add-on is available at https://github.com/hassio-addons/addon-bitwarden.

## Build script for Ubuntu 20.04
Dinger1986 has created a script to install bitwarden_rs from source on Ubuntu 20.04, see
https://github.com/dinger1986/bitwardenrs_install_script


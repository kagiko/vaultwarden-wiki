This page is an index of standalone deployment examples. If adding a new example, please create a new category if appropriate, and keep things organized in general.

## Google Cloud

* https://github.com/dadatuputi/bitwarden_gcloud

  Bitwarden installation optimized for Google Cloud's 'always free' f1-micro compute instance

## Kubernetes

* https://github.com/icicimov/kubernetes-bitwarden_rs

  Sets up a fully functional and secure `bitwarden_rs` application in Kubernetes behind [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) and AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products). It provides a little bit more than just simple deployment but you can use all or just part of the manifests depending on your needs and setup.

* https://github.com/Skeen/helm-bitwarden_rs

  Sets up a fully functional and secure `bitwarden_rs` application in Kubernetes behind an nginx controller of your choice. It works well and is tested with the [microk8s](https://microk8s.io/) setup. There is support for generating SSL certificates via [cert-manager](https://github.com/jetstack/cert-manager) too.

## Raspberry Pi

* https://github.com/martient/bitwardenrs-ansible

  Ansible deployment for bitwarden_rs on raspberry pi

## Shared hosting

* https://github.com/jjlin/bitwardenrs-shared-hosting

  Sample config for running `bitwarden_rs` on [DreamHost](https://www.dreamhost.com/), but should be readily adaptable to many other shared hosting services.

## NixOS 

* https://git.litschi.xyz/litschi/nixos/src/commit/9eff8b967d23c2a31bb25682448a2c387df2df92/machines/litschi.xyz/modules/bitwarden.nix

  There's a example bitwarden config for NixOS. It's not very complex, you have the backend option, for the type of Database you wanna use, the Backupdir for a dedicated Backup systemdserive, the option to enable it and the config Option. For the Config Option you simply pass the .env Variables [from the .env template](https://github.com/dani-garcia/bitwarden_rs/blob/1.13.1/.env.template) in nix syntax.
See [Proxy Examples](https://github.com/dani-garcia/bitwarden_rs/wiki/Proxy-examples) for a nixos-nginx example config.
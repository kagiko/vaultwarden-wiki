This page is an index of standalone deployment examples. If adding a new example, please create a new category if appropriate, and keep things organized in general.

## Google Cloud

* https://github.com/dadatuputi/bitwarden_gcloud

  Vaultwarden installation optimized for Google Cloud's 'always free' e2-micro compute instance

## Kubernetes

* https://github.com/icicimov/kubernetes-bitwarden_rs

  Sets up a fully functional and secure `vaultwarden` application in Kubernetes behind [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) and AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products). It provides a little bit more than just simple deployment but you can use all or just part of the manifests depending on your needs and setup.

* https://github.com/Skeen/helm-bitwarden_rs

  Sets up a fully functional and secure `vaultwarden` application in Kubernetes behind an nginx controller of your choice. It works well and is tested with the [microk8s](https://microk8s.io/) setup. There is support for generating SSL certificates via [cert-manager](https://github.com/jetstack/cert-manager) too.

## Raspberry Pi

* https://github.com/martient/vaultwarden-ansible

  Ansible deployment for vaultwarden on raspberry pi, for migrate from the previous configuration, follow this guide https://martient.medium.com/migrate-from-bitwarden-rs-to-vaultwarden-199aeb6927a3


## Shared hosting

* https://github.com/jjlin/vaultwarden-shared-hosting

  Sample config for running `vaultwarden` on [DreamHost](https://www.dreamhost.com/), but should be readily adaptable to many other shared hosting services.

* https://lab.uberspace.de/guide_bitwarden.html

  Instructions on how to install from source and run on [Uberspace](https://uberspace.de/en/) shared hosting provider.


## NixOS (by tklitschi)
  There's a example bitwarden config for NixOS. It's not very complex, you have the backend option, for the type of Database you wanna use, the Backupdir for a dedicated Backup systemdserive, the option to enable it and the config Option. For the Config Option you simply pass the .env Variables [from the .env template](https://github.com/dani-garcia/vaultwarden/blob/1.13.1/.env.template) in nix syntax.
See [Proxy Examples](https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples) for a nixos-nginx example config.
<details>
<summary>Example Config</summary><br/>

```nix
{ pkgs, ... }:
{
  services.bitwarden_rs = {
    enable = true;
    backupDir = "/mnt/bitwarden";
    config = {
      WEB_VAULT_FOLDER = "${pkgs.bitwarden_rs-vault}/share/bitwarden_rs/vault";
      WEB_VAULT_ENABLED = true;
      LOG_FILE = "/var/log/bitwarden";
      WEBSOCKET_ENABLED = true;
      WEBSOCKET_ADDRESS = "0.0.0.0";
      WEBSOCKET_PORT = 3012;
      SIGNUPS_VERIFY = true;
      ADMIN_TOKEN = (import /etc/nixos/secret/bitwarden.nix).ADMIN_TOKEN;
      DOMAIN = "https://exmaple.com";
      YUBICO_CLIENT_ID = (import /etc/nixos/secret/bitwarden.nix).YUBICO_CLIENT_ID;
      YUBICO_SECRET_KEY = (import /etc/nixos/secret/bitwarden.nix).YUBICO_SECRET_KEY;
      YUBICO_SERVER = "https://api.yubico.com/wsapi/2.0/verify";
      SMTP_HOST = "mx.example.com";
      SMTP_FROM = "bitwarden@example.com";
      SMTP_FROM_NAME = "Bitwarden_RS";
      SMTP_PORT = 587;
      SMTP_SSL = true;
      SMTP_USERNAME = (import /etc/nixos/secret/bitwarden.nix).SMTP_USERNAME;
      SMTP_PASSWORD = (import /etc/nixos/secret/bitwarden.nix).SMTP_PASSWORD;
      SMTP_TIMEOUT = 15;
      ROCKET_PORT = 8812;
    };
  };

  environment.systemPackages = with pkgs; [
    bitwarden_rs-vault
  ];
}
```

If you have any Questions about this part, feel Free to contact me. I on @litschi:litschi.xyz on matrix an litschi on IRC (hackint and freenode) or simply ask in the vaultwarden matrix.org chanel.

</details>

## QNAP NAS (ARM and x86)

* https://github.com/umireon/vaultwarden-qnap

You can install Vaultwarden into your secure network-attached storage (NAS) with Let's Encrypt.
Due to the QNAP's built-in HTTP(S) server, you cannot publish Vaultwarden on the standard HTTP(S) port (80 / 443).

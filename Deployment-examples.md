This page is an index of standalone deployment examples. If adding a new example, please create a new category if appropriate, and keep things organized in general.

## Self-hosted
This section documents different options to host Vaultwarden on your **own hardware** or any infrastructure that is primarily **managed by yourself**.

### Raspberry Pi

* https://github.com/martient/vaultwarden-ansible

  Ansible deployment for vaultwarden on raspberry pi, for migrate from the previous configuration, follow the guides linked on the page.

* https://dietpi.com/

  [DietPi](https://dietpi.com/) is a lightweight Debian-based distribution (image) for all kinds of devices like Raspberry Pi, Odroid, NanoPi and others. It offers a software script for installing various programs including Vaultwarden. That spares the user tinkering with installation commands.

  For installing Vaultwarden on DietPi just type `dietpi-software install 183` on the command line. More information about the installation process and first access to Vaultwarden on DietPi can be found at [https://dietpi.com/docs/software/cloud/#vaultwarden](https://dietpi.com/docs/software/cloud/#vaultwarden)

* https://mijo.remotenode.io/posts/tailscale-caddy-docker/

  A walkthrough guide for securing access to Vaultwarden with Tailscale and Caddy. All services are containerized and managed with Docker Compose, hosted on a Raspberry Pi.

### Shared hosting

* https://github.com/jjlin/vaultwarden-shared-hosting

  Sample config for running `vaultwarden` on [DreamHost](https://www.dreamhost.com/), but should be readily adaptable to many other shared hosting services.

* https://lab.uberspace.de/guide_vaultwarden.html?highlight=bitwarden

  Instructions on how to install from source and run on [Uberspace](https://uberspace.de/en/) shared hosting provider.

### NixOS (by tklitschi)
  There's a example bitwarden config for NixOS. It's not very complex, you have the backend option, for the type of Database you wanna use, the Backupdir for a dedicated Backup systemdserive, the option to enable it and the config Option. For the Config Option you simply pass the .env Variables [from the .env template](https://github.com/dani-garcia/vaultwarden/blob/1.13.1/.env.template) in nix syntax. Secrets ( SMTP_PASSWORD,... ) store inside another .env file outside /nix/store and include by [services.vaultwarden.environmentFile](https://search.nixos.org/options?channel=21.11&show=services.vaultwarden.environmentFile&from=0&size=50&sort=relevance&type=packages&query=vaultw)
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
 #    ADMIN_TOKEN = (import /etc/nixos/secret/bitwarden.nix).ADMIN_TOKEN;
      DOMAIN = "https://exmaple.com";
 #    YUBICO_CLIENT_ID = (import /etc/nixos/secret/bitwarden.nix).YUBICO_CLIENT_ID;
 #    YUBICO_SECRET_KEY = (import /etc/nixos/secret/bitwarden.nix).YUBICO_SECRET_KEY;
      YUBICO_SERVER = "https://api.yubico.com/wsapi/2.0/verify";
      SMTP_HOST = "mx.example.com";
      SMTP_FROM = "bitwarden@example.com";
      SMTP_FROM_NAME = "Bitwarden_RS";
      SMTP_PORT = 587;
      SMTP_SSL = true;
#     SMTP_USERNAME = (import /etc/nixos/secret/bitwarden.nix).SMTP_USERNAME;
#     SMTP_PASSWORD = (import /etc/nixos/secret/bitwarden.nix).SMTP_PASSWORD;
      SMTP_TIMEOUT = 15;
      ROCKET_PORT = 8812;
    };
    environmentFile = "/etc/nixos/secret/bitwarden.env";
  };
}
```

If you have any Questions about this part, feel Free to contact me. I on @litschi:litschi.xyz on matrix an litschi on IRC (hackint and freenode) or simply ask in the vaultwarden matrix.org chanel.

</details>

### QNAP NAS (ARM and x86)

* https://github.com/umireon/vaultwarden-qnap

You can install Vaultwarden into your secure network-attached storage (NAS) with Let's Encrypt.
Due to the QNAP's built-in HTTP(S) server, you cannot publish Vaultwarden on the standard HTTP(S) port (80 / 443).

### Kubernetes

* https://github.com/icicimov/kubernetes-bitwarden_rs

  Sets up a fully functional and secure `vaultwarden` application in Kubernetes behind [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) and AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products). It provides a little bit more than just simple deployment but you can use all or just part of the manifests depending on your needs and setup.

* https://github.com/Skeen/helm-bitwarden_rs

  Sets up a fully functional and secure `vaultwarden` application in Kubernetes behind an nginx controller of your choice. It works well and is tested with the [microk8s](https://microk8s.io/) setup. There is support for generating SSL certificates via [cert-manager](https://github.com/jetstack/cert-manager) too.

* https://github.com/guerzon/vaultwarden

  Deploy `Vaultwarden` to Kubernetes clusters using [Helm](https://helm.sh/docs/). This chart supports important customizations such as providing image tags and custom registry values, using an external MySQL or PostgreSQL database, using ingress controllers such as [nginx-ingress](https://kubernetes.github.io/ingress-nginx/deploy/) and [AWS LB Ingress Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/installation/), using service accounts, configuring SMTP, and configuring storage options. The chart is well-documented and will continue to introduce more configuration options in the future.

## PaaS Hosting
This section presents different options to host Vaultwarden **in the cloud** or using Platform as a Service providers. 

### Sealos

[![](https://raw.githubusercontent.com/labring-actions/templates/main/Deploy-on-Sealos.svg)](https://cloud.sealos.io/?openapp=system-fastdeploy%3FtemplateName%3Dvaultwarden)

Installs vaultwarden on Sealos using all free addons. Takes about 1 minutes to install. Gracefully handle high concurrency and offer dynamic scalability.

### Google Cloud

* https://github.com/dadatuputi/bitwarden_gcloud

  Vaultwarden installation optimized for Google Cloud's 'always free' e2-micro compute instance

* https://medium.com/@sreafterhours/terraform-helm-external-dns-cert-manager-nginx-and-vaultwarden-on-gke-5080f3b4909f

  Detailed Vaultwarden installation in Google Kubernetes Engine, which includes infrastructure and cluster configuration.

### Heroku

* https://github.com/davidjameshowell/vaultwarden_heroku

  Installs vaultwarden on Heroku using all free addons. Takes about 15 minutes to install.

### Fly.io

* https://github.com/nosovk/vaultwarden-fly-io/blob/main/fly.toml

Installs vaultwarden with SQLite database. But you need to create volume for database
```flyctl volumes create vaultwarden_data -a [your app name] -s 1```

* https://github.com/arthurgeek/vaultwarden-fly-template

Template to deploy Vaultwarden on Fly.io with websockets support (with caddy) and sqlite hourly backups using restic.

### Dokku

This is a script that automatically sets up vaultwarden using the docker image uploaded to DockerHub
and creates a [Dokku](https://dokku.com/) app. The script assumes you have a global domain set
up (i.e. the file `/home/dokku/VHOST` exists). Follow the prompts to set it up.

```sh
#!/usr/bin/env bash

set -euo pipefail

APPNAME=""

read -rp "Enter the name of the app: " APPNAME

# check if app name is empty
if [ -z "$APPNAME" ]; then
    echo "App name empty. Using default name: vaultwarden"
    APPNAME="vaultwarden"
fi

# check if dokku plugin exists
if ! dokku plugin:list | grep letsencrypt; then
    sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
fi
# check if global email for letsencrypt is set
if ! dokku config:get --global DOKKU_LETSENCRYPT_EMAIL; then
    read -rp "Enter email address for letsencrypt: " EMAIL
    dokku config:set --global DOKKU_LETSENCRYPT_EMAIL="$EMAIL"
fi

# pull the latest image
IMAGE_NAME="vaultwarden/server"
docker pull $IMAGE_NAME
image_sha="$(docker inspect --format='{{index .RepoDigests 0}}' $IMAGE_NAME)"
echo "Calculated image sha: $image_sha"
dokku apps:create "$APPNAME"
dokku storage:ensure-directory "$APPNAME"
dokku storage:mount "$APPNAME" /var/lib/dokku/data/storage/"$APPNAME":/data
dokku domains:add $APPNAME $APPNAME."$(cat /home/dokku/VHOST)"
dokku letsencrypt:enable "$APPNAME"
dokku proxy:ports-add "$APPNAME" http:80:80
dokku proxy:ports-add "$APPNAME" https:443:80
dokku proxy:ports-remove "$APPNAME" http:80:5000
dokku proxy:ports-remove "$APPNAME" https:443:5000
dokku git:from-image "$APPNAME" "$image_sha"
```

Copy the above script to your dokku host and run it. Once the script succeeds, the web vault will be
available at `https://$APPNAME.dokku.me`.

To update your vaultwarden server, run the following (remembering to replace `$APP_NAME` with your app's name):

```bash
docker rmi -f vaultwarden/server
docker pull vaultwarden/server:latest
image_sha="$(docker inspect --format='{{index .RepoDigests 0}}' vaultwarden/server)"
dokku git:from-image $APP_NAME $image_sha
```

### Azure

* https://github.com/adamhnat/vaultwarden-azure

  Vaultwarden installation optimized for Azure Container App service with fileshare for data

### Digital Ocean

* https://github.com/HarrisonLeach1/vaultwarden_digitalocean

  Vaultwarden installation for Digital Ocean's cheapest droplet. Resources setup via terraform
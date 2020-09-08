There are two options for deploying on Kubernetes:
* Natively
* Via [Helm](https://helm.sh/)

### Natively:
Please check the [kubernetes-bitwarden_rs](https://github.com/icicimov/kubernetes-bitwarden_rs) repository for example deployment in Kubernetes.

It will setup a fully functional and secure `bitwarden_rs` application in Kubernetes behind [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) and AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products). It provides a little bit more than just simple deployment but you can use all or just part of the manifests depending on your needs and setup.

### Via Helm:
Please check the [helm-bitwarden_rs](https://github.com/Skeen/helm-bitwarden_rs) repository for example deployment in Kubernetes.

It will setup a fully functional and secure `bitwarden_rs` application in Kubernetes behind an nginx controller of your choice. It works well and is tested with the [microk8s](https://microk8s.io/) setup. There is support for generating SSL certificates via [cert-manager](https://github.com/jetstack/cert-manager) too.

Another option with as much, or even more, flexibility would be: https://github.com/gissilabs/charts/tree/master/bitwardenrs
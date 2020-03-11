# Typhoon <img align="right" src="https://storage.googleapis.com/poseidon/typhoon-logo.png">

Typhoon is a minimal and free Kubernetes distribution.

* Minimal, stable base Kubernetes distribution
* Declarative infrastructure and configuration
* [Free](#social-contract) (freedom and cost) and privacy-respecting
* Practical for labs, datacenters, and clouds

Typhoon distributes upstream Kubernetes, architectural conventions, and cluster addons, much like a GNU/Linux distribution provides the Linux kernel and userspace components.

## Features <a href="https://www.cncf.io/certification/software-conformance/"><img align="right" src="https://storage.googleapis.com/poseidon/certified-kubernetes.png"></a>

* Kubernetes v1.17.3 (upstream)
* Single or multi-master, [Calico](https://www.projectcalico.org/) or [flannel](https://github.com/coreos/flannel) networking
* On-cluster etcd with TLS, [RBAC](https://kubernetes.io/docs/admin/authorization/rbac/)-enabled, [network policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
* Advanced features like [worker pools](advanced/worker-pools/), [preemptible](cl/google-cloud/#preemption) workers, and [snippets](advanced/customization/#container-linux) customization
* Ready for Ingress, Prometheus, Grafana, CSI, or other [addons](addons/overview/)

## Modules

Typhoon provides a Terraform Module for each supported operating system and platform.

| Platform      | Operating System | Terraform Module | Status |
|---------------|------------------|------------------|--------|
| AWS           | Container Linux  | [aws/container-linux/kubernetes](cl/aws.md) | stable |
| Azure         | Container Linux  | [azure/container-linux/kubernetes](cl/azure.md) | alpha |
| Bare-Metal    | Container Linux  | [bare-metal/container-linux/kubernetes](cl/bare-metal.md) | stable |
| Digital Ocean | Container Linux  | [digital-ocean/container-linux/kubernetes](cl/digital-ocean.md) | beta |
| Google Cloud  | Container Linux  | [google-cloud/container-linux/kubernetes](cl/google-cloud.md) | stable |

Typhoon is available for [Fedora CoreOS](https://getfedora.org/coreos/).

| Platform      | Operating System | Terraform Module | Status |
|---------------|------------------|------------------|--------|
| AWS           | Fedora CoreOS | [aws/fedora-coreos/kubernetes](fedora-coreos/aws.md) | beta |
| Bare-Metal    | Fedora CoreOS | [bare-metal/fedora-coreos/kubernetes](fedora-coreos/bare-metal.md) | beta |
| Google Cloud  | Fedora CoreOS | [google-cloud/fedora-coreos/kubernetes](google-cloud/fedora-coreos/kubernetes) | alpha |

Typhoon is available for [Flatcar Container Linux](https://www.flatcar-linux.org/releases/).

| Platform      | Operating System | Terraform Module | Status |
|---------------|------------------|------------------|--------|
| AWS           | Flatcar Linux    | [aws/container-linux/kubernetes](cl/aws.md) | stable |
| Azure         | Flatcar Linux    | [azure/container-linux/kubernetes](cl/azure.md) | alpha |
| Bare-Metal    | Flatcar Linux    | [bare-metal/container-linux/kubernetes](cl/bare-metal.md) | stable |
| Google Cloud  | Flatcar Linux  | [google-cloud/container-linux/kubernetes](cl/google-cloud.md) | alpha |
| Digital Ocean | Flatcar Linux  | [digital-ocean/container-linux/kubernetes](cl/digital-ocean.md) | alpha |

## Documentation

* Architecture [concepts](architecture/concepts.md) and [operating-systems](architecture/operating-systems.md)
* Tutorials for [AWS](cl/aws.md), [Azure](cl/azure.md), [Bare-Metal](cl/bare-metal.md), [Digital Ocean](cl/digital-ocean.md), and [Google-Cloud](cl/google-cloud.md)

## Example

Define a Kubernetes cluster by using the Terraform module for your chosen platform and operating system. Here's a minimal example.

```tf
module "yavin" {
  source = "git::https://github.com/poseidon/typhoon//google-cloud/container-linux/kubernetes?ref=v1.17.3"

  # Google Cloud
  cluster_name  = "yavin"
  region        = "us-central1"
  dns_zone      = "example.com"
  dns_zone_name = "example-zone"

  # configuration
  ssh_authorized_key = "ssh-rsa AAAAB3Nz..."

  # optional
  worker_count = 2
}

# Obtain cluster kubeconfig
resource "local_file" "kubeconfig-yavin" {
  content  = module.yavin.kubeconfig-admin
  filename = "/home/user/.kube/configs/yavin-config"
}
```

Initialize modules, plan the changes to be made, and apply the changes.

```sh
$ terraform init
$ terraform plan
Plan: 62 to add, 0 to change, 0 to destroy.
$ terraform apply
Apply complete! Resources: 62 added, 0 changed, 0 destroyed.
```

In 4-8 minutes (varies by platform), the cluster will be ready. This Google Cloud example creates a `yavin.example.com` DNS record to resolve to a network load balancer across controller nodes.

```
$ export KUBECONFIG=/home/user/.kube/configs/yavin-config
$ kubectl get nodes
NAME                                       ROLES    STATUS  AGE  VERSION
yavin-controller-0.c.example-com.internal  <none>   Ready   6m   v1.17.3
yavin-worker-jrbf.c.example-com.internal   <none>   Ready   5m   v1.17.3
yavin-worker-mzdm.c.example-com.internal   <none>   Ready   5m   v1.17.3
```

List the pods.

```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY  STATUS    RESTARTS  AGE
kube-system   calico-node-1cs8z                         2/2    Running   0         6m
kube-system   calico-node-d1l5b                         2/2    Running   0         6m
kube-system   calico-node-sp9ps                         2/2    Running   0         6m
kube-system   coredns-1187388186-dkh3o                  1/1    Running   0         6m
kube-system   coredns-1187388186-zj5dl                  1/1    Running   0         6m
kube-system   kube-apiserver-controller-0               1/1    Running   0         6m
kube-system   kube-controller-manager-controller-0      1/1    Running   0         6m
kube-system   kube-proxy-117v6                          1/1    Running   0         6m
kube-system   kube-proxy-9886n                          1/1    Running   0         6m
kube-system   kube-proxy-njn47                          1/1    Running   0         6m
kube-system   kube-scheduler-controller-0               1/1    Running   0         6m
```

## Help

Ask questions on the IRC #typhoon channel on [freenode.net](http://freenode.net/).

## Motivation

Typhoon powers the author's cloud and colocation clusters. The project has evolved through operational experience and Kubernetes changes. Typhoon is shared under a free license to allow others to use the work freely and contribute to its upkeep.

Typhoon addresses real world needs, which you may share. It is honest about limitations or areas that aren't mature yet. It avoids buzzword bingo and hype. It does not aim to be the one-solution-fits-all distro. An ecosystem of Kubernetes distributions is healthy.

## Social Contract

Typhoon is not a product, trial, or free-tier. It is not run by a company, does not offer support or services, and does not accept or make any money. It is not associated with any operating system or platform vendor.

Typhoon clusters will contain only [free](https://www.debian.org/intro/free) components. Cluster components will not collect data on users without their permission.

## Donations

Typhoon does not accept money donations. Instead, we encourage you to donate to one of [these organizations](https://github.com/poseidon/typhoon/wiki/Donations) to show your appreciation.


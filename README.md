# Kubernetes Setup
Version Kubernetes v1.31

3 Nodes Cluster: 1 master node + 2 worker nodes

## Pre-requesties
My Environment:
> | Nodes    | IP         | OS                   | CPU    | Mem  |
> | -------- |:----------:|:--------------------:|:------:|:----:|
> | `master` | 10.0.0.101 | Ubuntu 24.04 Desktop | 6 Core | 16GB |
> | `node1`  | 10.0.0.102 | Ubuntu 24.04 Desktop | 2 Core | 8GB  |
> | `node2`  | 10.0.0.103 | Ubuntu 24.04 Desktop | 2 Core | 8GB  |

* Update softwares
* Set up passwordless SSH access for root user among the cluster.
* Set up static IP address

## Install Docker
[Docker Installation Guide](https://docs.docker.com/engine/install/ubuntu/)
1. Uninstall old versions
```shell
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
2. Install using the `apt` repository
* Set up Docker's `apt` repository
```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
* Install the Docker packages
```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
* Verify
```shell
sudo systemctl status docker
```
## Install CRI-Docker

## Install kubeadm kubectl kubelet

## Cluster init

## Cluster Monitoring
Kube-Prometheus

## Reverse Proxy

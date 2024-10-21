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
[CRI-Docker Installation](https://github.com/Mirantis/cri-dockerd/releases)

```shell
# Download dpkg file from latest release
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.15/cri-dockerd_0.3.15.3-0.ubuntu-jammy_amd64.deb
# Install deb
sudo dpkg -i cri-dockerd_0.3.15.3-0.ubuntu-jammy_amd64.deb
# Enable service
sudo systemctl enable cri-docker
```
## Install Kubernetes
[Kubernetes Installation Guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Disable swap
```shell
# Disable on current session, will be restored after reboot
sudo swapoff -a
# Permenant disable
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
Install kubeadm kubectl kubelet
```shell
# Add GPG key for kubernetes
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# Add kubernetes repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
# Update local apt repo
sudo apt-get update
# Install Kubernetes packages
sudo apt-get install kubeadm kubelet kubectl -y
# To hold the versions so that the versions will not get accidently upgraded.
sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```
Enable the iptables bridge
```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

Install socat
```shell
sudo apt-get install socat -y
```
## Cluster init
On control panel (master node)
### Init with Calico CIDR
```shell
# Calico network
# Make sure to copy the join command
sudo kubeadm init --apiserver-advertise-address=<control_plane_ip> --cri-socket unix:///var/run/cri-dockerd.sock  --pod-network-cidr=192.168.0.0/16

# Alternate1: Use below command if the node network is 192.168.x.x
sudo kubeadm init --apiserver-advertise-address=<control_plane_ip> --cri-socket unix:///var/run/cri-dockerd.sock  --pod-network-cidr=10.244.0.0/16

# Alternate2: Use below command if the node network is 172.16.x.x
sudo kubeadm init --apiserver-advertise-address=<control_plane_ip> --cri-socket unix:///var/run/cri-dockerd.sock  --pod-network-cidr=172.16.0.0/16

# Make sure your control panel IP is correct
hostname -I | awk '{print $1}'

# If you are controling kubernetes under regular user, need to execute the following commands
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# If you are root user, run as below
export KUBECONFIG=/etc/kubernetes/admin.conf
```

On worker nodes
```shell
# Copy your join command and keep it safe.
# Below is a sample format
# Add --cri-socket /var/run/cri-dockerd.sock to the command
sudo kubeadm join {control_plane_ip}:6443 --token {token} --cri-socket unix:///var/run/cri-dockerd.sock --discovery-token-ca-cert-hash sha256:{hash}
```

## Cluster Monitoring
Kube-Prometheus

## Reverse Proxy

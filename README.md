# RedHat-Based installation

## Set SELinux in permissive mode

```sh
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```
## Enable IPv4 packet forwarding

```sh
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
```
### Verify that net.ipv4.ip_forward is set to 1 with:

```sh
sysctl net.ipv4.ip_forward
```

### Disable swap

```sh
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Firewalld configuration
```sh
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=179/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=2379/tcp
firewall-cmd --permanent --add-port=2380/tcp
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=8443/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10254/tcp
firewall-cmd --reload
```


### Apply sysctl params without reboot
```sh
sudo sysctl --system
```
## Install Docker Engine

```sh
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl enable --now containerd
```

### Installation iproute-tc
```sh
sudo yum install -y iproute-tc
```

## Add the Kubernetes yum repository

```sh
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```

## Install kubelet, kubeadm and kubectl

```sh
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

### Reconfigure containerd

```sh
rm -rf  /etc/containerd/config.toml
systemctl restart containerd.service
```

## Cluster init

```sh
sudo kubeadm init --apiserver-advertise-address=10.159.86.79 --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=10.159.86.79 --v=5
```
## Installation CNI

```sh
curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```
## Setup control-plane

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



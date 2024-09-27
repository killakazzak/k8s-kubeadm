# RedHat-Based installation

## Set SELinux in permissive mode

```sh
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```
## Enable and Verify IPv4 packet forwarding

```sh
cat <<EOF | sudo tee /etc/sysctl.conf
net.ipv4.ip_forward = 1
EOF
echo 1 > /proc/sys/net/ipv4/ip_forward
sysctl net.ipv4.ip_forward
```
### Disable swap

```sh
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Firewalld configuration

```sh
systemctl dislabe --now firewalld
```

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
firewall-cmd --permanent --add-port=4789/udp
firewall-cmd --permanent --add-port=5473/tcp
firewall-cmd --permanent --add-port=6379/tcp
firewall-cmd --permanent --add-port=51820/udp
firewall-cmd --permanent --add-port=51821/udp
firewall-cmd --permanent --add-port=4789/udp
firewall-cmd --permanent --add-port=30000-32767/tcp
firewall-cmd --permanent --add-rich-rule='rule protocol value="4" accept'
firewall-cmd --reload
```


### Apply sysctl params without reboot
```sh
sudo sysctl --system
```
## Install Docker Engine and iproute-tc

```sh
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin iproute-tc
systemctl enable --now containerd
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
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
systemctl restart containerd.service
```

## Cluster init

```sh
sudo kubeadm init --apiserver-advertise-address=10.159.86.79 --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=10.159.86.79 --v=5
```
## Setup admin access

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## Installation CNI

```sh
curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

## Setup Nginx Ingress Controller

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/baremetal/deploy.yaml
```


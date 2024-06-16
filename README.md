# k8s-kubeadm

# Set SELinux in permissive mode (effectively disabling it)

```sh
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

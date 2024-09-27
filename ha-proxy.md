### Installation and Configuration HAProxy, Lsyncd, Keepalived

```sh
yum install -y haproxy lsyncd keepalived
systemctl enable --now haproxy lsyncd keepalived

mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak

cat <<EOF > /etc/haproxy/haproxy.cfg
global
    log /dev/log  local0 warning
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
   stats socket /var/lib/haproxy/stats
defaults
  log global
  option  httplog
  option  dontlognull
        timeout connect 5000
        timeout client 50000
        timeout server 50000
frontend kube-apiserver
  bind *:6443
  mode tcp
  option tcplog
  default_backend kube-apiserver
backend kube-apiserver
    mode tcp
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
    server ismail-ctr101p  10.207.173.121:6443 check
    server ismail-ctr102p  10.207.173.122:6443 check
    server ismail-ctr103p  10.207.173.123:6443 check

EOF

systemctl enable --now haproxy
```

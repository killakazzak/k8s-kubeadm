
### MASTER NODE

```bash
yum install keepalived -y
cat <<EOF > /etc/keepalived/keepalived.conf
vrrp_script chk_haproxy {
    script "killall -0 haproxy"
    interval 2
}

vrrp_instance VI_1 {
    interface ens192
    state MASTER
    priority 100

    virtual_router_id 1
    unicast_src_ip {IP MASTER NODE}
    unicast_peer {
        {IP SLAVE NODE}
    }

    track_script {
        chk_haproxy
    }

        virtual_ipaddress {
        {IP VIP}
    }
}
EOF
systemctl enable --now keepalived
systemctl status keepalived
```
### SLAVE NODE

```bash
yum install keepalived -y
cat <<EOF > /etc/keepalived/keepalived.conf
vrrp_script chk_haproxy {
    script "killall -0 haproxy"
    interval 2
}

vrrp_instance VI_1 {
    interface ens192
    state SLAVE
    priority 90

    virtual_router_id 1
    unicast_src_ip {IP SLAVE NODE}
    unicast_peer {
        {IP MASTER NODE}
    }

    track_script {
        chk_haproxy
    }

        virtual_ipaddress {
       {IP VIP}
    }
}
EOF
systemctl enable --now keepalived
systemctl status keepalived
```

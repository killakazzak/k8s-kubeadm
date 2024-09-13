
```bash
yum install keepalived -y
```


```sh

cat <<EOF > 
vrrp_script chk_haproxy {
    script "killall -0 haproxy"
    interval 2
}

vrrp_instance VI_depr_1 {
    interface ens192
    state MASTER
    priority 100

    virtual_router_id 35
    unicast_src_ip 10.206.123.31
    unicast_peer {
        10.206.123.32
    }

    track_script {
        chk_haproxy
    }

        virtual_ipaddress {
       10.206.123.243
    }
}
EOF
```
```
systemctl enable --now keepalived
systemctl status keepalived
```

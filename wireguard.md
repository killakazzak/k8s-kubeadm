
```
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install wireguard
cat <<EOF > /etc/sysctl.conf
net.ipv4.ip_forward=1
EOF
sudo sysctl -p
```

```sh
sudo apt install ufw
sudo ufw allow ssh
sudo ufw allow 51820/udp
sudo ufw enable
sudo ufw status
```

### Generating private and public keys

```sh
wg genkey | tee /etc/wireguard/privatekey | wg pubkey | tee /etc/wireguard/publickey
```

```sh
cd /etc/wireguard
umask 077
cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
PrivateKey = <privatekey> # Приватный ключ из файла privatekey.
Address = 10.0.0.1/24
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51820

[Peer]
PublicKey = <client-publickey>
AllowedIPs = 10.0.0.2/32
EOF
```

### Starting WireGuard and enabling it at boot
```sh
wg-quick up wg0
systemctl enable wg-quick@wg0
sudo modprobe wireguard
systemctl enable wg-quick@wg0.service
wg show
```

Client configuration


sudo nano /etc/wireguard/wg0.conf

```
[Interface]
Address = 10.0.0.2/32
PrivateKey = <contents-of-client-privatekey>
DNS = 1.1.1.1

[Peer]
PublicKey = <contents-of-server-publickey>
Endpoint = <server-public-ip>:51820
AllowedIPs = 0.0.0.0/0, ::/0
```

sudo wg-quick up wg0

sudo systemctl start wg-quick@wg0

sudo wg-quick down wg0
sudo systemctl stop wg-quick@wg0

Adding more clients

sudo nano /etc/wireguard/wg0.conf

[//]: # (Add the following entry at the end of the file to include your second client’s public key and set the IP address.)
```
[Peer]
PublicKey = <content-of-client2-publickey>
AllowedIPs = 10.0.0.3/32
```

sudo systemctl restart wg-quick@wg0


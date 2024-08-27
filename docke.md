### Docker behind proxy

```bash
mkdir /etc/systemd/system/docker.service.d
```

```
Environment="HTTP_PROXY=http://proxy.example.com:80/"
Environment="HTTPS_PROXY=http://proxy.example.com:80/"
Environment="NO_PROXY=localhost,127.0.0.0/8,docker-registry.somecorporation.com"
```

```bash
sudo systemctl daemon-reload
systemctl show --property Environment docker
systemctl restart docker
```

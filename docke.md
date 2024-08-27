### Docker behind proxy

```bash
mkdir /etc/systemd/system/docker.service.d
```

```bash
sudo bash -c 'echo -e "Environment=\"HTTP_PROXY=http://proxy.example.com:80/\"\nEnvironment=\"HTTPS_PROXY=http://proxy.example.com:80/\"\nEnvironment=\"NO_PROXY=localhost,127.0.0.0/8,docker-registry.somecorporation.com\"" > /etc/systemd/system/docker.service.d/http-proxy.conf'

```

```bash
sudo systemctl daemon-reload
systemctl show --property Environment docker
systemctl restart docker
```

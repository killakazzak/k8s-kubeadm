### Docker behind proxy

```bash
mkdir /etc/systemd/system/docker.service.d
```

```bash
sudo bash -c 'echo -e "[Service]\nEnvironment=\"HTTP_PROXY=${{PROXY}}\"\nEnvironment=\"HTTPS_PROXY=http://10.159.86.102:63128/\"\nEnvironment=\"NO_PROXY=localhost,127.0.0.0/8,oblako.mos.ru,ismail.mos.ru\"" > /etc/systemd/system/docker.service.d/http-proxy.conf'

```

```bash
sudo systemctl daemon-reload
systemctl show --property Environment docker
systemctl restart docker
docker info
```
```
docker login -u $USER
Password: https://app.docker.com/settings/personal-access-tokens
```



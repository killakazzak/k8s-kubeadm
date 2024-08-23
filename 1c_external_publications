```bash
vi /usr/local/openresty/nginx/conf/dev2-balance.mos.ru.conf
```

```nginx
server {
    listen 80;
    server_name dev2-balance.mos.ru;
    include dev2_1c_location.conf;
}
```

```bash
vi /usr/local/openresty/nginx/conf/dev2_1c_location.conf
```

```nginx
proxy_set_header X-Forwarded-Port 443;
proxy_connect_timeout 60s;
proxy_send_timeout    3600;
proxy_read_timeout    3600;
proxy_next_upstream   error;
proxy_set_header      Host              $host;
proxy_set_header      X-Real-IP         $remote_addr;
proxy_set_header      X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header      X-Forwarded-Proto https;
proxy_http_version 1.1;
proxy_set_header Connection "";

location /octopus-pilot {
#  include allow_ip.conf;
#  error_page 403 = @denypage;
#  include 1c_app.conf;
#  include    1c_error.conf;
  proxy_pass  http://m1.web.a1.application.dev2-balance.mos.local;
}
```

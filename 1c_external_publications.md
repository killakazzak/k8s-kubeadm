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
```
vi /usr/local/openresty/nginx/conf/nginx.conf
```

```nginx
#user  openresty;
worker_processes  2;
worker_rlimit_nofile 10240;
error_log  logs/error.log;
pid        logs/nginx.pid;


events {
    worker_connections  4096;
    use                 epoll;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    gzip          on;
    sendfile      on;
    tcp_nopush    on;
    tcp_nodelay   on;
    server_tokens off;

    proxy_buffering          on;
    proxy_buffer_size        32k;
    proxy_buffers            20 512k;
    proxy_connect_timeout    5;
    proxy_max_temp_file_size 0;

    keepalive_timeout             300 300;
    server_names_hash_max_size    4096;
    server_names_hash_bucket_size 128;
    client_max_body_size          4096m;
    client_body_buffer_size       256k;
    client_header_buffer_size     4k;


    log_format  auth    '$time_local\t"$request"\t$status\t"$http_user_agent"\t"$http_cookie"\t"$request_body"';



    log_format  main    '$time_local\t"$request"\t$status\t$bytes_sent\t$request_length\t$request_time\t$proxy_host\t$upstream_addr\t|\t'
                        '$remote_addr\t$remote_user\t'
                        '"$http_referer"\t"$http_user_agent"\t$sent_http_set_cookie\t';

    access_log  logs/access.log  main;

    include /usr/local/openresty/nginx/conf/dev2-balance.mos.ru.conf;
    include /usr/local/openresty/nginx/conf/ssl_dev2-balance.mos.ru.conf;

    set_real_ip_from  10.206.97.0/24;
    real_ip_header    X-Forwarded-For;
    real_ip_recursive on;
#    include    allow_ip.conf;
    #error_page     403 = @denypage;
}
```

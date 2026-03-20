# Default nginx.conf

```
user nginx;
worker_processes auto;

error_log  /var/log/nginx/error.log notice;
pid        /run/nginx.pid;

worker_rlimit_nofile 200000;

events {
  worker_connections  49152;
  multi_accept on;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  log_format main '$remote_addr - $remote_user [$time_local] "$request"'
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"'
                  '$upstream_addr ';

  access_log  /var/log/nginx/access.log  main;
  sendfile        on;
  tcp_nopush     on;
  tcp_nodelay on;
  keepalive_timeout  65;
  keepalive_requests 100000;
  gzip on;
  gzip_types *;
  gzip_min_length 10240;
  gzip_comp_level 6;
  gzip_vary off;
  gzip_proxied any;
  gzip_disable msie6;
  gzip_http_version 1.0;

  include /etc/nginx/conf.d/*.conf;
}
```
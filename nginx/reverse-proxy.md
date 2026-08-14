# Cấu hình reverse proxy

```nginx
upstream frontend_servers {
  server ip_server1;
  server ip_server2;

  keepalive 1024;
  keepalive_timeout 60s;
}

upstream api_servers {
  server ip_server1;
  server ip_server2;

  keepalive 1024;
  keepalive_timeout 60s;
}

server {
  server_name example.com; # server_name .example.com Trong trường hợp muốn handle tất cả subdomain

  large_client_header_buffers 8 32k;

  location / {
    server_tokens off;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection '';
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://frontend_servers;
    proxy_set_header Accept-Encoding gzip;    
 
  }

  location /api/v1 {
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    client_max_body_size 200M;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://api_servers;
  }

  # This helps cloudflare can serve static file without cors
  location ^~ /_next/static/ {
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_set_header Host $host;

    proxy_pass http://frontend_servers;

    add_header Access-Control-Allow-Origin "*" always;
    add_header Cross-Origin-Resource-Policy "cross-origin" always;
  }

  location /socket.io {
    client_max_body_size 10M;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_pass http://api_servers;
    proxy_redirect off;
  }
   
  listen 443 ssl; # managed by Certbot

  ssl_certificate path-fullchain.pem; 
  ssl_certificate_key path.private.pem;
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = example.com) {
        return 301 https://$host$request_uri;
   } # managed by Certbot


  server_name example.com;

  listen 80;
  return 404;
}
```
# Increase max file open nginx

## 1. Kiểm tra giới hạn hiện tại

```bash
ulimit -n
cat /proc/$(pidof nginx | awk '{print $1}')/limits | grep "Max open files"
```

## 2. Tăng giới hạn ở OS (ulimit)
Sửa file:

```bash
sudo nano /etc/security/limits.conf
```

Thêm:
```
* soft nofile 200000
* hard nofile 200000
```
Hoặc nếu chạy bằng user riêng (vd: www-data):

```
www-data soft nofile 200000
www-data hard nofile 200000
```

## 3. Nếu dùng systemd (QUAN TRỌNG)

Nginx chạy bằng systemd nên phải override:

```
sudo systemctl edit nginx
```

Thêm:

```
[Service]
LimitNOFILE=200000
```

Reload:
```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart nginx
```
## 4. Cấu hình trong Nginx

Sửa file nginx config:
```
sudo nano /etc/nginx/nginx.conf
```
Thêm / sửa:

```nginx
worker_rlimit_nofile 200000;

events {
    worker_connections 65535;
    multi_accept on;
}
```

## 5. Tuning OS

Sửa file ```/etc/sysctl.conf```
```
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 10
net.core.somaxconn = 65535
net.ipv4.tcp_timestamps=1 
net.ipv4.tcp_tw_recycle=0
net.ipv4.tcp_max_tw_buckets=10000
```

## 6. Kiểm tra lại sau khi apply
```
cat /proc/$(pidof nginx | awk '{print $1}')/limits | grep "Max open files"
```
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

## 5. Kiểm tra lại sau khi apply
```
cat /proc/$(pidof nginx | awk '{print $1}')/limits | grep "Max open files"
```
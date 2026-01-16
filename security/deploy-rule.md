# Đây là tổng hợp kinh nghiệm deploy sau khi bị nhiễm mã độc
## 1. Deploy checklist

### 1.1. Setup server
- Tạo user mới không có quyền root để chạy app.

- User chạy app không được tạo crontab.

- User chạy app không được sử dụng `curl/wget`.

- Không cho kết nối internet, khi cần mới mở kết nối, **không được mở port 80**.

#### 1.1.1 Setup firewall
***Bật `ufw`***

```bash
sudo ufw allow OpenSSH #Bật ssh phòng trường hợp bị ngắt session
sudo ufw enable
```

Áp rules. Thứ tự rule cực kì quan trọng. Rule nào được khai báo trước sẽ được áp dùng trước.

```bash
# Chặn tất cả kết nối ra ngoài
sudo ufw default deny outgoing 

# Chặn kết nối đến IP nghi ngờ là của hacker
ufw deny out to HACKER_IP
ufw deny in  from HACKER_IP

#Chỉ open khi thực sự cần thiết. Cho phép kết nối tcp qua 443. 
ufw allow out 443/tcp

#Chỉ open khi thực sự cần thiết. Cho phép kết nối đến DNS.
ufw allow out 53
```

#### 1.1.2. Setup user chạy app
***Tạo mới user***

```bash
sudo adduser <username>
```

Không cho phép user đó chạy crontab

***Tạo file `cron.deny`***

```bash
sudo nano /etc/cron.deny
```

Thêm `username` vào file `cron.deny`. Mỗi user là một dòng.

***Chặn `wget/curl`***

Bước này sẽ thực hiện cuối cùng sau khi đã tải các tool cần thiết cho user đó sử dụng.

```bash
sudo chmod 000 /usr/bin/wget /usr/bin/curl
```

### 1.2. Build code
- Build qua `docker`, server build đảm bảo tuyệt đối là server sạch

## 2. Cách kiểm tra khi nghi ngờ server bị nhiễm mã độc
Với server bị nhiễm mã độc sẽ có các dấu hiệu bất thường như sau

- Có crontab lạ.
- Có file lạ.
- Nhiều process lạ đang chạy.
- Không thể truy cập được vào bằng ssh.
- Có request đến địa chỉ IP lạ.

### 2.1. Kiểm tra crontab
- Vào user khác (có sudo), chạy `sudo crontab -u <username> -l`  nếu xuất hiện crontab lạ, khả năng cao là đã dính mã độc.
- `stat /var/spool/cron/crontabs/gtel` kiểm tra thời gian tạo, chỉnh sửa của crontab.
- Xóa crontab `sudo crontab -i <username> -r`

### 2.2. Kiểm tra file lạ
Các file này sẽ thường là các file excutable, để chạy với crontab.

- Kiểm tra trong `/tmp`, `/var/tmp`,... xem có file nào có tên lạ, đặc biệt là xuất hiện các file excutable. Kiểm tra`/home/<username>` các file .profile, .bashrc.. xem có file lạ hay code lạ được inject vào không.


### 2.3. Kiểm tra user session

Dấu hiệu: user bị treo, không connect được đến ssh. Có nhiều session, process lạ.

Đăng nhập vào server bằng user khác, chạy lệnh.

```bash
id <username> # Lấy user id
sudo systemctl status user-<id>.slice
```

Nếu thấy có nhiều session/process lạ. Lập tức kill user.

```bash
pkill -f <username>
```
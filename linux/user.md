# Các lệnh hữu ích liên quan đến user

### Tạo user

`sudo adduser <username>`

### Cấp quyền root

`sudo usermod -aG sudo <username>`

### Gỡ quyền root

Check từng bước, sau mỗi bước thử sudo lại xem ăn không

- Gỡ user khỏi groups sudo `sudo deluser <username> sudo`.
- Kiểm tra user có nằm trong file sudoers không `sudo visudo`. Nếu có xóa user đó khỏi file

### Check process của user đó

1. Lấy id của user `id <username>`
2. Check system `sudo systemctl status user-<userId>.slice` (`userId` được lấy từ lệnh bên trên)
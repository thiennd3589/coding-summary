# 1️⃣ Cài MongoDB trên cả 3 server
## Ubuntu (ví dụ 22.04)
```bash
wget -qO - https://pgp.mongodb.com/server-7.0.asc | sudo apt-key add -

echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

sudo apt update
sudo apt install -y mongodb-org
```

# 2️⃣ Config mongod.conf (QUAN TRỌNG)

## Sửa file:

```bash
sudo nano /etc/mongod.conf
```

## Cấu hình:

```yaml
net:
  port: 27017
  bindIp: 0.0.0.0   # hoặc IP nội bộ

replication:
  replSetName: rs0

security:
  authorization: enabled
```

# 3️⃣ Tạo keyFile (bắt buộc cho production)

## 👉 Trên 1 server (mongo1):

```bash
openssl rand -base64 756 > /opt/mongo-keyfile
chmod 400 /opt/mongo-keyfile
chown mongodb:mongodb /opt/mongo-keyfile
```

## Copy sang 2 server còn lại:

```bash
scp /opt/mongo-keyfile mongo2:/opt/mongo-keyfile
scp /opt/mongo-keyfile mongo3:/opt/mongo-keyfile
```

## Update config thêm:

```yaml
security:
  keyFile: /opt/mongo-keyfile
  authorization: enabled
```

# 4️⃣ Start MongoDB trên cả 3 node

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
```

## Check:

```bash
sudo systemctl status mongod
```

# 5️⃣ Initiate Replica Set (chỉ chạy 1 lần trên mongo1)

```bash
mongosh
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "10.0.0.1:27017" },
    { _id: 1, host: "10.0.0.2:27017" },
    { _id: 2, host: "10.0.0.3:27017" }
  ]
})
```
# 6️⃣ Kiểm tra trạng thái
```bash
rs.status()
```

## 👉 Kỳ vọng:

1 PRIMARY

2 SECONDARY

# 7️⃣ Tạo user admin (bắt buộc vì bật auth)
```bash
use admin

db.createUser({
  user: "admin",
  pwd: "strongpassword",
  roles: [ { role: "root", db: "admin" } ]
})
```

# 8️⃣ Test connect
```bash
mongosh "mongodb://admin:strongpassword@10.0.0.1:27017,10.0.0.2:27017,10.0.0.3:27017/?replicaSet=rs0"
```

# 9️⃣ Một số config production nên có
## 🔥 Priority (control primary)

```bash
rs.conf()
```

## Update:

```bash
cfg = rs.conf()
cfg.members[0].priority = 2
cfg.members[1].priority = 1
cfg.members[2].priority = 1
rs.reconfig(cfg)
```

## 🔥 Hidden node (analytics)
```bash
cfg.members[2].hidden = true
cfg.members[2].priority = 0
```

## 🔥 Arbiter (nếu thiếu node)
```bash
rs.addArb("10.0.0.4:27017")
```
# 1️⃣ 0️⃣ Firewall cần mở

## Trên mỗi server:

```bash 
ufw allow 27017
```

## 👉 Hoặc chỉ allow internal network (khuyến nghị)

## ⚠️ Lỗi hay gặp
### ❌ Node không join được

- Sai keyFile

- Sai permission (phải 400)

### ❌ Replica stuck STARTUP2

- Check network (ping giữa node)

- Check bindIp

### ❌ No primary

- Chưa đủ majority

- Clock lệch (NTP chưa sync)

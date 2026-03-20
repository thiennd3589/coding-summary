# Tuning Redis

Giả định server 8 core 16gb ram

## 🎯 1. Hiểu đúng về Redis (rất quan trọng)

👉 Redis chỉ dùng 1 core chính để xử lý command
→ 16 core KHÔNG = 16x performance

📌 Vì vậy:

- bottleneck = CPU single thread + network

- scale = Redis Cluster hoặc sharding, không phải tăng core

## 🎯 2. Phân bổ RAM chuẩn (32GB)

👉 KHÔNG dùng hết RAM cho Redis

**Khuyến nghị:**

```
Redis maxmemory: 20GB – 24GB
OS + page cache: 8GB – 12GB
```

## 🎯 3. Config Redis chuẩn production
```/etc/redis/redis.conf```

```
################################
# NETWORK
################################
bind 0.0.0.0
protected-mode no
tcp-backlog 65535
timeout 0
tcp-keepalive 300

################################
# CLIENTS
################################
maxclients 200000

################################
# MEMORY
################################
maxmemory 24gb
maxmemory-policy allkeys-lru

################################
# PERSISTENCE (tùy use case)
################################

# Nếu dùng cache / queue realtime:
save ""
appendonly no

# Nếu cần durability:
# appendonly yes
# appendfsync everysec

################################
# PERFORMANCE
################################
io-threads 4
io-threads-do-reads yes
```

## 🎯 4. OS tuning (cực kỳ quan trọng)
```/etc/sysctl.conf```
```
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 262144
net.core.netdev_max_backlog = 250000

vm.overcommit_memory = 1
```
Apply:

```
sysctl -p
```

## 🎯 5. File descriptor

```
ulimit -n 200000
```
**systemd:**
```
sudo systemctl edit redis
[Service]
LimitNOFILE=200000
```
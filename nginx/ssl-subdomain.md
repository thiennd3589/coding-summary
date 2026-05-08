# Cấu hình SSL cho tất cả subdomain

```bash
certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d "*.example.com" \
  -d example.com
```


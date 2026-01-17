# Lệnh chạy docker trên terminal

## Build

```bash
docker build \
  --platform=<running-image-platform> \ # Chọn OS của server. Nếu không chỉ định docker sẽ lấy OS của máy build
  --target <stage> \ # Chọn stage để build. Nếu trong Dockerfile có nhiều stage, chỉ định stage sẽ chạy.
  -t <image-name>:<image-version> \ # frontend:1.0.0
  <build-path> \ # path folder để chạy build
```
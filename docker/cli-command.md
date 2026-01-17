# Lệnh chạy docker trên terminal

## Build

```bash
docker build \
  --platform=linux/amd64 \ 
  --target <stage> \
  -t <image-name>:<image-version> \ # frontend:1.0.0
  <build-path>
```

Trong đó:

- --platform: Chọn OS của server. Nếu không chỉ định docker sẽ lấy OS của máy build. Eg: `linux/amd64`
- --target: Chọn stage để build. Nếu trong Dockerfile có nhiều stage, chỉ định stage sẽ chạy.
- -t: Tên image sau khi build. Eg: `frontend:1.0.0`
- build-path: path folder để chạy build

## Run
```bash
 docker run -d \
  --name api \
  --restart unless-stopped \
  --platform=linux/amd64
  --env-file ./.env.production \
  -e APP_NAME=api \
  -p 3010:3010 \ # expose port
  my-api:1.0.0 
```

Trong đó:

- --name: Tên container
- --restart: Chưa tìm hiểu kỹ, chắc là điều kiện restart container
- --platform: OS để running container
- --env-file: ENV
- -e: Biến môi trường để thêm vào script
- -p: Expose port

---
title: "Tìm hiểu về Docker"
date: 2025-08-20 01:17:00  +0700
categories: [Học tập]
tags: [Học tập]
---

# Tìm hiểu về Docker

<p align="center">
  <img src="/assets/images/docker/1.png" alt="Image title_1" />
</p>

## 1. Docker để làm gì?

Docker là một nền tảng cho phép đóng gói ứng dụng của bạn và toàn bộ môi trường (ngôn ngữ lập trình, thư viện, dependency, system tools…) thành một container.
Điều này có nghĩa là:
- Code trên máy Windows, chạy trên máy Mac, deploy lên Linux đều như nhau.
- Dễ dàng scale ứng dụng bằng cách nhân bản container.

Hiểu đơn giản: Docker = `hộp chứa ứng dụng kèm môi trường chạy` → chạy ở đâu cũng như nhau.

## 2. Dockerfile là gì?

- Dockerfile giống như một `kịch bản` (script) mô tả cách build image.
- Mỗi `image` tạo ra từ Dockerfile có thể chạy thành nhiều container.

## 3. Các từ khóa quan trọng trong Dockerfile

- `FROM`: Dùng để chọn image gốc. Ví dụ:
```
FROM node:22-alpine
```
-> lấy NodeJS 22 bản nhẹ alpine làm base.

- `WORKDIR`: Chỉ định thư mục làm việc bên trong container.
```
WORKDIR /app
```

- `COPY`: Copy file từ máy host vào container.
```
COPY package*.json ./
COPY . .
```

- `RUN`: Chạy một câu lệnh khi build image.
```
RUN npm install
RUN npm run build
```

- `EXPOSE`: Khai báo cổng mà container sẽ sử dụng.
```
EXPOSE 3000
```

- `CMD`: Câu lệnh mặc định khi container khởi động.
```
CMD ["npm", "run", "preview", "--", "--host", "0.0.0.0", "--port", "3000"]
```

## 4. Các câu lệnh Docker cơ bản

 - Xem danh sách các container đang chạy: `docker ps`
 - Xem danh sách tất cả các container (bao gồm cả đã dừng): `docker ps -a`
 - Xem log của một container: `docker logs <container_id>`
 - Xóa một container: `docker rm <container_id>`
 - Xóa một image: `docker rmi <image_id> --force`
 - Xem thông tin chi tiết của một image: `docker inspect <image_id>`
 - Xem dung lượng của các image: `docker images`
 - Xem dung lượng của các container: `docker ps -s`
 - Xem dung lượng của các volume: `docker volume ls`
 - Xem dung lượng của các network: `docker network ls`
 - Xem thông tin về Docker: `docker info`
 - Xem phiên bản Docker: `docker version`
 - Xem log của một container: `docker logs -f <container_id>`

## Case Study: Viết Dockerfile cho Backend NodeJS
Ví dụ bạn có một app NodeJS chạy ở cổng 8080.
Dockerfile sẽ như sau:

```
# Sử dụng Node 22 bản alpine nhẹ
FROM node:22-alpine

# Đặt thư mục làm việc trong container
WORKDIR /app

# Copy file package.json và package-lock.json
COPY package*.json ./

# Copy toàn bộ source code vào container
COPY . .

# Cài đặt dependencies
RUN npm install --force  

# Build app (ví dụ build ra thư mục dist nếu dùng build tool là vite)
RUN npm run build

# Khai báo cổng 8080
EXPOSE 8080

# Câu lệnh chạy ứng dụng
CMD ["npm", "run", "preview", "--", "--host", "0.0.0.0", "--port", "8080"]
```

### Chạy thử app
1. Build image:
```
docker build -t backend-app .
```
2. Run container
```
docker run -p 8080:8080 backend-app
```
3. Truy cập ứng dụng tại: `http://localhost:3000`


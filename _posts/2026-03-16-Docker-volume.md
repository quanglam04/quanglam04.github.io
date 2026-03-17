---
title: "Docker Volumes & Mounts: Giải quyết bài toán dữ liệu trong container"
date: 2026-03-16 00:00:00 +0700
categories: [DevOps, Docker]
tags: [docker, volumes, mount, container, devops, hot-reload]
---


# Docker Volumes & Mounts: Giải quyết bài toán dữ liệu trong container

---

## 1. Vấn đề là gì?

Khi bạn chạy một ứng dụng trong Docker container, có một điều quan trọng cần hiểu: **container hoàn toàn cô lập với máy host**.

Điều đó có nghĩa là:

```
Máy host                  Container
─────────────────         ─────────────────
/home/user/my-app   ≠     /app
```

Hãy tưởng tượng bạn đang phát triển một ứng dụng Node.js. Bạn sửa file `index.js` trên máy, nhưng container vẫn đang chạy phiên bản **cũ** — vì nó không biết gì về thay đổi trên máy bạn cả.

Tệ hơn nữa, khi container bị **xóa hoặc restart**, toàn bộ dữ liệu bên trong **biến mất hoàn toàn**. Điều này gây ra 2 vấn đề lớn:

### Vấn đề 1: Development workflow chậm

```
Sửa code → Build lại image → Restart container → Test
```

Mỗi lần sửa một dòng code, bạn phải lặp lại toàn bộ quy trình trên. Không ai muốn làm vậy.

### Vấn đề 2: Mất dữ liệu

```
Container chạy MySQL → Lưu 1000 records
    ↓
docker rm container
    ↓
Toàn bộ 1000 records biến mất 
```

Database, file upload, logs — tất cả đều mất khi container bị xóa.

---

**Docker Volumes & Mounts** sinh ra để giải quyết đúng 2 vấn đề này: kết nối filesystem của host với container, và persist dữ liệu vượt qua vòng đời của container.

---

## 2. Các loại Volume trong Docker

Docker có 3 loại volume, mỗi loại phục vụ một mục đích khác nhau.

---

### Loại 1: Bind Mount

**Cú pháp:**
```yaml
volumes:
  - /đường/dẫn/host:/đường/dẫn/container
  # hoặc dùng relative path
  - ./src:/app/src
```

**Cơ chế:**
```
Máy host ./src  ←── sync 2 chiều, real-time ──→  Container /app/src

Sửa file trên host  → Container thấy ngay 
Sửa file trong container → Host thấy ngay 
```

**Ví dụ thực tế — Hot reload cho Node.js:**
```yaml
services:
  app:
    build: .
    volumes:
      - ./src:/app/src    # mount source code
    command: npm run dev  # nodemon watch
```

Mỗi lần bạn nhấn **Ctrl+S**, nodemon trong container phát hiện file thay đổi và tự restart. Không cần build lại image.

**Khi nào dùng Bind Mount:**
- Source code trong môi trường development
- Config file (nginx.conf, settings.php...)
- File cần chia sẻ real-time giữa host và container

---

### Loại 2: Named Volume

**Cú pháp:**
```yaml
services:
  db:
    image: mysql
    volumes:
      - db_data:/var/lib/mysql   # tên_volume:container_path

volumes:
  db_data:    # khai báo ở cuối file
```

**Cơ chế:**
```
Docker managed storage (/var/lib/docker/volumes/db_data)
         ↕
Container /var/lib/mysql

Container bị xóa → Data vẫn còn 
docker-compose down → Data vẫn còn 
docker-compose down -v → Data mới bị xóa 
```

Docker tự quản lý nơi lưu trữ trên máy host — bạn không cần biết path cụ thể là đâu.

**Ví dụ thực tế — Persist database:**
```yaml
services:
  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: secret
    volumes:
      - mysql_data:/var/lib/mysql

  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data

volumes:
  mysql_data:
  redis_data:
```

Dù bạn restart container bao nhiêu lần, data vẫn an toàn.

**Khi nào dùng Named Volume:**
- Database (MySQL, PostgreSQL, MongoDB...)
- Cache data (Redis)
- Bất kỳ dữ liệu nào cần persist qua vòng đời container

---

### Loại 3: Anonymous Volume (Trick đặc biệt)

**Cú pháp:**
```yaml
volumes:
  - /app/node_modules    # chỉ có container path, không có host path
```

**Cơ chế:**
```
Bind Mount đang sync ./code → /app

Nếu không có anonymous volume:
  host ./node_modules (Mac/Windows) → ghi đè /app/node_modules (Linux) 
  → Lỗi do binary không tương thích

Với anonymous volume:
  /app/node_modules được Docker bảo vệ, KHÔNG bị sync từ host 
```

**Ví dụ thực tế — Node.js project:**
```yaml
volumes:
  - ./src:/app/src          # sync source code 
  - /app/node_modules       # bảo vệ node_modules của container 
```

Đây là pattern **bắt buộc** với Node.js để tránh conflict giữa `node_modules` trên Mac/Windows với Linux container.

**Khi nào dùng Anonymous Volume:**
- Bảo vệ `node_modules`, `vendor`, `.venv` trong container khỏi bị ghi đè
- Giữ lại build artifacts trong container

---

### Tóm tắt 3 loại

| Loại | Cú pháp | Docker quản lý? | Persist? | Dùng cho |
|---|---|---|---|---|
| Bind Mount | `./host:/container` |  Bạn quản lý |  Trên host | Source code, config |
| Named Volume | `name:/container` |  Docker quản lý |  Trong Docker | Database, persistent data |
| Anonymous Volume | `/container` |  Docker quản lý |  Tạm thời | Bảo vệ folder container |

---

## 3. Best Practices

###  Chỉ mount những gì cần thiết

```yaml
#  Không nên — mount toàn bộ project
volumes:
  - .:/app

#  Nên — chỉ mount folder source code
volumes:
  - ./src:/app/src
```

Mount toàn bộ project kéo theo cả `.git`, `node_modules`, `.env` vào container — gây chậm và có thể gây lỗi.

---

###  Luôn dùng Anonymous Volume cho dependencies

```yaml
#  Pattern chuẩn cho Node.js
volumes:
  - ./src:/app/src
  - /app/node_modules    # không bao giờ bỏ dòng này

#  Pattern chuẩn cho PHP
volumes:
  - ./src:/app/src
  - /app/vendor
```

---

###  Phân biệt rõ môi trường dev và production

```yaml
# docker-compose.yml (production)
services:
  app:
    build: .
    # Không có volumes — code đã được COPY vào image lúc build

# docker-compose.dev.yml (development)
services:
  app:
    build: .
    volumes:
      - ./src:/app/src      # chỉ mount khi dev
    command: npm run dev
```

Chạy dev:
```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```

---

###  Dùng Named Volume thay vì Bind Mount cho database

```yaml
#  Không nên
volumes:
  - ./mysql-data:/var/lib/mysql    # dễ bị conflict permissions

#  Nên
volumes:
  - mysql_data:/var/lib/mysql      # Docker tự quản lý, an toàn hơn

volumes:
  mysql_data:
```

---

###  Thêm `:ro` (read-only) khi container không cần ghi

```yaml
volumes:
  - ./config/nginx.conf:/etc/nginx/nginx.conf:ro     # nginx chỉ đọc config
  - ./custom:/app/modules/custom:ro                  # PHP modules chỉ đọc
  - ./docker-config/settings.php:/app/settings.php:ro
```

Tránh container vô tình ghi đè source code hoặc config quan trọng.

---

###  Đặt tên volume có ý nghĩa

```yaml
#  Không rõ ràng
volumes:
  data1:
  data2:

#  Rõ ràng
volumes:
  mysql_data:
  redis_data:
  uploads_storage:
```

---

###  Backup Named Volume trước khi xóa

```bash
# Backup
docker run --rm \
  -v mysql_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/mysql_backup.tar.gz /data

# Xóa volume
docker volume rm mysql_data

# Restore
docker run --rm \
  -v mysql_data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/mysql_backup.tar.gz -C /
```

---

## Tổng kết

```
Cần hot-reload code khi dev?        → Bind Mount  ./src:/app/src
Cần persist database?               → Named Volume mysql_data:/var/lib/mysql
Cần bảo vệ node_modules?           → Anonymous Volume /app/node_modules
Cần ngăn container ghi vào file?   → Thêm :ro vào cuối
```



---
title: "Kỹ thuật Deferred Join - Tối ưu hóa phân trang cho bảng dữ liệu lớn"
date: 2025-11-21 01:17:00  +0700
categories: [database, pagination]
tags: [optimize]
---

# Kỹ thuật Deferred Join - Tối ưu hóa phân trang cho bảng dữ liệu lớn

## Giới thiệu

**Deferred Join** (hay còn gọi là Delayed Join) là một kỹ thuật tối ưu hóa SQL giúp cải thiện đáng kể hiệu suất của các truy vấn phân trang trên bảng dữ liệu lớn. Kỹ thuật này đặc biệt hữu ích khi bạn cần thực hiện phân trang với `OFFSET` lớn trên bảng có hàng triệu bản ghi.

### Tóm tắt nhanh
- **Vấn đề**: Phân trang chậm khi OFFSET lớn (vd: trang 50,000)
- **Giải pháp**: Tách truy vấn thành 2 bước - lấy ID trước, join sau
- **Kết quả**: Giảm thời gian từ 6 giây xuống 1 giây (cải thiện 6x)

## Vấn đề

### Cách phân trang truyền thống

```sql
SELECT * 
FROM users 
WHERE status = 'active'
LIMIT 20 OFFSET 1000000;
```

### Tại sao cách này chậm?

Khi bạn có bảng với **9 triệu bản ghi** và muốn lấy trang thứ 50,000 (mỗi trang 20 bản ghi):

```
OFFSET = 50,000 × 20 = 1,000,000
```

**MySQL phải:**
1. Quét qua 1,000,000 bản ghi đầu tiên
2. Với MỖI bản ghi, đọc **TẤT CẢ các cột** (id, name, email, address, phone, ...)
3. Tốn rất nhiều I/O và bộ nhớ
4. Sau đó mới bỏ đi và chỉ lấy 20 bản ghi tiếp theo

**Kết quả:** Truy vấn mất 6 giây 

### Minh họa vấn đề

```
Bản ghi:    1  2  3  4  ... 999,999  1,000,000  [1,000,001 → 1,000,020]
                                                        ↑
                                            Chỉ cần 20 bản ghi này
                                            
Nhưng phải quét:  [━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━] × (tất cả cột)
                  1,000,000 bản ghi với đầy đủ dữ liệu
```

## 💡 Giải pháp: Deferred Join

### Ý tưởng cốt lõi

**Thay vì lấy tất cả dữ liệu ngay**, chúng ta:
1. **Bước 1**: Lấy chỉ ID (rất nhanh)
2. **Bước 2**: JOIN để lấy dữ liệu đầy đủ của những ID đó

### Cách triển khai

#### Bước 1: Lấy ID nhanh

```sql
-- Chỉ lấy khóa chính
SELECT id 
FROM users 
WHERE status = 'active'
LIMIT 20 OFFSET 1000000;
```

**Tại sao nhanh?**
- Chỉ quét cột `id` (thường có index)
- Dữ liệu nhỏ gọn (4-8 bytes/bản ghi)
- Index scan thay vì full table scan

#### Bước 2: JOIN để lấy dữ liệu đầy đủ

```sql
-- Sử dụng subquery
SELECT u.* 
FROM users u
INNER JOIN (
    SELECT id 
    FROM users 
    WHERE status = 'active'
    LIMIT 20 OFFSET 1000000
) AS temp ON u.id = temp.id;
```

**Hoặc sử dụng WITH (CTE):**

```sql
WITH user_ids AS (
    SELECT id 
    FROM users 
    WHERE status = 'active'
    LIMIT 20 OFFSET 1000000
)
SELECT u.* 
FROM users u
INNER JOIN user_ids ON u.id = user_ids.id;
```

## So sánh hiệu suất

### Trước khi tối ưu

```sql
SELECT * FROM users WHERE status = 'active' LIMIT 20 OFFSET 1000000;
```

| Metric | Giá trị |
|--------|---------|
| Thời gian thực thi | **6 giây** |
| Rows examined | 1,000,020 |
| Data scanned | ~500 MB (giả sử mỗi row 500 bytes) |
| Using index |  Không |

### Sau khi tối ưu (Deferred Join)

```sql
SELECT u.* FROM users u
INNER JOIN (
    SELECT id FROM users WHERE status = 'active'
    LIMIT 20 OFFSET 1000000
) temp ON u.id = temp.id;
```

| Metric | Giá trị |
|--------|---------|
| Thời gian thực thi | **1 giây** |
| Rows examined (step 1) | 1,000,020 |
| Rows examined (step 2) | 20 |
| Data scanned | ~8 MB (step 1) + 10 KB (step 2) |
| Using index | Có (step 1) |

### Kết quả

```
┌─────────────────────┬──────────┬──────────────┐
│ Phương pháp         │ Thời gian│ Cải thiện    │
├─────────────────────┼──────────┼──────────────┤
│ Truyền thống        │  6s      │ -            │
│ Deferred Join       │  1s      │ 6x nhanh hơn │
└─────────────────────┴──────────┴──────────────┘
```

## 🔧 Ví dụ thực tế

### Ví dụ 1: Bảng Users

**Schema:**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100),
    address TEXT,
    phone VARCHAR(20),
    created_at TIMESTAMP,
    status ENUM('active', 'inactive'),
    INDEX idx_status (status)
);
```

**Cách cũ (chậm):**
```sql
SELECT id, name, email, address, phone, created_at
FROM users
WHERE status = 'active'
ORDER BY created_at DESC
LIMIT 20 OFFSET 500000;
```

**Cách mới (nhanh):**
```sql
SELECT u.id, u.name, u.email, u.address, u.phone, u.created_at
FROM users u
INNER JOIN (
    SELECT id
    FROM users
    WHERE status = 'active'
    ORDER BY created_at DESC
    LIMIT 20 OFFSET 500000
) temp ON u.id = temp.id
ORDER BY u.created_at DESC;
```

### Ví dụ 2: Bảng Orders với nhiều điều kiện

**Cách cũ:**
```sql
SELECT *
FROM orders
WHERE status = 'completed' 
  AND created_at >= '2024-01-01'
  AND total_amount > 1000
LIMIT 50 OFFSET 100000;
```

**Cách mới:**
```sql
SELECT o.*
FROM orders o
INNER JOIN (
    SELECT id
    FROM orders
    WHERE status = 'completed' 
      AND created_at >= '2024-01-01'
      AND total_amount > 1000
    LIMIT 50 OFFSET 100000
) temp ON o.id = temp.id;
```

### Ví dụ 3: Với JOIN phức tạp

**Cách cũ:**
```sql
SELECT u.*, p.profile_data, s.settings
FROM users u
LEFT JOIN profiles p ON u.id = p.user_id
LEFT JOIN settings s ON u.id = s.user_id
WHERE u.status = 'active'
LIMIT 20 OFFSET 200000;
```

**Cách mới:**
```sql
-- Bước 1: Lấy ID
WITH user_ids AS (
    SELECT id
    FROM users
    WHERE status = 'active'
    LIMIT 20 OFFSET 200000
)
-- Bước 2: JOIN với tất cả bảng cần thiết
SELECT u.*, p.profile_data, s.settings
FROM user_ids
INNER JOIN users u ON user_ids.id = u.id
LEFT JOIN profiles p ON u.id = p.user_id
LEFT JOIN settings s ON u.id = s.user_id;
```

## Khi nào nên sử dụng

### Nên sử dụng khi:

1. **OFFSET lớn** (thường > 1,000)
   ```
   LIMIT 20 OFFSET 50000  ← Nên dùng
   ```

2. **Bảng có nhiều cột**
   ```sql
   -- Bảng với 20-30 cột, mỗi row ~500 bytes
   CREATE TABLE products (
       id, name, description, price, 
       stock, category, brand, images, 
       specifications, ...  -- nhiều cột
   );
   ```

3. **Bảng có hàng triệu bản ghi**
   ```
   > 1,000,000 rows ← Nên dùng
   ```

4. **Đã có index trên WHERE và PRIMARY KEY**
   ```sql
   INDEX idx_status (status)
   PRIMARY KEY (id)
   ```

### Không cần thiết khi:

1. **OFFSET nhỏ** (< 1,000)
   ```
   LIMIT 20 OFFSET 100  ← Không cần, đã đủ nhanh
   ```

2. **Bảng nhỏ** (< 100,000 rows)
   ```
   SELECT * FROM categories  -- chỉ có 50 categories
   ```

3. **Ít cột**
   ```sql
   -- Bảng chỉ có 3-4 cột
   CREATE TABLE tags (id, name, slug);
   ```

4. **Không có index phù hợp**
   ```
   Không có index → Deferred Join cũng chậm
   ```

## Best Practices

### 1. Đảm bảo có Index phù hợp

```sql
-- Index trên cột WHERE
CREATE INDEX idx_status ON users(status);

-- Composite index cho nhiều điều kiện
CREATE INDEX idx_status_created ON users(status, created_at);

-- Covering index (nếu có thể)
CREATE INDEX idx_covering ON users(status, created_at, id);
```

### 2. Sử dụng EXPLAIN để kiểm tra

```sql
EXPLAIN SELECT u.*
FROM users u
INNER JOIN (
    SELECT id FROM users WHERE status = 'active'
    LIMIT 20 OFFSET 1000000
) temp ON u.id = temp.id;
```

**Cần xem:**
- `type`: Nên là `ref` hoặc `range`, không phải `ALL`
- `key`: Nên sử dụng index
- `rows`: Số lượng rows examined

### 3. Kết hợp với cursor-based pagination

Thay vì OFFSET, sử dụng WHERE:

```sql
-- Thay vì OFFSET
SELECT id FROM users WHERE status = 'active' LIMIT 20 OFFSET 1000000;

-- Dùng cursor (nhanh hơn nhiều)
SELECT id FROM users 
WHERE status = 'active' AND id > 1000000
LIMIT 20;
```

### 4. Cân nhắc materialized view hoặc cache

Nếu truy vấn được gọi nhiều lần:

```sql
-- Tạo bảng tạm lưu kết quả
CREATE TABLE active_user_ids AS
SELECT id FROM users WHERE status = 'active';

-- Phân trang trên bảng tạm (rất nhanh)
SELECT id FROM active_user_ids LIMIT 20 OFFSET 1000000;
```

### 5. Monitoring và điều chỉnh

```sql
-- Kiểm tra slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;

-- Phân tích query performance
SELECT * FROM mysql.slow_log 
WHERE sql_text LIKE '%users%' 
ORDER BY query_time DESC;
```

## Giải thích sâu hơn

### Tại sao Index-only scan nhanh hơn?

**Full table scan:**
```
Disk → [id|name|email|address|...] → Memory
       ↑
       Đọc toàn bộ row (500 bytes)
```

**Index scan (chỉ lấy ID):**
```
Index → [id] → Memory
        ↑
        Chỉ đọc 8 bytes
```

### So sánh I/O operations

| Operation | Truyền thống | Deferred Join |
|-----------|--------------|---------------|
| **Step 1** | Đọc 1M rows × 500 bytes = 500 MB | Đọc 1M IDs × 8 bytes = 8 MB |
| **Step 2** | - | Đọc 20 rows × 500 bytes = 10 KB |
| **Tổng I/O** | 500 MB | 8 MB + 10 KB ≈ 8 MB |
| **Tỷ lệ** | 62x chậm hơn | 62x nhanh hơn |

### Ví dụ với số liệu thực tế

**Bảng users: 9,000,000 rows, mỗi row 500 bytes**

```
Phân trang trang 50,000 (OFFSET = 1,000,000)

┌──────────────────────────────────────────────────────────┐
│ CÁCH TRUYỀN THỐNG                                        │
├──────────────────────────────────────────────────────────┤
│ 1. Quét 1,000,020 rows                                   │
│    • Đọc từ disk: 1,000,020 × 500 bytes ≈ 476 MB         │
│    • Load vào memory: 476 MB                             │
│                                                          │
│ 2. Bỏ đi 1,000,000 rows                                  │
│                                                          │
│ 3. Trả về 20 rows                                        │
│                                                          │
│ Thời gian: 6 giây                                        │
└──────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────┐
│ DEFERRED JOIN                                            │
├──────────────────────────────────────────────────────────┤
│ BƯỚC 1: Lấy ID                                           │
│    • Quét index: 1,000,020 IDs                           │
│    • Đọc từ disk: 1,000,020 × 8 bytes ≈ 7.6 MB           │
│    • Load vào memory: 7.6 MB                             │
│    • Lấy 20 IDs                                          │
│                                                          │
│ BƯỚC 2: JOIN lấy dữ liệu                                 │
│    • Tìm 20 rows theo ID (very fast)                     │
│    • Đọc từ disk: 20 × 500 bytes = 10 KB                 │
│                                                          │
│ Thời gian: 1 giây                                        │
└──────────────────────────────────────────────────────────┘
```


## Tham khảo thêm

### Các kỹ thuật liên quan

1. **Cursor-based Pagination**
   ```sql
   SELECT * FROM users WHERE id > :last_id LIMIT 20;
   ```

2. **Keyset Pagination**
   ```sql
   SELECT * FROM users 
   WHERE (created_at, id) > (:last_created_at, :last_id)
   ORDER BY created_at, id
   LIMIT 20;
   ```

3. **Covering Index**
   ```sql
   CREATE INDEX idx_covering ON users(status, created_at, id, name, email);
   ```

### Tài liệu

- [MySQL EXPLAIN Documentation](https://dev.mysql.com/doc/refman/8.0/en/explain.html)
- [Use The Index, Luke - Pagination](https://use-the-index-luke.com/sql/partial-results/fetch-next-page)
- [PostgreSQL Performance Optimization](https://www.postgresql.org/docs/current/performance-tips.html)





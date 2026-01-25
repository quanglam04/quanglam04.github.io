---
title: "Tìm hiểu về tiến trình Vacuum trong PostgreSQL"
date: 2026-01-24 01:17:00  +0700
categories: [database, optimize, postgre]
tags: [optimize]
---

# VACUUM trong PostgreSQL - Kiến thức cơ bản cho Backend Developer

## 1. Tại sao PostgreSQL cần VACUUM?

PostgreSQL sử dụng cơ chế **MVCC (Multi-Version Concurrency Control)** để quản lý đồng thời. Khác với các database khác, PostgreSQL **không xóa ngay** dữ liệu cũ khi bạn UPDATE hoặc DELETE.

### Ví dụ đơn giản:

```sql
-- Bảng users ban đầu
id | name  | email
1  | Alice | alice@email.com

-- Khi bạn UPDATE
UPDATE users SET email = 'alice.new@email.com' WHERE id = 1;

-- PostgreSQL làm gì?
-- KHÔNG xóa bản ghi cũ, mà:
-- 1. Tạo bản ghi mới với email mới
-- 2. Đánh dấu bản ghi cũ là "dead tuple" (bản ghi chết)
-- 3. Cả 2 bản ghi đều tồn tại trên đĩa!

-- Kết quả trong file vật lý:
id | name  | email                  | status
1  | Alice | alice@email.com        | DEAD (invisible)
1  | Alice | alice.new@email.com    | LIVE (visible)
```

### Hậu quả:

- **Table Bloat**: Bảng ngày càng phình to vì chứa cả dữ liệu cũ và mới
- **Performance giảm**: Query phải scan qua nhiều bản ghi chết
- **Lãng phí đĩa**: Dữ liệu cũ chiếm chỗ mà không dùng đến

**→ VACUUM sinh ra để dọn dẹp các dead tuples này!**

---

## 2. Cơ chế MVCC

### 2.1. MVCC là gì?

MVCC cho phép nhiều transactions đọc/ghi đồng thời mà không chặn nhau bằng cách **lưu nhiều phiên bản** của cùng một bản ghi.

### 2.2. So sánh với các database khác:

| Database | Cách xử lý dữ liệu cũ |
|----------|----------------------|
| **Oracle** | Chuyển vào UNDO tablespace riêng |
| **SQL Server** | Chuyển vào TempDB |
| **PostgreSQL** | Lưu trực tiếp trong bảng gốc |

### 2.3. Transaction ID và Visibility

Mỗi bản ghi có 2 thông tin ẩn:

```
xmin: Transaction ID tạo ra bản ghi này
xmax: Transaction ID xóa/update bản ghi này
```

**Ví dụ:**

```sql
-- Transaction 100: INSERT
INSERT INTO users VALUES (1, 'Alice');
→ xmin=100, xmax=NULL (vẫn còn sống)

-- Transaction 150: UPDATE
UPDATE users SET name='Alice Smith' WHERE id=1;
→ Bản ghi cũ: xmin=100, xmax=150 (đã chết)
→ Bản ghi mới: xmin=150, xmax=NULL (đang sống)
```

PostgreSQL quyết định bản ghi nào "visible" cho mỗi transaction dựa trên xmin/xmax.

---

## 3. Auto-VACUUM hoạt động như thế nào?

### 3.1. Tổng quan

Auto-VACUUM là tiến trình tự động chạy ngầm để dọn dẹp dead tuples. Bạn **không cần** chạy VACUUM thủ công trong hầu hết trường hợp.

### 3.2. Khi nào Auto-VACUUM chạy?

PostgreSQL kiểm tra mỗi **1 phút** (mặc định) và chạy VACUUM khi đủ điều kiện.

**Công thức:**

```
Chạy VACUUM khi:
số_dead_tuples >= threshold + (scale_factor × số_bản_ghi_trong_bảng)
```

**Tham số mặc định:**

```ini
autovacuum_vacuum_threshold = 50
autovacuum_vacuum_scale_factor = 0.2  (20%)
```

### 3.3. Ví dụ tính toán:

```
Bảng có 100,000 bản ghi:
threshold = 50 + (0.2 × 100,000) = 20,050 dead tuples
→ Cần >= 20,050 dead tuples thì Auto-VACUUM mới chạy

Bảng có 10,000,000 bản ghi:
threshold = 50 + (0.2 × 10,000,000) = 2,000,050 dead tuples
→ Cần >= 2 triệu dead tuples mới chạy
```

### 3.4. Các loại VACUUM:

| Loại | Mục đích | Lock bảng? | Khi nào dùng? |
|------|----------|------------|---------------|
| **VACUUM** | Đánh dấu dead tuples có thể tái sử dụng | Không | Tự động, thường xuyên |
| **VACUUM FULL** | Xóa hoàn toàn dead tuples, thu nhỏ file | Có (Exclusive) | Hiếm khi, khi bloat quá lớn |
| **VACUUM ANALYZE** | VACUUM + cập nhật thống kê | Không | Sau khi load data lớn |

### 3.5. Monitoring đơn giản

```sql
-- Xem Auto-VACUUM chạy lần cuối khi nào
SELECT 
    relname as table_name,
    last_autovacuum,
    n_dead_tup as dead_tuples,
    n_live_tup as live_tuples,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup, 0), 2) AS dead_percentage
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 10;
```

---

## 4. Các vấn đề thường gặp

### 4.1. Long-Running Transactions

**Vấn đề:** Query hoặc transaction chạy quá lâu (vài tiếng) sẽ ngăn VACUUM dọn dẹp.

**Tại sao?** PostgreSQL phải giữ lại dữ liệu cũ để phục vụ transaction đó.

**Ví dụ:**

```sql
-- Session 1: Transaction chạy lâu (8:00 AM)
BEGIN;
SELECT * FROM huge_report;  -- Query chạy 4 tiếng
-- Chưa COMMIT...

-- Session 2: Cập nhật dữ liệu (9:00 AM)
UPDATE products SET price = price * 1.1;
-- Tạo ra 1 triệu dead tuples

-- Auto-VACUUM chạy (10:00 AM)
-- KHÔNG THỂ dọn được vì Session 1 vẫn đang chạy!
-- Phải đợi đến 12:00 khi Session 1 COMMIT
```

**Giải pháp cho Backend Developer:**

```python
# Bad - Transaction mở quá lâu
conn = psycopg2.connect(...)
cursor = conn.cursor()
cursor.execute("BEGIN")
cursor.execute("SELECT * FROM large_table")  # Query lâu
time.sleep(3600)  # Làm việc khác 1 tiếng
cursor.execute("COMMIT")

# Good - Đóng transaction sớm
conn = psycopg2.connect(...)
cursor = conn.cursor()
cursor.execute("SELECT * FROM large_table")
results = cursor.fetchall()  # Lấy dữ liệu ra ngay
# Transaction tự động đóng
# Xử lý dữ liệu sau
process_data(results)
```

### 4.2. Hot Standby Feedback

**Vấn đề:** Nếu có database dự phòng (Standby), query chạy lâu trên Standby cũng có thể block VACUUM trên Primary.

**Khi nào xảy ra:**

```
Primary Database (production)
    ↓ Replication
Standby Database (cho báo cáo)
```

Nếu bật `hot_standby_feedback = on`:
- Query chạy lâu trên Standby
- Gửi tín hiệu về Primary: "Đừng xóa dữ liệu cũ!"
- VACUUM trên Primary bị block

**Giải pháp:** Tắt `hot_standby_feedback` hoặc tách Standby riêng cho báo cáo.

---

## 5. PostgreSQL 17 - Điểm mới

### 5.1. Vấn đề của các phiên bản cũ

**PostgreSQL < 17:**
- VACUUM lưu danh sách dead tuples trong **mảng** (array)
- Giới hạn bộ nhớ: **1GB** cho mỗi VACUUM process
- Chỉ xử lý được ~**178 triệu** dead tuples trong 1 lần quét

**Hậu quả:**
- Bảng lớn với nhiều dead tuples cần VACUUM nhiều lần
- Mỗi lần phải scan lại toàn bộ indexes → chậm

### 5.2. Cải tiến trong PostgreSQL 17
 Chuyển từ cấu trúc **Array** sang **Tree** (Radix Tree)
 **Không giới hạn** số lượng dead tuples có thể xử lý
 Tiết kiệm bộ nhớ lên đến **20 lần**
 VACUUM nhanh hơn rất nhiều với bảng lớn

**Benchmark:**

| Kịch bản | PG 16 | PG 17 | Cải thiện |
|----------|-------|-------|-----------|
| Bảng 1TB | 12GB RAM | 600MB RAM | 20x |
| Thời gian VACUUM | 45 phút | 18 phút | 2.5x |

---

## 6. Tóm tắt nhanh (Cheat Sheet)

### Key Concepts:

```
MVCC → UPDATE không xóa dữ liệu cũ
      → Tạo dead tuples
      → Cần VACUUM để dọn dẹp
      → Auto-VACUUM tự động làm việc này
```

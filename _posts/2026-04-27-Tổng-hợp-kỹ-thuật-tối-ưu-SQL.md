---
title: "Tổng hợp các kỹ thuật tối ưu Query SQL — Từ cơ bản đến nâng cao"
date: 2026-04-27 21:38:00 +0700
categories: [Database, Optimize]
tags: [database, optimize]
---

# Tổng hợp các kỹ thuật tối ưu Query SQL — Từ cơ bản đến nâng cao

> Bài viết tổng hợp từ mini series **Optimize SQL Query** — đúc kết những kỹ thuật thực chiến giúp câu lệnh SQL của bạn chạy nhanh hơn, tiêu tốn ít tài nguyên hơn.

---

## Mở đầu

Khi hệ thống bắt đầu lớn lên, dữ liệu ngày càng nhiều, những câu query tưởng chừng ổn lúc development bỗng trở thành nút thắt cổ chai nghiêm trọng trên production. Một query chậm không chỉ làm người dùng bực bội — nó còn kéo theo CPU, RAM, I/O và làm ảnh hưởng đến toàn bộ hệ thống.

Dưới đây là 9 nhóm kỹ thuật tối ưu SQL mà mình đã tổng hợp, từ những thứ cơ bản nhất đến các chiến lược kiến trúc dữ liệu nâng cao.

---

## 1. Sử dụng Index hợp lý

### 1.1. Đánh index cho các column thường xuyên được query

Index là cấu trúc dữ liệu đặc biệt giúp database tìm kiếm nhanh hơn mà không cần duyệt toàn bộ bảng (Full Table Scan). Khi thực hiện `SELECT`, `JOIN` hoặc lọc qua `WHERE` trên column đã được đánh index, hệ quản trị CSDL sẽ tra cứu trên cấu trúc index thay vì scan từng dòng một.

> **Lưu ý:** Lạm dụng index sẽ phản tác dụng. Mỗi lần ghi dữ liệu (`INSERT`/`UPDATE`/`DELETE`), hệ thống cũng phải cập nhật index — vừa tốn thêm bộ nhớ, vừa làm chậm thao tác ghi.

### 1.2. Chọn đúng loại index

| Loại index | Thích hợp cho | Hạn chế |
|---|---|---|
| **B-Tree Index** | Tìm kiếm chính xác, range query | — |
| **Hash Index** | So sánh bằng (`=`) | Không dùng được cho range |
| **Full-Text Index** | Tìm kiếm văn bản phức tạp | Tốn tài nguyên khi index lớn |

---

## 2. Viết câu lệnh SQL đúng cách

### 2.1. Không dùng `SELECT *`

Lấy toàn bộ column khi chỉ cần một vài trường là lãng phí tài nguyên mạng, bộ nhớ và CPU — đặc biệt với các bảng có nhiều column lớn như `TEXT` hay `BLOB`.

```sql
-- Nên làm
SELECT id, name, email FROM users WHERE status = 'active';

-- Tránh
SELECT * FROM users WHERE status = 'active';
```

### 2.2. Tách query phức tạp thành nhiều bước nhỏ

Một query quá phức tạp vừa khó debug, vừa khó cho query optimizer xử lý hiệu quả. Đôi khi, chia nhỏ thành từng bước rõ ràng — thậm chí dùng bảng tạm — lại cho hiệu suất tốt hơn.

### 2.3. Chỉ dùng JOIN khi thực sự cần

JOIN nhiều bảng cùng lúc khiến execution plan trở nên phức tạp và chi phí xử lý tăng vọt. Một vài nguyên tắc khi dùng JOIN:

- Luôn đảm bảo **cột dùng để JOIN có index** (đặc biệt là foreign key).
- Ưu tiên tính toán aggregate ở từng bảng riêng trước, sau đó mới JOIN kết quả lại — thay vì aggregate trực tiếp trên nhiều bảng lớn cùng lúc.

---

## 3. Phân tích và đo lường hiệu suất

### 3.1. Dùng `EXPLAIN` để xem execution plan

Hầu hết các hệ quản trị (MySQL, PostgreSQL, SQL Server) đều hỗ trợ lệnh `EXPLAIN` hoặc `EXPLAIN ANALYZE`. Đây là công cụ không thể thiếu để hiểu database đang làm gì với câu query của bạn.

```sql
EXPLAIN ANALYZE
SELECT id, name FROM orders WHERE created_at > '2025-01-01';
```

Từ execution plan, bạn có thể thấy:
- Query có đang dùng index không?
- Có xảy ra Full Table Scan không?
- Bước nào tốn thời gian nhất?

### 3.2. So sánh tốc độ trước và sau khi tối ưu

Đừng tối ưu theo cảm tính. Hãy đo thời gian thực thi thực tế trước và sau khi thay đổi. Một kỹ thuật hiệu quả ở môi trường này chưa chắc đã phù hợp ở nơi khác — vì còn phụ thuộc vào cấu hình phần cứng, dung lượng dữ liệu và kiểu dữ liệu thực tế.

---

## 4. Chuẩn hóa dữ liệu (Normalization)

Normalization giúp loại bỏ sự dư thừa (redundancy) trong cơ sở dữ liệu, làm cho cấu trúc bảng gọn gàng và nhất quán hơn.

Khi dữ liệu được tổ chức đúng chuẩn:
- Khối lượng dữ liệu trong mỗi truy vấn giảm đi đáng kể.
- Index hoạt động hiệu quả hơn.
- JOIN trở nên đơn giản và có thể dự đoán được.

> 💡 Tuy nhiên, trong một số trường hợp đặc thù (ví dụ data warehouse, reporting), **denormalization** có thể được cân nhắc để giảm số lần JOIN.

---

## 5. Tận dụng Cache và Partition

### 5.1. Cache kết quả truy vấn

Nếu hệ thống phải thực hiện cùng một query nhiều lần với kết quả ít thay đổi, hãy cache lại kết quả đó thay vì truy vấn database liên tục.

Các lựa chọn phổ biến:
- **Redis** — in-memory cache, hỗ trợ TTL linh hoạt
- **Memcached** — đơn giản, hiệu suất cao cho key-value cache
- **Application-level cache** — cache trong bộ nhớ ứng dụng

### 5.2. Phân vùng bảng dữ liệu (Partition)

Với các bảng chứa hàng chục triệu dòng trở lên, **partition** giúp mỗi query chỉ làm việc với một phân vùng nhỏ thay vì toàn bộ bảng.

Các chiến lược partition phổ biến:
- **Range partition** theo cột ngày tháng (ví dụ: partition theo tháng/năm)
- **Hash partition** theo khóa chính (`id`)
- **List partition** theo giá trị danh mục (ví dụ: `region`, `status`)

---

## 6. Tránh dùng Function trên Column trong mệnh đề WHERE

Khi bạn bọc một column trong function tại mệnh đề `WHERE`, database thường không thể dùng index trên column đó nữa — vì nó phải tính toán function cho từng dòng trước khi so sánh.

```sql
-- Không dùng được index trên created_at
SELECT * FROM orders WHERE YEAR(created_at) = 2025;

-- Index hoạt động bình thường
SELECT * FROM orders
WHERE created_at >= '2025-01-01' AND created_at < '2026-01-01';
```

Nguyên tắc chung: **hãy để column "trần" ở vế trái**, còn biến đổi thì thực hiện ở vế phải của điều kiện.

---

## 7. Tối ưu cấu trúc bảng và kiểu dữ liệu

Chọn đúng kiểu dữ liệu ngay từ đầu có thể tiết kiệm đáng kể bộ nhớ và tăng tốc truy xuất:

- Dùng `INT` thay `BIGINT` nếu giá trị không vượt quá ~2 tỷ.
- Dùng `TINYINT` cho các cột boolean hoặc trạng thái nhỏ.
- Tránh `VARCHAR(255)` hoặc `TEXT` cho những trường chỉ lưu vài ký tự — chi phí xử lý chuỗi cao hơn số nguyên đáng kể.
- Cân nhắc dùng `ENUM` cho các cột có tập giá trị cố định, nhỏ.

> Lưu ý: Thay đổi kiểu dữ liệu trên bảng production lớn là thao tác nặng — cần có kế hoạch migration cẩn thận.

---

## 8. Hạn chế Subquery lồng nhau

Subquery lồng nhau (nested subquery) trong mệnh đề `SELECT` hoặc `WHERE` có thể bị thực thi lặp lại cho từng dòng của query cha — gây ra hiệu suất rất kém với dữ liệu lớn.

```sql
-- Subquery có thể chạy N lần
SELECT name FROM users
WHERE id IN (SELECT user_id FROM orders WHERE status = 'pending');

-- Dùng JOIN thay thế
SELECT DISTINCT u.name FROM users u
JOIN orders o ON o.user_id = u.id
WHERE o.status = 'pending';
```

Trong các trường hợp phức tạp hơn, hãy cân nhắc dùng **CTE (Common Table Expressions)** với cú pháp `WITH` — code dễ đọc hơn và optimizer có thể xử lý tốt hơn trong nhiều tình huống.

---

## 9. Chạy query nặng vào thời điểm phù hợp

Không phải query nào cũng cần chạy real-time. Các báo cáo, thống kê phức tạp với grouping lớn nên được lên lịch chạy vào **khung giờ thấp điểm** để giảm tải cho hệ thống trong giờ cao điểm.

Một số chiến lược thực tế:
- Dùng **cron job** để chạy aggregation định kỳ (hàng giờ, hàng ngày).
- Lưu kết quả vào **summary table** và query trực tiếp từ đó thay vì tính toán lại mỗi lần.
- Với dữ liệu analytics, cân nhắc tách riêng **read replica** hoặc data warehouse để không ảnh hưởng database chính.

---

## Kết luận

Tối ưu SQL không phải là việc tìm ra một "công thức thần kỳ" rồi áp dụng một lần là xong. Đó là một quá trình liên tục — đo lường, phân tích, thử nghiệm và cải thiện.

Tóm gọn 9 kỹ thuật trong bài:

1. **Index đúng chỗ** — không thừa, không thiếu
2. **Viết SQL rõ ràng** — tránh `SELECT *`, JOIN có chủ đích
3. **Dùng `EXPLAIN`** — đừng đoán, hãy đo
4. **Chuẩn hóa dữ liệu** — nền tảng cho mọi tối ưu khác
5. **Cache & Partition** — chiến lược cho dữ liệu lớn
6. **Không bọc column trong function** ở `WHERE`
7. **Chọn đúng kiểu dữ liệu** từ đầu
8. **Hạn chế subquery lồng** — ưu tiên JOIN hoặc CTE
9. **Lên lịch query nặng** vào giờ thấp điểm



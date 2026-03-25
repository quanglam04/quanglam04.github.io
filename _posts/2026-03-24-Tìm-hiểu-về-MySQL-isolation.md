---
title: "MySQL Transaction Isolation Levels: Hiểu Rõ Để Xử Lý Đồng Thời Hiệu Quả"
date: 2026-03-24 21:38:00 +0700
categories: [Database, MySQL]
tags: [mysql, transaction, isolation-levels]
---

# MySQL Transaction Isolation Levels

> **TL;DR:** MySQL cung cấp 4 cấp độ cách ly (Isolation Levels) để kiểm soát cách các transaction tương tác với nhau khi truy cập dữ liệu đồng thời. Hiểu rõ từng cấp độ giúp bạn đưa ra quyết định đúng đắn giữa **hiệu suất** và **tính nhất quán dữ liệu**.

---

## Mở đầu: Tại sao cần quan tâm đến Isolation Levels?

Hãy tưởng tượng bạn đang xây dựng một hệ thống thương mại điện tử. Vào ngày flash sale, hàng nghìn người dùng cùng lúc đặt hàng, thanh toán, kiểm tra tồn kho. Mỗi thao tác đó là một **transaction** — và câu hỏi đặt ra là: khi hàng trăm transaction chạy đồng thời trên cùng một bảng dữ liệu, làm sao MySQL đảm bảo dữ liệu không bị "loạn"?

Câu trả lời nằm ở **Isolation Levels** — cơ chế cách ly giữa các transaction.

---

## Transaction trong MySQL: Nền tảng cần nắm

### Cú pháp cơ bản

Một transaction trong MySQL được bao bọc bởi ba câu lệnh chính:

```sql
START TRANSACTION;

-- Các câu lệnh SQL ở đây
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;

COMMIT;  -- Xác nhận thay đổi
-- hoặc
ROLLBACK;  -- Hủy toàn bộ nếu có lỗi
```

Nguyên tắc hoạt động rất đơn giản: hoặc **tất cả thành công** (`COMMIT`), hoặc **tất cả thất bại** (`ROLLBACK`). Không có chuyện chuyển tiền trừ tài khoản A mà quên cộng vào tài khoản B.

### Tính độc lập giữa các session

Mỗi kết nối (session) đến MySQL hoạt động độc lập. Session A đang thực hiện transaction không ảnh hưởng trực tiếp đến session B — **cho đến khi** dữ liệu được commit. Mức độ "cho đến khi" này chính là thứ mà Isolation Levels quyết định.

### Savepoint — Rollback từng phần

MySQL cho phép đặt các điểm lưu (savepoint) bên trong transaction:

```sql
START TRANSACTION;

INSERT INTO orders (product_id, quantity) VALUES (1, 10);
SAVEPOINT before_payment;

UPDATE wallets SET balance = balance - 1000 WHERE user_id = 5;
-- Giả sử bước này lỗi
ROLLBACK TO before_payment;
-- Chỉ rollback phần thanh toán, giữ lại đơn hàng

COMMIT;
```

**Thực tế:** Hầu hết lập trình viên ít sử dụng savepoint. Lý do? Rủi ro khó kiểm soát. Triết lý phổ biến hơn là **"all or nothing"** — thành công hoàn toàn hoặc thất bại hoàn toàn. Đơn giản, an toàn, dễ debug.

---

## Bốn cấp độ cách ly (Isolation Levels)

MySQL hỗ trợ 4 cấp độ cách ly, xếp theo thứ tự từ **ít nghiêm ngặt** đến **nghiêm ngặt nhất**:

### 1. Read Uncommitted — "Đọc bẩn cũng được"

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

Đây là cấp độ cách ly **thấp nhất**. Một transaction có thể đọc được dữ liệu mà transaction khác **chưa commit**.

**Ví dụ minh họa:**

| Thời điểm | Session A | Session B |
|-----------|-----------|-----------|
| T1 | `START TRANSACTION;` | |
| T2 | `UPDATE products SET stock = 0 WHERE id = 1;` | |
| T3 | | `SELECT stock FROM products WHERE id = 1;` → **Kết quả: 0** |
| T4 | `ROLLBACK;` (Hủy thay đổi!) | |
| T5 | | Session B đã đọc dữ liệu **sai** |

Session B đọc được `stock = 0` dù Session A chưa commit và sau đó đã rollback. Đây gọi là hiện tượng **Dirty Read** (Đọc bẩn).

**Khi nào dùng?** Gần như không bao giờ trong production. Chỉ phù hợp cho các tác vụ đọc dữ liệu không cần chính xác tuyệt đối, ví dụ dashboard thống kê gần đúng, nơi hiệu suất được ưu tiên hơn độ chính xác.

---

### 2. Read Committed — "Chỉ đọc dữ liệu đã xác nhận"

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

Khắc phục được Dirty Read. Một transaction **chỉ thấy** dữ liệu đã được commit bởi các transaction khác.

**Ví dụ minh họa:**

| Thời điểm | Session A | Session B |
|-----------|-----------|-----------|
| T1 | `START TRANSACTION;` | `START TRANSACTION;` |
| T2 | `UPDATE products SET price = 200 WHERE id = 1;` | |
| T3 | | `SELECT price FROM products WHERE id = 1;` → **100** (giá cũ) |
| T4 | `COMMIT;` | |
| T5 | | `SELECT price FROM products WHERE id = 1;` → **200** (giá mới) |

Session B không thấy giá trị 200 cho đến khi Session A commit. Tuy nhiên, **cùng một câu SELECT trong cùng một transaction lại cho ra kết quả khác nhau** ở T3 và T5. Hiện tượng này gọi là **Non-Repeatable Read**.

**Khi nào dùng?** Phù hợp với các hệ thống cần dữ liệu tương đối chính xác nhưng chấp nhận được việc kết quả đọc thay đổi trong cùng một transaction. Oracle Database và PostgreSQL sử dụng đây làm mặc định.

---

### 3. Repeatable Read — "Đọc ổn định" (Mặc định của MySQL)

```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

Đây là **cấp độ mặc định** của MySQL (InnoDB). Khi một transaction bắt đầu, nó sẽ "chụp ảnh" (snapshot) trạng thái dữ liệu tại thời điểm đó. Mọi câu SELECT trong transaction luôn trả về cùng một kết quả, bất kể transaction khác có commit gì đi nữa.

**Ví dụ minh họa:**

| Thời điểm | Session A | Session B |
|-----------|-----------|-----------|
| T1 | `START TRANSACTION;` | `START TRANSACTION;` |
| T2 | | `UPDATE products SET price = 300 WHERE id = 1;` |
| T3 | | `COMMIT;` |
| T4 | `SELECT price FROM products WHERE id = 1;` → **100** (giá cũ!) | |
| T5 | `SELECT price FROM products WHERE id = 1;` → **100** (vẫn giá cũ!) | |
| T6 | `COMMIT;` | |
| T7 | `SELECT price FROM products WHERE id = 1;` → **300** (transaction mới, thấy giá mới) | |

Dù Session B đã commit giá 300 từ T3, Session A vẫn thấy giá 100 cho đến khi kết thúc transaction của mình. Điều này đảm bảo **tính nhất quán trong suốt vòng đời của transaction**.

**Cơ chế bên dưới:** MySQL sử dụng **MVCC (Multi-Version Concurrency Control)** — lưu nhiều phiên bản dữ liệu để mỗi transaction đọc đúng phiên bản snapshot của mình mà không cần khóa dữ liệu.

**Khi nào dùng?** Đây là lựa chọn mặc định và phù hợp cho hầu hết các ứng dụng. Cân bằng tốt giữa hiệu suất và tính nhất quán.

---

### 4. Serializable — "Xếp hàng từng người một"

```sql
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

Cấp độ cách ly **cao nhất và nghiêm ngặt nhất**. Các transaction bị ép chạy tuần tự — transaction nào đến trước xử lý trước, transaction sau phải **chờ**.

**Ví dụ minh họa:**

| Thời điểm | Session A | Session B |
|-----------|-----------|-----------|
| T1 | `START TRANSACTION;` | |
| T2 | `SELECT * FROM products WHERE id = 1;` (Khóa bảng!) | |
| T3 | | `UPDATE products SET price = 400 WHERE id = 1;` → **BỊ BLOCK, CHỜ...** |
| T4 | `COMMIT;` | |
| T5 | | Câu UPDATE bây giờ mới được thực thi |

Session B muốn ghi dữ liệu nhưng bị **khóa** (lock) cho đến khi Session A hoàn tất. Nếu Session A chạy quá lâu, Session B có thể bị **timeout**.

**Khi nào dùng?** Chỉ khi dữ liệu **tuyệt đối không được phép sai** — ví dụ hệ thống tài chính, ngân hàng, hoặc các giao dịch liên quan đến tiền. Đổi lại, hiệu suất cực kỳ thấp vì hệ thống về cơ bản trở thành **đơn luồng trên mỗi bảng**.

---

## So sánh tổng quan

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Hiệu suất |
|---|---|---|---|---|
| **Read Uncommitted** | Có thể xảy ra | Có thể xảy ra | Có thể xảy ra | Cao nhất |
| **Read Committed** | Không | Có thể xảy ra | Có thể xảy ra | Cao |
| **Repeatable Read** | Không | Không | Có thể xảy ra* | Trung bình |
| **Serializable** | Không | Không | Không | Thấp nhất |

> *MySQL InnoDB thực tế ngăn được Phantom Read ở mức Repeatable Read nhờ cơ chế **gap locking**, nhưng theo lý thuyết SQL standard thì Phantom Read vẫn có thể xảy ra ở cấp độ này.

---

## Kiểm tra và thay đổi Isolation Level

```sql
-- Xem isolation level hiện tại
SELECT @@transaction_isolation;

-- Thay đổi cho session hiện tại
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Thay đổi global (ảnh hưởng tất cả session mới)
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

---


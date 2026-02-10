---
title: "ACID là gì? Tìm hiểu về các thuộc tính ACID trong Database"
date: 2026-02-09 21:38:00 +0700
categories: [Programming, Database]
tags: [Database, ACID, Transaction, SQL, Database Design, System Design, Data Integrity]
---

# ACID là gì? Tìm hiểu về các thuộc tính ACID trong Database

## Giới thiệu

ACID là viết tắt của bốn thuộc tính quan trọng đảm bảo tính tin cậy của các giao dịch (transactions) trong hệ thống cơ sở dữ liệu: **Atomicity** (Tính nguyên tử), **Consistency** (Tính nhất quán), **Isolation** (Tính cô lập), và **Durability** (Tính bền vững). Đây là những nguyên tắc nền tảng giúp đảm bảo dữ liệu luôn chính xác, an toàn và đáng tin cậy, đặc biệt quan trọng trong các ứng dụng yêu cầu độ chính xác cao như ngân hàng, thương mại điện tử, hay các hệ thống quản lý quan trọng.

---

## 1. Atomicity - Tính Nguyên Tử

### Khái niệm

Atomicity đảm bảo rằng một giao dịch được thực hiện **toàn bộ hoặc không thực hiện gì cả** (all-or-nothing). Nếu bất kỳ phần nào của giao dịch thất bại, toàn bộ giao dịch sẽ được rollback về trạng thái ban đầu.

### Đặc điểm chính

- **Hoàn thành hoặc rollback đầy đủ**: Giao dịch phải được commit hoàn toàn hoặc rollback hoàn toàn
- **Thực thi tất cả hoặc không**: Không có trạng thái trung gian
- **Đảm bảo không có cập nhật một phần**: Tránh tình trạng dữ liệu không đồng bộ

### Ví dụ thực tế

Giả sử bạn chuyển tiền từ tài khoản A sang tài khoản B:

```
Transaction:
Write 1: Trừ 100,000 VNĐ từ tài khoản A
Write 2: Cộng 100,000 VNĐ vào tài khoản B
Write 3: Ghi log giao dịch
Write 4: Cập nhật thời gian giao dịch
```

Nếu Write 2 thất bại (ví dụ: mất kết nối), thì Write 1, 3, 4 cũng phải được rollback. Tuyệt đối không được để tiền bị trừ ở tài khoản A mà không được cộng vào tài khoản B.

---

## 2. Consistency - Tính Nhất Quán

### Khái niệm

Consistency đảm bảo rằng một giao dịch sẽ đưa cơ sở dữ liệu từ một **trạng thái hợp lệ sang một trạng thái hợp lệ khác**. Tất cả các ràng buộc, quy tắc và logic nghiệp vụ đều phải được tuân thủ.

### Đặc điểm chính

- **Duy trì logic nghiệp vụ nhất quán**: Tất cả các quy tắc kinh doanh đều được áp dụng
- **Cân bằng tài khoản sau chuyển khoản**: Tổng tiền trước và sau giao dịch phải bằng nhau
- **Thực thi tuân thủ tất cả các quy tắc**: Constraints, triggers, và rules được kiểm tra

### Ví dụ thực tế

Trong hệ thống ngân hàng:

```
Trạng thái 1 (Consistent):
- Tài khoản A: 500,000 VNĐ
- Tài khoản B: 300,000 VNĐ
- Tổng: 800,000 VNĐ

Transactions: Chuyển 100,000 VNĐ từ A sang B

Trạng thái 2 (Consistent):
- Tài khoản A: 400,000 VNĐ
- Tài khoản B: 400,000 VNĐ
- Tổng: 800,000 VNĐ (vẫn bằng 800,000)
```

Các ràng buộc như "số dư không được âm", "tổng tiền không đổi" đều được đảm bảo.

---

## 3. Isolation - Tính Cô Lập

### Khái niệm

Isolation đảm bảo rằng các giao dịch đồng thời **không ảnh hưởng lẫn nhau**. Mỗi giao dịch được thực hiện như thể nó là giao dịch duy nhất đang chạy trên hệ thống.

### Đặc điểm chính

- **Ngăn chặn chồng chéo trong quá trình thực thi**: Tránh xung đột khi nhiều giao dịch cùng truy cập dữ liệu
- **Giữ các giao dịch độc lập**: Một giao dịch không nhìn thấy kết quả chưa commit của giao dịch khác
- **Tránh can thiệp trong hoạt động**: Đảm bảo tính toàn vẹn khi thực thi song song

### Ví dụ thực tế

Hai giao dịch chạy đồng thời:

```
Transaction A:          Transaction B:
Write 1: Đọc số dư      Write 3: Đọc số dư
Write 2: -100,000       Write 4: -50,000

Isolated (Cô lập) ✓
```

Nhờ Isolation, Transaction B không thấy kết quả chưa commit của Transaction A, tránh việc đọc dữ liệu không chính xác (dirty read).

### Các mức độ Isolation

1. **Read Uncommitted**: Cho phép đọc dữ liệu chưa commit
2. **Read Committed**: Chỉ đọc dữ liệu đã commit
3. **Repeatable Read**: Đảm bảo đọc lại cùng dữ liệu
4. **Serializable**: Mức độ cô lập cao nhất

---

## 4. Durability - Tính Bền Vững

### Khái niệm

Durability đảm bảo rằng sau khi một giao dịch được **commit thành công**, dữ liệu sẽ **được lưu trữ vĩnh viễn** ngay cả khi hệ thống gặp sự cố (mất điện, crash, lỗi phần cứng).

### Đặc điểm chính

- **Đảm bảo bản ghi tồn tại sau khi commit**: Dữ liệu được ghi vào bộ nhớ lâu dài
- **Bảo toàn dữ liệu quan trọng sau sự cố**: Có khả năng phục hồi
- **Duy trì dữ liệu ngay cả sau khi khôi phục**: Sử dụng các cơ chế như replication, backup

### Ví dụ thực tế

```
Transaction → Commit → Database → Replicated → Replica 1
                                              → Replica 2
```

Sau khi giao dịch được commit:
- Dữ liệu được ghi vào database chính
- Tự động sao chép sang Replica 1 và Replica 2
- Nếu database chính bị lỗi, dữ liệu vẫn tồn tại ở các replica

### Cơ chế đảm bảo Durability

- **Write-Ahead Logging (WAL)**: Ghi log trước khi thay đổi dữ liệu thực
- **Replication**: Sao chép dữ liệu sang nhiều server
- **Backup**: Sao lưu định kỳ
- **Journaling**: Ghi nhật ký thay đổi

---

## Tại sao ACID quan trọng?

### 1. Đảm bảo tính toàn vẹn dữ liệu
ACID giúp dữ liệu luôn chính xác, không bị mất mát hay không nhất quán.

### 2. Tin cậy trong môi trường đa người dùng
Cho phép nhiều người dùng truy cập đồng thời mà không gây xung đột dữ liệu.

### 3. Phục hồi sau sự cố
Hệ thống có thể khôi phục về trạng thái nhất quán sau khi gặp lỗi.

### 4. Tuân thủ yêu cầu nghiệp vụ
Đảm bảo các quy tắc kinh doanh luôn được thực thi đúng đắn.

---

## Các hệ quản trị cơ sở dữ liệu hỗ trợ ACID

### RDBMS truyền thống (Hỗ trợ đầy đủ ACID)
- PostgreSQL
- MySQL (InnoDB engine)
- Oracle Database
- Microsoft SQL Server
- SQLite

### NoSQL (Hỗ trợ một phần hoặc có thể cấu hình)
- MongoDB (hỗ trợ ACID từ version 4.0)
- Redis (hỗ trợ Atomicity với transactions)
- CouchDB

---

## ACID vs BASE

Trong thế giới NoSQL, người ta thường nhắc đến mô hình BASE thay vì ACID:

| ACID | BASE |
|------|------|
| Atomicity | Basically Available |
| Consistency | Soft state |
| Isolation | Eventually consistent |
| Durability | - |

**BASE** ưu tiên khả năng mở rộng (scalability) và sẵn sàng (availability) hơn tính nhất quán tức thời, phù hợp với các hệ thống phân tán lớn.

---



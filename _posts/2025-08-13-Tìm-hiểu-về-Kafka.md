---
title: "Tìm hiểu về Kafka(loading...)"
date: 2025-08-13 01:17:00  +0700
categories: [Học tập]
tags: [Học tập]
---

# Tìm hiểu về Kafka(loading...)

<p align="center">
  <img src="/assets/images/kafka/image.png" alt="Image title_1" />
</p>

Các khái niệm chính:

1. Producer: Các ứng dụng gửi dữ liệu vào Kafka
2. Consumer: Các ứng dụng đọc dữ liệu từ Kafka
3. Topic: Nơi phân loại các loại tin nhắn khác nhau
4. BrokerL: Máy chủ lưu trữ và phân phối tin nhắn

## Case Study

### Bước 1: Producer gửi dữ liệu

```
Website → Khách hàng đặt pizza
{
  "order_id": "ORD001",
  "customer": "Nguyễn Văn A",
  "pizza": "Margherita",
  "price": 150000
}
```

### Bước 2: Kafka nhận và phân phối

```
Kafka nhìn vào "order_id": "ORD001"
→ Tính hash: hash("ORD001") = 1
→ Gửi vào Partition 1 của Topic "orders"
→ Lưu ở Broker 2 (nơi có Partition 1)
```

### Bước 3: Consumer xử lý

```
Consumer A (Inventory Service):
- Đọc từ Partition 1
- "Cần pizza Margherita → Kiểm tra kho"

Consumer B (Payment Service):
- Đọc từ Partition 1
- "150,000 VNĐ → Xử lý thanh toán"

Consumer C (Email Service):
- Đọc từ Partition 1
- "Gửi email xác nhận cho Nguyễn Văn A"
```

**Tại sao lại thiết kế như vậy?**

- 3 consumer xử lý `song song`
- Giúp bảo toàn dữ liệu
- Dễ mở rộng

## So sánh với cách làm truyền thống

```
Đơn hàng 1 → Kho → Thanh toán → Email → ✅ (3 giây)
                ↓
Đơn hàng 2 → Kho → Thanh toán → Email → ✅ (6 giây)
                ↓
Đơn hàng 3 → Kho → Thanh toán → Email → ✅ (9 giây)

Xử lý 1000 đơn = 3000 giây = 50 phút!
```

**Với Kafka**

```
Đơn hàng 1 → Topic "orders" → Kho ↘
Đơn hàng 2 → Topic "orders" → Thanh toán → Cùng lúc!
Đơn hàng 3 → Topic "orders" → Email ↗

Xử lý 1000 đơn = 1000 giây = 16 phút!
```

### So sánh cụ thể hơn:

```
API → Database → Email → SMS → Push Notification
 ↓       ↓         ↓       ↓            ↓
1s  →   2s    →   1s   →  0.5s   →     0.5s = 5s/đơn hàng
```

Với Kafka

```
API → Kafka Topic → Database  (2s)
              ↓ → Email       (1s)
              ↓ → SMS         (0.5s) ← Cùng lúc!
              ↓ → Push        (0.5s)

Tổng thời gian = max(2s, 1s, 0.5s, 0.5s) = 2s/đơn hàng
```

### Lợi ích khác:

1. Khả năng chịu lỗi:

```
Truyền thống: Email service lỗi → Cả hệ thống dừng
Kafka: Email service lỗi → Kho + Thanh toán vẫn chạy bình thường
```

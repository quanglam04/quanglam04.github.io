---
title: "Kiến thức cơ bản về mạng"
date: 2025-08-25 01:17:00  +0700
categories: [Programming, Network]
tags: [networking]
---

---

# Kiến thức cơ bản cần nắm về mạng

1. Mạng là gì?

Mạng máy tính là một tập hợp các thiết bị như máy tính, điện thoại, router hay switch kết nối với nhau để truyền và nhận dữ liệu. Chẳng hạn, khi bạn truy cập Internet bằng laptop, bạn chính là một node trong mạng. Laptop gửi dữ liệu qua router Wi-Fi, router lại gửi dữ liệu ra ngoài Internet để đến máy chủ (server) của website bạn muốn vào.
Ví dụ: bạn gõ `youtube.com` → máy tính của bạn (client) gửi yêu cầu đến server của YouTube → server trả lại video cho bạn. Đây chính là một tương tác mạng cơ bản.

2. Mô hình mạng

Mô hình OSI là mô hình `7 tầng lý thuyết`, còn TCP/IP là mô hình thực tế được dùng trên Internet.
Ví dụ: Khi bạn gửi một email, nội dung bức thư đi qua tầng Application (ứng dụng email), rồi xuống tầng Transport (TCP chia nhỏ dữ liệu), sau đó đến tầng Internet (IP định tuyến gói tin), cuối cùng qua tầng Network Access (Wi-Fi/Ethernet) để ra ngoài. Ở chiều ngược lại, máy nhận sẽ lắp ghép lại toàn bộ quá trình này.

3. Giao thức mạng 

Giả sử bạn mở một website: trình duyệt sẽ sử dụng giao thức HTTP hoặc HTTPS để nói chuyện với web server. Nếu bạn xem video trực tuyến, thường thì UDP sẽ được sử dụng, bởi nó nhanh hơn TCP và không cần đảm bảo từng gói dữ liệu phải đến chính xác.
Ví dụ: Khi chơi game online, nếu một gói tin UDP bị mất, nhân vật của bạn có thể “giật” nhẹ nhưng trò chơi vẫn tiếp tục. Ngược lại, nếu dùng TCP, mỗi lần mất gói tin thì game sẽ bị đứng hình để đợi gửi lại, gây ra trải nghiệm rất tệ.

4. Địa chỉ IP và Domain 

Mỗi thiết bị đều có một địa chỉ IP. Bạn có thể kiểm tra địa chỉ IP của mình bằng cách mở Command Prompt

```
ipconfig
```

Bạn sẽ thấy thông tin như:

```
IPv4 Address. . . . . . . . . . . : 192.168.1.5
```
Đây chính là địa chỉ IP trong mạng nội bộ. Nếu bạn truy cập trang whatismyip.com
, bạn sẽ thấy địa chỉ Public IP của mình trên Internet.
Ví dụ khác: Khi bạn gõ `ptit.edu.vn` trên trình duyệt, DNS sẽ dịch tên miền này sang địa chỉ IP thực tế như `203.162.10.108`, để máy tính biết phải kết nối đến đâu.

5. Port và vai trò của nó 

Một máy tính có thể chạy nhiều dịch vụ cùng lúc. Port chính là cách phân biệt các dịch vụ đó. Ví dụ, khi bạn gõ `http://localhost:8080/` trong trình duyệt, bạn đang yêu cầu dịch vụ web trên cổng 8080 của máy tính.
Hãy thử gõ lệnh:
```
netstat -an
```

Bạn sẽ thấy một danh sách các cổng đang được mở, ví dụ:
```
TCP    0.0.0.0:80       LISTENING
TCP    0.0.0.0:3306     LISTENING
```
Điều này có nghĩa là port 80 đang phục vụ web, còn port 3306 đang phục vụ cơ sở dữ liệu MySQL.

6. Socket - điểm cuối của giao tiếp

Hãy tưởng tượng bạn muốn gọi điện cho một người bạn. Bạn cần biết số điện thoại (tương ứng với IP) và người đó đang dùng máy nào trong nhiều máy (tương ứng với port). Khi bạn có cả hai thông tin, bạn mới thực sự kết nối được. Đó chính là socket.
Ví dụ: một socket có thể được viết là `192.168.1.5:80`, nghĩa là dịch vụ web (port 80) trên máy tính có địa chỉ IP 192.168.1.5.

7. Mô hình Client/Server

Khi bạn mở Facebook trên điện thoại, ứng dụng (client) sẽ gửi yêu cầu đến server của Facebook để lấy dữ liệu (bài viết, ảnh, video). Server phản hồi lại và bạn nhìn thấy kết quả trên màn hình. Đây chính là mô hình client/server.
Trong khi đó, torrent hoạt động theo mô hình peer-to-peer. Khi bạn tải một bộ phim bằng torrent, máy tính của bạn vừa tải dữ liệu từ người khác (client), vừa chia sẻ dữ liệu cho người khác (server).

8. Công cụ kiểm tra và giám sát mạng
- `ping google.com` → kiểm tra xem bạn có kết nối đến Google được không.
- `nslookup facebook.com` → tìm địa chỉ IP thật sự của Facebook.
- `telnet smtp.gmail.com 25` → kiểm tra dịch vụ email của Gmail có đang mở port 25 không.

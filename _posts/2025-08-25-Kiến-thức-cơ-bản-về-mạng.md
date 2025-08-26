---
title: "Kiến thức cơ bản về mạng"
date: 2025-08-25 01:17:00  +0700
categories: [Programming, Network]
tags: [networking]
---

---

# Kiến thức cơ bản cần nắm về mạng

1. Mạng là gì?

- Mạng là một tập hợp các máy tính và thiết bị khác mà có thể thực hiện việc gửi và nhận dữ liệu cho nhau
  - Mỗi thiết bị trong một mạng gọi là một `node`
  - Một node có đầy đủ chức năng của một máy tính được gọi là `host`
- Mỗi node trên mạng có một địa chỉ, là một chuỗi byte định danh duy nhất node đó
  - Địa chỉ vật lý 
  - Địa chỉ được gán khác nhau trên các loại mạng khác nhau
    - Địa chỉ Internet/Wan/Public/External (ISP)
    - Địa chỉ LAN/Local/Internal
      - 10., 172.16, 172.31 và 192.168 được sử dụng cho mạng internal
      - 127.(127.0.0.1 hoặc hostname localhost) địa chỉ local loopback



2. Mô hình mạng

Mô hình OSI là mô hình `7 tầng lý thuyết`, còn TCP/IP là mô hình thực tế được dùng trên Internet.
Ví dụ: Khi bạn gửi một email, nội dung bức thư đi qua tầng Application (ứng dụng email), rồi xuống tầng Transport (TCP chia nhỏ dữ liệu), sau đó đến tầng Internet (IP định tuyến gói tin), cuối cùng qua tầng Network Access (Wi-Fi/Ethernet) để ra ngoài. Ở chiều ngược lại, máy nhận sẽ lắp ghép lại toàn bộ quá trình này.

<p align="center">
  <img src="/assets/images/network-basic/1.png" alt="Image title_1" />
</p>

3. Giao thức mạng 

- Giao thức truyền thông là tập các qui tắc, qui ước mà mọi thực thể tham gia truyền thông phải tuân theo để mạng có thể hoạt động tốt
- Phân loại theo phương thức hoạt động:
  - Giao thức hoạt động có kết nối (TCP)
    - Thiết lập kết nối
    - Truyền dữ liệu kèm theo cơ chế kiểm soát chặt chẽ
    - Hủy bỏ kết nối
  - Giao thức hoạt động không kết nối (UDP)
    - Quá trình truyền thông chỉ có một giai đoạn duy nhất là truyền dữ liệu, không có giai đoạn thiết lập kết nối cũng như hủy bỏ kết nối

<p align="center">
  <img src="/assets/images/network-basic/2.png" alt="Image title_1" />
</p>

4. Địa chỉ IP và Domain 

- Mỗi máy tính trên mạng IPv4 được định danh duy nhất bởi một địa chỉ 4 byte
  - Ex: 199.1.32.90
  - Giá trị của mỗi số trong địa chỉ là một byte không dấu có giá trị 0 - 255
  - IPv6 sử dụng 16 byte địa chỉ
- Domain Name System (DNS) được phát triển để dịch địa chỉ sang dạng con người dễ nhớ
  - Ex: DNS: ptit.edu.vn | IP: 203.162.10.108
- Một vài dải địa chỉ đặc biệt:
  - 10., 172.16, 172.31 và 192.168 được sử dụng cho mạng internal
  - 127.(127.0.0.1 hoặc hostname localhost) địa chỉ local loopback

5. Port

- Máy tính hiện đại thực hiện nhiều tiến trình khác nhau cùng lúc như Email, FTP, Web, ... -> thực hiện thông qua port
- Mỗi máy tính với một địa chỉ IP có 65,535 logic port từ 1->65,535. Mỗi port có thể cấp phát cho một dịch vụ

<p align="center">
  <img src="/assets/images/network-basic/5.png" alt="Image title_1" />
</p>

6. Socket - điểm cuối của giao tiếp

- socket là giao diện và là một cấu trúc truyền thông đóng vai trò như là một điểm cuối (end point) để truyền thông
- Một địa chỉ socket là một tổ hợp gồm 2 thành phần: địa chỉ IP và địa chỉ port
- Phân loại giao diện socket:
  - Stream socket (TCP Socket)
  - Datagram socket (UDP Socket)
  - Raw Socket

<p align="center">
  <img src="/assets/images/network-basic/3.png" alt="Image title_1" />
</p>

7. Mô hình Client/Server

Khi bạn mở Facebook trên điện thoại, ứng dụng (client) sẽ gửi yêu cầu đến server của Facebook để lấy dữ liệu (bài viết, ảnh, video). Server phản hồi lại và bạn nhìn thấy kết quả trên màn hình. Đây chính là mô hình client/server.
Trong khi đó, torrent hoạt động theo mô hình peer-to-peer. Khi bạn tải một bộ phim bằng torrent, máy tính của bạn vừa tải dữ liệu từ người khác (client), vừa chia sẻ dữ liệu cho người khác (server).

<p align="center">
  <img src="/assets/images/network-basic/4.png" alt="Image title_1" />
</p>

8. Công cụ kiểm tra và giám sát mạng
- `ping google.com` → kiểm tra xem bạn có kết nối đến Google được không.
- `nslookup facebook.com` → tìm địa chỉ IP thật sự của Facebook.
- `telnet smtp.gmail.com 25` → kiểm tra dịch vụ email của Gmail có đang mở port 25 không.

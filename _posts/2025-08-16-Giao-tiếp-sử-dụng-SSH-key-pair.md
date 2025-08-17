---
title: "Cơ chế xác thực SSH bằng cặp khóa"
date: 2025-08-16 01:17:00  +0700
categories: [Học tập]
tags: [Học tập]
---

# Cơ chế xác thực SSH bằng cặp khóa (SSH Key Pair Authentication)

Khi chúng ta kết nối vào một máy chủ thông qua SSH, có hai cách phổ biến để xác thực: dùng mật khẩu hoặc dùng cặp khóa SSH. Trong đó, xác thực bằng khóa được xem là an toàn hơn nhiều, vì không có mật khẩu nào được gửi và cũng tránh được nguy cơ brute-force. Để hiểu rõ hơn, hãy đi qua từng bước trong quá trình xác thực bằng cặp khóa:

<p align="center">
  <img src="/assets/images/auth-ssh/1.png" alt="Image title_1" />
</p>

## Cơ chế xác thực 

Mỗi cặp khóa SSH bao gồm một `private key` (khóa riêng) và một `public key` (khóa công khai)
- **Private Key**: luôn được giữ bí mật trên máy client (máy dùng để SSH đến Server)
- **Public Key**: được copy lên máy server, thường sẽ lưu trong file `~/.ssh/authorized_keys` của user. Public key có thể chia sẻ công khai, vì bản thân nó không đủ để đăng nhập nếu không có private key đi kèm

### Bước 1: Bắt đầu phiên SSH

Client mở kết nối đến máy chủ SSH (port 22). Sau đó hai bên tiến hành key exchange để thiết lập một kênh mã hóa tạm thời. Kể từ đây, mọi dữ liệu trao đổi đều được mã hóa, kể cả thông tin xác thực

Song song đó, client sẽ kiểm tra host key của máy chủ để đảm bảo rằng nó đang kết nối đến đúng máy chủ, chứ không phải kẻ giả mạo. Host key được lưu trong file `~/.ssh/knows_hosts` để những lần kết nối sau có thể so khớp

## Bước 2: Client đề nghị dùng public key

Sau khi kênh mã hóa đã sẵn sàng, client thông báo với server rằng nó muốn đăng nhập bằng phương thức publickey thay vì password. Client sẽ gửi thông tin về public key của mình để server kiểm tra

## Bước 3: Server kiểm tra public key

Server mở file `~/.ssh/authorized_keys` và tìm xem public key mà client đưa ra có nằm trong danh sách cho phép hay không. Nếu không có, server lập tức từ chối. Nếu có, quá trình đi tiếp sang bước challenge

## Bước 4: Server tạo "challenge"

Để đảm bảo rằng client thực sự nắm trong tay private key, server sẽ gửi một challenge - tức một chuỗi dữ liệu đặc biệt gắn liên với phiên làm việc (sessionID). Điểm quan trọng ở đây là challege luôn thay đổi, giúp ngăn chặn kẻ tấn công sử dụng lại dữ liệu cũ (replay attack)

## Bước 5: Client ký challenge bằng private key

Client nhận challenge từ server, sau đó dùng private key để tạo một chữ ký số (digital signature) lên dữ liệu này. Đây là bằng chứng duy nhất chứng minh client đang giữ private key

## Bước 6: Client gửi chữ ký cho server

Client gửi lại chữ ký vừa tạo. Lưu ý: private key không bao giờ được gửi đi, chỉ có chữ ký (một chuỗi dữ liệu đã mã hóa) được truyền.

## Bước 7: Server xác minh chữ ký bằng public key

Server dùng public key đã lưu sẵn để kiểm tra chữ ký. Nếu chữ ký hợp lệ, server tin rằng client đúng là chủ nhân của private key. Ngược lại, nếu sai, yêu cầu đăng nhập sẽ bị từ chối

## Bước 3: Phiên làm việc được mở

Khi xác thực thành công, client có thể mở một phiên shell, chạy lệnh hoặc thiết lập port-forwarding. Tất cả dữ liệu trao đổi tiếp theo đều được mã hóa bằng khóa phiên (session keys) đẫ được sinh ra từ bước key exchange ban đầu.

## Vì sao cách này an toàn?

Cơ chế này đảm bảo nhiều lớp an toàn:

- Private key không bao giờ rời khởi client
- Dữ liệu trao đổi được mã hóa từ đầu đến cuối, tránh nghe lén
- Challenge gắn với session, ngăn việc tái sử dụng dữ liệu cũ
- Có thể ràng buộc quyền bằng các tùy chọn trong `authorizied_keys`. ví dụ chỉ cho phép chạy một lệnh cố định, hoặc giới hạn IP

## Case Study: Kết nối SSH từ máy cá nhân đến VPS 

Giả sử chúng ta có một VPS với thông tin như sau:
- IP: `103.140.249.98`
- User: `prod`(dùng để phân biệt khi triển khai trên các môi trường dev/stagging/production)
- Server đã lưu public key trong file `~/.ssh/authorized_keys`
Mục tiêu: kết nối từ máy tính cá nhân (client) lên VPS hoàn toàn bằng SSH Key, không dùng mật khẩu

1. Chuẩn bị trên máy cá nhân (Client)

Trước tiên, cần tạo một cặp khóa SSH (private + public). Nếu chưa có, hãy tạo mới:

```
ssh-keygen -t ed25519 -C "trinhquanglam2k4@gmail.com"
```

- Tùy chọn `-t ed25519` chọn loại khóa hiện đại, gọn nhẹ và an toàn.
- Sau khi chạy lệnh, bạn sẽ được hỏi nơi lưu khóa (mặc định `~/.ssh/id_ed25519`) và passphrase (mật khẩu bảo vệ private key).

Kết quả:
- Private key: `~/.ssh/id_ed25519` (giữ bí mật tuyệt đối).
- public key: `~/.ssh/id_ed25519.pub` (có thể chia sẻ cho server).

2. Copy public key lên server

Trong case study này, public key đã được thêm vào `~/.ssh/authorized_keys` của user `prod`. Nghĩa là bước này hoàn tất.

Thông thường,có 2 cách phổ biến để đưa public key lên server:

- Cách 1: 
Tại cửa sổ terminal VPS, di chuyển đến vị trí file 'authorized_keys', chỉnh sửa file: 

```
nano authorized_keys
```

Tại máy client, copy nội dung trong file `.ssh/id_ed25519.pub` sau đó paste vào file `authorized_keys` trên server

- Cách 2:
Tại cửa sổ termial máy client:

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub prod@103.140.249.98
```

3. Kết nối SSH vào VPS

Giờ thì có thể đăng nhập thẳng mà không cần mật khẩu:

```
ssh prod@103.140.249.98
```

Lần đầu kết nối, bạn sẽ được hỏi xác nhận host key của server

```
The authenticity of host '103.140.249.98 (103.140.249.98)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?

```

Bạn gõ 'yes'. Từ đó về sau, fingerprint này sẽ được lưu trong `~/.ssh/known_hosts` trên máy của bạn
Nếu mọi thứ đúng, bạn sẽ thấy:

```
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-xx-generic x86_64)
prod@103.140.249.98:~$
```

<p align="center">
  <img src="/assets/images/auth-ssh/2.png" alt="Image title_1" />
</p>

Ngoài ra, có thể kết nối nhanh hơn bằng cách cấu hình file config tại thư mục .ssh như sau:

```
Host trinhlam
HostName 103.140.249.98
User prod
IdentityFile ~/.ssh/id_ed25519
```

Bây giờ, để kết nối, ta không cần ghi chi tiết ip của VPS, chỉ cần:

```
ssh trinhlam
```
<p align="center">
  <img src="/assets/images/auth-ssh/3.png" alt="Image title_1" />
</p>

Kết quả tương tự như cách làm ở trên. Bây giờ bạn đã kết nối đến VPS thành công. Tiến hành set-up cho việc deploy thôi!


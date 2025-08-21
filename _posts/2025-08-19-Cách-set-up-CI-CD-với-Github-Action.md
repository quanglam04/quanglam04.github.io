---
title: "Cách Set-up một hệ thống CI/CD sử dụng Github Action"
date: 2025-08-19 01:17:00  +0700
categories: [Học tập]
tags: [Học tập]
---


# CI/CD là gì? Cách thiết lập CI/CD với Github Actions (NodeJS + ReactJS)

## CI/CD là gì? (loading...)

CI/CD là tập hợp các hành động nhằm tự động hóa vòng đời phát triển phần mềm, trong đó:
- **CI - Continuous Integration**: khi push code, hệ thống tự cài dependencies -> build -> chạy lint/test. Mục tiêu để phát hiện lỗi sớm, giữ nhánh chính luôn xanh
- **CD - Continuous Delivery**: sau khi CI pass, hệ thống đóng gói artifact(ví dụ: như mục build của React) và sẵn sàng deploy 

## Github Actions - Các khái niệm cốt lõi 

- `Workflow`: 1 kịch bản tự động hóa (file .yml) chạy theo trigger (push, PR, cron, manual,...)
- `Job`: 1 nhóm bước chạy trên 1 máy ảo (runner). Nhiều job có thể chạy song song hoawjca theo needs
- `Step`: 1 lệnh đơn hoặc 1 Action
- `Action`: khối tái sử dụng (ví dụ actions/checkout, action/setup-node)
- `Runner`: máy thực thi (Ubuntu, Windows, macOS). Thường dùng ubuntu-latest
- `Secrets/Variables`: cấu hình nhạy cảm (token) & biến không nhạy cảm
- `Artifacts & Cache`: Lưu output (artiface) và cache dependencies để chạy nhanh hơn

## Case Study

Hiện tại đã có:
- VPS (OS: Ubuntu)
- Dùng NVM để quản lý version NodeJS
- Dùng pm2 để chạy và quản lys tiến trình NodeJS Backend

1. CI (trên Github Actions)

```
Code
```

2. CD (Deploy lên VPS qua SSH)

Ở job này, Github Action sẽ SSH vào VPS -> Kéo Code mới về -> cài deps -> build lại -> restart bằng pm2

Bước chuẩn bị trên VPS
1. Cài NodeJS qua nvm
2. Cài pm2 global: `npm i -g pm2`
3. Tạo key SSH để Github có thể truy cập VPS (không dùng mật khẩu)
- Trên máy local: `ssh-keygen -t ed25519`
- Copy public key (`.pub`) vào `~/.ssh/authorized_keys` trên VPS
- Thêm private key Github

```
Code
```

Như vậy flow sẽ là:
- Push code lên `main` -> CI chạy build/test
- Nếu pass -> deploy job SSH vào VPS, kéo Code, build lại, restart bằng PM2



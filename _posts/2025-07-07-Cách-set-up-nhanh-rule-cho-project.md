---
title: "Cách set-up nhanh rules cho project"
date: 2025-07-07 21:38:00  +0700
categories: [Programming, ProjectSetup]
tags: [rules, setup, project]
---

---

# Cách thiết lập quy tắc bảo vệ nhánh Git trong Github

## Bước 1: Điều hướng đến trang chủ kho lưu trữ tại Github

<p align="center">
  <img src="/assets/images/git-protect/1.png" alt="Image title_1" />
</p>

Sau đó nhấp vào tùy chọn cài đặt. Nhấp vào các nhánh để thiết lập quy tắc bảo vệ nhánh

<p align="center">
  <img src="/assets/images/git-protect/2.png" alt="Image title_1" />
</p>

## Bước 2: Nhấp vào tùy chọn _Thêm quy tắc_ như hiển thị bên dưới

<p align="center">
  <img src="/assets/images/git-protect/3.png" alt="Image title_1" />
</p>

## Bước 3: Thêm tên nhánh muốn bảo vệ vào trường tên nhánh

Nó cũng hỗ trợ các tùy chọn ký tự đại diện. Ví dụ: nếu muốn bảo vệ các nhánh bắt đầu bằng Release, bạn có thể sử dụng tên **Release** .

## Bước 4: Thêm các quy tắc bảo vệ vào nhánh

<p align="center">
  <img src="/assets/images/git-protect/4.png" alt="Image title_1" />
</p>

## Bước 5: Sau khi tạo thành công các quy tắc, tiến hành apply vào nhánh

<p align="center">
  <img src="/assets/images/git-protect/5.png" alt="Image title_1" />
</p>



Vậy là từ bây giờ, mỗi một commit vào trên nhánh chính `master` cần phải đảm bảo
- Ngăn chặn việc `push force`
- Phải qua review của `code owner`
- Phải qua `merge request` từ một nhánh khác, không được push trực tiếp

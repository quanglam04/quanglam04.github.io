---
title: "So sánh SPA (Single-Page-Application) và MPA (Mutilple-Page-Application)"
date: 2025-07-11 21:38:00  +0700
categories: [Web, Architecture, Frontend]
tags: [web,architecture,frontend]
---

---

# So sánh SPA (Single-Page-Application) và MPA (Mutilple-Page-Application)

MPA là Multiple Page Application, tức là những website truyền thống chuyển trang thì sẽ load lại toàn bộ trang web.

## Độ khó học

- SPA khó hơn so với MPA khi phải học thêm một đống thứ xung quanh js framework

## SEO

- MPA SEO tốt hơn so với SPA vì trả về source html ngay khi load trang web. Điều này sẽ giúp cho các hệ thống search-engine có thể tìm kiếm kết quả nhanh 1 cách chính xác

- SPA thì phải mất thời gian mới render ra html

- Những bot crawler hiện nay không đọc tốt những trang web mà SPA

**Tuy nhiên** vẫn có cách cải thiện điểm yếu này là render html ngay tại server luôn rồi trả về tương tự MPA. Đó chính là lý do **NextJS** framework ra đời

## UX

- SPA tăng trải nghiệm người dùng vì không phải tải lại toàn bộ trang web
- MPA tải lại cả trang web mỗi khi chuyển trang đem lai trải nghiệm không thân thiện

## Thân thiện dev

- SPA giúp phân chia rõ ràng code giữa frontend và backend => phát triển dễ dàng
- SPA có thể tải sử dụng các component dễ dàng
- MPA thì không có sự phân chia rõ ràng giữa frontend và backend, backend đôi khi cũng phải xử lý những công việc của frontend

## Tốc độ tải trang

- Load lần đầu: MPA nhanh hơn so với SPA
- Những lần chuyển trang tiếp theo: SPA nhanh hơn MPA
- Server bên SPA sẽ được giảm tải hơn khi so với MPA
- Server thiết kế cho SPA cũng có thể dùng được cho mobile => tiết kiệm thời gian

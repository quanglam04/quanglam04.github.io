---
title: "Giải thích về event-loop trong JavaScript"
date: 2025-11-13 01:17:00  +0700
categories: [thread, event-loop]
tags: [javascript]
---

# Giải thích về Event Loop trong JavaScript

## Tại sao Event Loop sinh ra

JavaScript ban đầu được thiết kế để chạy trong trình duyệt và nó có một đặc điểm quan trọng: chỉ có một luồng thực thi (single-threaded). Nghĩa là tại một thời điểm chỉ có một đoạn mã JavaScript đang được thực thi trong call stack. Nếu JavaScript gặp một việc “phải chờ” — ví dụ gọi mạng (HTTP), đọc file, chờ timer — thì nếu không có cơ chế đặc biệt, toàn bộ luồng đó sẽ bị dừng lại cho đến khi việc chờ kết thúc. Hệ quả thực tế: giao diện người dùng sẽ đứng hình, thao tác bị treo, hoặc trên server (Node.js) mọi kết nối khác sẽ phải chờ cho đến khi công việc đang chạy xong. Đó là lý do triết lý thiết kế: thay vì để luồng chính chờ, ta tách phần “chờ” ra cho môi trường thực thi (trình duyệt hoặc runtime như Node) xử lý — và chỉ khi xong mới báo lại để callback được thực thi. Event Loop là cơ chế trung tâm đảm bảo việc này diễn ra trơn tru: nó phối hợp giữa nơi thực thi JavaScript (call stack), các dịch vụ bên ngoài (Web APIs / hệ thống I/O), và các hàng đợi chứa callback chờ chạy. Nhờ Event Loop, JavaScript vẫn “đơn luồng” nhưng có thể xử lý nhiều nhiệm vụ bất đồng bộ gần như song song mà không cần tạo nhiều threads — đây chính là cách Node.js có thể phục vụ hàng chục nghìn kết nối với tài nguyên rất nhỏ.

## Các khái niệm cần nắm

<p align="center">
  <img src="/assets/images/event-loop/1.png" alt="Image title_1" />
</p>

Bức ảnh minh họa một JavaScript Runtime với vài khối chính: Call Stack, Web APIs, Task Queue (macrotask), Microtask Queue, và Event Loop.

1. **Call Stack** là nơi mã JavaScript được thực thi — think of it như một ngăn xếp (stack) gọi hàm. Khi bạn gọi một hàm, hàm đó được đẩy vào stack; nếu hàm gọi tiếp hàm khác thì hàm mới lên trên; khi hàm hoàn thành thì bị pop khỏi stack. Vì chỉ có một stack nên bất kỳ đoạn mã đồng bộ nặng nào (ví dụ vòng lặp tính toán lớn, đọc file đồng bộ) đều “block” toàn bộ stack, ngăn không cho bất kỳ callback nào khác chạy cho tới khi hoàn tất.

2. Web APIs trên ảnh là những API do môi trường cung cấp, không phải là phần cốt lõi của ngôn ngữ JavaScript. Trong trình duyệt, đó là `fetch`, `setTimeout`, DOM, localStorage... Trong Node.js môi trường tương đương (libuv, hệ thống I/O, threadpool) cung cấp các dịch vụ như TCP, file system, timers. Khi mã gọi `setTimeout` hay `fetch`, lệnh sẽ rời khỏi Call Stack và được xử lý bởi Web APIs; sau khi hoàn tất, kết quả (callback) sẽ được đưa vào một trong các hàng đợi chờ để Event Loop kéo vào xử lý.

3. Task Queue (thường gọi macrotask queue) là vùng chứa callback “lớn” — ví dụ hàm của `setTimeout`, `setInterval`, hoặc callback từ các sự kiện I/O. Microtask Queue là vùng chứa các tác vụ ưu tiên cao hơn: Promises `.then()`, `queueMicrotask`, MutationObserver (trình duyệt). Điểm mấu chốt: sau mỗi lần Call Stack rỗng, Event Loop luôn xử lý hết mọi microtask trước khi lấy một macrotask từ Task Queue. Vì vậy, callback Promise thường chạy “ngay sau” khối hiện tại kết thúc, còn `setTimeout(..., 0)` phải chờ cho tới khi microtasks được dọn sạch và một macrotask khác được lấy ra.

4. Event Loop là vòng lặp điều phối: nó kiểm tra Call Stack, nếu rỗng thì xử lý microtasks, sau đó lấy 1 macrotask, đưa vào Call Stack để thực thi; lặp đi lặp lại. Ảnh có biểu tượng vòng quay bên cạnh, tượng trưng cho chu kỳ không ngừng này. Trong Node.js có thêm các phase (timers, pending callbacks, idle/prepare, poll, check, close callbacks) do libuv quản lý, nhưng về bản chất vẫn là vòng lặp kiểm tra/đưa callback vào stack.

## Cơ chế hoạt động
1. Một đoạn mã đồng bộ bắt đầu chạy trong Call Stack. Call Stack có thể chứa nhiều hàm con do lời gọi lồng nhau.
2. Mã gặp một phép gọi bất đồng bộ, ví dụ `setTimeout(cb, 10)` hoặc một `fetch`. JavaScript sẽ đăng ký thao tác đó với Web APIs (hay với libuv nếu là Node). Lệnh này rời khỏi Call Stack ngay lập tức (không chờ kết quả). Web APIs hoặc runtime bắt đầu xử lý: setTimeout sẽ đếm thời gian; fetch sẽ gửi request; file read sẽ chạy ở threadpool.
3. Khi Call Stack rỗng (không còn mã đồng bộ đang chạy), Event Loop “chạy”: trước tiên nó kiểm tra Microtask Queue. Nếu có microtasks (Promise callbacks, `queueMicrotask`, v.v.), nó sẽ xử lý hết tất cả các microtask liên tiếp, thêm chúng vào Call Stack một cái một cho tới khi microtask queue rỗng. Sau khi microtasks được dọn sạch, Event Loop sẽ lấy một macrotask từ Task Queue (ví dụ callback của setTimeout hoặc một sự kiện I/O hoàn thành) và đưa vào Call Stack để thực thi. Khi macrotask đó xong, Event Loop lại quay lại và lặp lại: dọn microtasks, lấy macrotask, ... Chu kỳ tiếp tục vô hạn.

```bash
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => console.log('promise'));

console.log('end');
```
Bước chạy từng dòng: `console.log('start')` in ra ngay; `setTimeout` đăng ký timer ở Web APIs; `Promise.resolve().then(...)` tạo một microtask và đăng ký callback vào Microtask Queue; `console.log('end')` in ra. Khi đoạn đồng bộ kết thúc, Call Stack rỗng, Event Loop bắt đầu dọn Microtask Queue: in `promise`. Sau khi microtask rỗng, Event Loop lấy macrotask (callback của setTimeout) và in `timeout`. Kết quả cuối cùng: `start`, `end`, `promise`, `timeout`.

## Kết luận

Event Loop là giải pháp thông minh để giữ JavaScript nhẹ, đơn luồng nhưng vẫn xử lý được nhiều tác vụ bất đồng bộ hiệu quả. Những khái niệm Call Stack, Web APIs, Microtask và Task Queue là “các bánh răng” cơ bản trong máy móc này; Event Loop là người chỉ huy, dọn microtasks trước và sau đó xử lý macrotask, lặp liên tục. Hiểu sâu Event Loop không chỉ giúp bạn tránh những lỗi block/đơ mà còn cho bạn công cụ để tối ưu performant code—đặc biệt quan trọng khi làm backend với Node.js hoặc các ứng dụng frontend có nhiều thao tác async.

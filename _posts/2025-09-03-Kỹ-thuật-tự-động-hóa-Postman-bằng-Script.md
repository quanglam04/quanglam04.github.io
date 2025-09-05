---
title: "Kỹ thuật tự động hóa Postman bằng Script"
date: 2025-09-03 01:17:00  +0700
categories: [Programming, API Testing]
tags: [Postman, API, Automation, Testing, Scripting]
---

# Kỹ thuật viết Script để tự động hóa một số tác vụ trên Postman

Vấn đề đặt ra: Khi làm việc với Postman để test API, chúng ta thường phải thực hiện nhiều hao tác lặp lại như nhập lại `access_token`, copy dữ liệu từ response này để đưa sang request khác, hay viết tay các điều kiện kiểm thử. Nhưng việc này không chỉ mất thời gian mà còn dễ gây nhầm lẫn.

Vậy có cách nào để Postman tự động giúp chúng ta những thao tác này, thay vì phải lặp lại thủ công? Đây chính là lý do cần đến **script trong Postman**

Trước tiên chúng ta sẽ cùng tìm hiểu về cách làm truyền thống:

*Case Study:* Cần Test API Verify mã OTP. Để gọi được API này thì frontend cần gửi lên access_token để định danh người dùng.

Đầu tiên Login vào hệ thống để lấy access token 

<p align="center">
  <img src="/assets/images/postman/1.png" alt="Image title_1" />
</p>

Copy access token này dùng để call API Verify mã OTP

<p align="center">
  <img src="/assets/images/postman/2.png" alt="Image title_1" />
</p>

Nếu trong 1 hệ thống Backend có rất nhiều API cần phải verify access token từ frontend gửi lên, vậy thì việc test như thế này sẽ tốn rất nhiều thời gian.

## Biến môi trường

Trong Postman, chúng ta có thể tận dụng biến môi trường **(Environment Variables)** để quản lý và tái sử dụng dữ liệu. Các biến này có phạm vi áp dụng trong toàn bộ một collection, nghĩa là nhiều request khác nhau có thể dùng chung cùng một giá trị mà không cần phải nhập đi nhập lại.

Cách sử dụng cũng rất đơn giản: chỉ cần đưa những giá trị dùng chung (ví dụ như `access_token`, `base_url`, `refresh_token…`) vào biến môi trường. Khi cần gọi tới, bạn chỉ việc viết tên biến trong cặp ngoặc nhọn kép `{{ }}`. Ví dụ: `{{access_token}}` hoặc `{{base_url}}`.

Nhờ vậy, việc viết và quản lý request trở nên gọn gàng hơn, dễ bảo trì hơn, đồng thời hạn chế được sai sót do phải chỉnh sửa thủ công ở nhiều nơi.

Để tạo môi trường, tại thanh bên trái postman, tìm đến `Environments`. Tạo mới một môi trường và thêm các biến cần dùng cùng với giá trị của nó 

<p align="center">
  <img src="/assets/images/postman/3.png" alt="Image title_1" />
</p>

Bây giờ làm sao để sử dụng được biến môi trường này? Rất đơn giản. Tại góc phải màn hình Postman, chọn vào `Environments` và chọn môi trường vừa khởi tạo

Kết quả không thay đổi

<p align="center">
  <img src="/assets/images/postman/4.png" alt="Image title_1" />
</p>

Tiếp đến ta cần set giá trị của access token vừa nhận được từ server gửi về vào biến môi trường. Bôi đen giá trị access token, nhấn chuột phải -> Set as variable

<p align="center">
  <img src="/assets/images/postman/5.png" alt="Image title_1" />
</p>

Bây giờ để lấy được biến này ra, chỉ cần `{{access_token}}`

## [UPDATE] Cải thiến lần 1

Thay vì khi test từng API, chúng ta phải copy mã access token và gán vào `Auth Type: Bearer Token` của môi request, thì bây giờ chúng ta có thể gán cho toàn bộ `Collections` hay `Folder`, khi đó các request nằm bên trong sẽ kế thừa từ cha của nó

<p align="center">
  <img src="/assets/images/postman/6.png" alt="Image title_1" />
</p>

Bây giờ, thay vì khi test từng request, chúng ta phải gán thủ công access token vào đầu request, thì có thể lựa chọn `Inherit auth from parent` trong `Auth Type`. Kết quả không thay đổi

## [UPDATE] Cải thiến lần 2

Đây là vấn đề chính cần được thực hiện trong bài viết này, đó là làm thế nào để khi login xong, postman sẽ tự động gán được access token vào biến môi trường để chúng ta không phải làm thủ công bằng tay? Postman có hỗ trợ viết Script để tự động hóa việc này. Các dòng lệnh này sẽ được biên dịch và thực thi tương tự như mã JavaScript.

Tại thanh Tab Bar của request login, chọn vào `Scripts` -> `Post-response` (tức là sau khi api này được gọi)

<p align="center">
  <img src="/assets/images/postman/7.png" alt="Image title_1" />
</p>


```
pm.test("Login thành công", function () {
  pm.response.to.have.status(200);
  const response = pm.response.json();
  if (response.data && response.data.accessToken) {
    pm.environment.set("access_token", response.data.accessToken);
    console.log("Gán giá trị cho access_token thành công")
  }
});
```
**Giải thích:**

Đoạn Script trên được viết trong `Postman Tests Tab` với mục tiêu kiểm tra kết quả đăng nhập và tự động lưu lại `access_token` để tái sử dụng cho những request sau

- `pm.test("Login thành công", function () { ... })`: Đây là cách định nghĩa một test case trong Postman. Test này có tên là “Login thành công”. Nếu các điều kiện bên trong được thỏa mãn, test sẽ pass.
- `pm.response.to.have.status(200);`: Điều kiện đầu tiên là HTTP status code của response phải bằng 200, tức là request login được thực thi thành công.
- `const response = pm.response.json();`: Dòng này chuyển đổi response từ dạng JSON về object JavaScript, giúp chúng ta dễ dàng truy cập vào các trường dữ liệu bên trong.
- `if (response.data && response.data.accessToken) { ... }`: Kiểm tra xem trong response có tồn tại trường `data` và `accessToken` hay không. Nếu có, nghĩa là hệ thống đã trả về token hợp lệ.
- `pm.environment.set("access_token", response.data.accessToken);`: Nếu điều kiện đúng, Postman sẽ tự động gán giá trị `accessToken` vào biến môi trường `access_token`. Nhờ đó, ở những request tiếp theo, bạn chỉ cần gọi `{{access_token}}` thay vì nhập thủ công.
- `console.log("Gán giá trị cho access_token thành công")`: Thêm một thông báo ra console để dễ theo dõi, giúp chắc chắn rằng token đã được lưu thành công.

Kết quả sau khi chạy:

<p align="center">
  <img src="/assets/images/postman/8.png" alt="Image title_1" />
</p>

Ví dụ trên chỉ minh họa việc gán `access_token`. Trên thực tế, trong một hệ thống Backend, chúng ta thường phải quản lý thêm nhiều biến môi trường khác như `refresh_token`, `base_url`, `user_id`, hay các `API key`... Cách triển khai cũng tương tự: chỉ cần gán giá trị vào biến môi trường một lần, và Postman sẽ giúp bạn tái sử dụng ở mọi nơi trong collection.





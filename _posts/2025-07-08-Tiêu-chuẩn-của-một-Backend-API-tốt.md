---
title: "Tiêu chuẩn của một API Backend tốt"
date: 2025-07-08 21:38:00  +0700
categories: [Backend, API, BestPractices]
tags: [api, rest, backend]
---

---

# Các tiêu chuẩn của một API đảm bảo chất lượng tốt

## 1. Bảo mật API

Bảo mật API là một yêu cầu bắt buộc và cần để ở mức ưu tiên cao nhất cho bất kỳ một dự án nào. Chúng ta cần tuân thủ một số nguyên tắc sau

- Sử dụng giao thức HTTPS
- Tất cả các API chỉ cho phép đi qua một cổng duy nhất
- Đối với thông tin nhạy cảm cần được mã hóa
- Thực hiện phân quyền cho mỗi user
- Mã hóa mật khẩu trước khi lưu xuống cơ sở dữ liệu

## 2. Tiêu chuẩn RESTful API

**2.1 Quy ước đặt tên (Naming Convention)**

Để duy trì tính nhất quán giữa tất cả các dự án, chúng ta cần xác định một quy ước cho các trường hợp khác nhau mà chúng ta cho là quan trọng. Các quyết định được đưa ra nên dựa các tiêu chuẩn HTTP và cơ sở dữ liệu hoặc để nhất quán giữa các công nghệ trong hệ thống

**2.2 Các tham số của API (Payload)**

Để thống nhất giữa các bên tham gia vào dự án (Mobile, Web) dễ dàng trong việc tiếp cận và sử dụng API chúng ta cần quy định rõ ràng các tham số dưới dạng `cameCase`

```
{
    "firstName": "John",
    "lastName": "Doe",
    "phone": "0123-456-789",
    "email": "johndoe@email.com",
    "address": {
        "street": "Pham Van Dong",
        "district": "North Tu Liem",
        "city": "Hanoi",
        "country": "Vietnam",
        "postalCode": "100000",
        "text": ""
    }
}
```

**2.3 Cấu trúc tham số đầu ra của API (Response Structure)**

Các API response cần có cấu trúc rõ ràng bao gồm các thông tin về status, message và data. Ví dụ cụ thể cho một số API như sau

- POST /user/add

```
{
    "status": 201,
    "message": "Add user successful",
    "data": "1"
}
```

- PUT /user/update

```
{
    "status": 202,
    "message": "Update user successful",
    "data": null
}
```

- PATCH /user/{id}?enable=false

```
{
    "status": 204,
    "message": "Deactivate user successful",
    "data": null
}
```

- DELETE /user/{id}

```
{
    "status": 205,
    "message": "Delete user successful",
    "data": null
}
```

- GET /user/{id}

```
{
    "status": 200,
    "message": "user",
    "data": {
        "id": "1",
        "firstName": "John",
        "lastName": "Doe",
        "phone": "0123-456-789",
        "email": "johndoe@email.com",
        "address": {
            "street": "Pham Van Dong",
            "district": "North Tu Liem",
            "city": "Hanoi",
            "country": "Vietnam",
            "postalCode": "100000",
            "text": ""
        }
    }
}
```

- GET /users

```
{
    "status": 200,
    "message": "users",
    "data": [
        {
            "id": "1",
            "firstName": "John",
            "lastName": "Doe",
            "phone": "0123-456-789",
            "email": "johndoe@email.com",
            "address": {
                "street": "Pham Van Dong",
                "district": "North Tu Liem",
                "city": "Hanoi",
                "country": "Vietnam",
                "postalCode": "100000",
                "text": ""
            }
        },
        {
            "id": "2",
            "firstName": "Leo",
            "lastName": "Messi",
            "phone": "0123-456-456",
            "email": "leomessi@email.com",
            "address": {
                "street": "Dummy text",
                "district": "Unnkown",
                "city": "Maiami",
                "country": "USA",
                "postalCode": "Dummy text",
                "text": ""
            }
        }
    ]
}
```

**2.4 Chuyển tiếp khi có API gặp lỗi (Error Forward)**

Khi thông báo cho người dùng về lỗi request API nội dung phản hồi sẽ được trả về bằng cấu trúc sau:

- `400 BAD_REQUEST`: Mã trạng thái phản hồi cho biết máy chủ không thể hoặc sẽ không xử lý yêu cầu do lỗi nào đó được coi là lỗi máy khách
- `401 UNAUTHORIZED`: Mã trạng thái phản hồi cho biết yêu cầu của khách hàng chưa được hoàn thành vì thiếu thông tin xác thực hợp lệ cho tài nguyên được yêu cầu.
- `403 FORBIDDEN`: Mã trạng thái phản hồi cho biết máy chủ hiểu yêu cầu nhưng từ chối ủy quyền.
- `404 NOT_FOUND`: Mã trạng thái phản hồi cho biết máy chủ không thể tìm thấy tài nguyên được yêu cầu.
- `406 NOT_ACCEPTABLE`: Mã phản hồi lỗi máy khách cho biết rằng máy chủ không thể tạo phản hồi khớp với danh sách các giá trị có thể chấp nhận được xác định trong tiêu đề đàm phán nội dung chủ động của yêu cầu và máy chủ không sẵn lòng cung cấp đại diện mặc định.
- `409 CONFLICT`: Mã trạng thái phản hồi xung đột HTTP 409 cho biết xung đột yêu cầu với trạng thái hiện tại của tài nguyên đích.
- `500 INTERNAL_SERVER_ERROR`: Mã phản hồi lỗi máy chủ cho biết máy chủ gặp phải tình trạng không mong muốn khiến máy chủ không thể thực hiện yêu cầu.
- `502 BAD_GATEWAY`: Mã phản hồi lỗi máy chủ cho biết rằng máy chủ, trong khi hoạt động như một cổng hoặc proxy, đã nhận được phản hồi không hợp lệ từ máy chủ ngược dòng.
- `503 SERVICE_UNAVAILABLE`: Mã phản hồi lỗi máy chủ cho biết máy chủ chưa sẵn sàng xử lý yêu cầu.
- `504 GATEWAY_TIMEOUT`: Mã phản hồi lỗi máy chủ cho biết rằng máy chủ, trong khi hoạt động như một cổng hoặc proxy, đã không nhận được phản hồi kịp thời từ máy chủ ngược tuyến mà nó cần để hoàn thành yêu cầu.

**2.5 Đánh số phiên bản cho API (API version)**

Để thuận tiện cho việc bảo trì và nâng cấp về sau chúng ta nên đánh version cho các API trên headers

```
@GetMapping(path = "/welcome", headers = "apiVersion=v1.0")
public String welcome() {
    return "Welcome";
}
```

**2.6 Mã phản hồi của API (Response Status Code)**

- `200 OK`: Mã phản hồi trạng thái thành công cho biết yêu cầu đã thành công. Theo mặc định, phản hồi 200 được lưu vào bộ nhớ đệm.
- `201 CREATED`: Mã phản hồi trạng thái thành công cho biết yêu cầu đã thành công và dẫn đến việc tạo tài nguyên.
- `202 ACCEPTED`: Mã trạng thái phản hồi cho biết yêu cầu đã được chấp nhận để xử lý nhưng quá trình xử lý chưa hoàn tất; trên thực tế, quá trình xử lý có thể chưa bắt đầu. Yêu cầu cuối cùng có thể được thực hiện hoặc không, vì nó có thể không được phép khi quá trình xử lý thực sự diễn ra.
- `204 NO_CONTENT`: Mã phản hồi trạng thái thành công cho biết yêu cầu đã thành công nhưng khách hàng không cần phải điều hướng khỏi trang hiện tại.
- `205 RESET_CONTENT`: Trạng thái phản hồi yêu cầu khách hàng đặt lại chế độ xem tài liệu, chẳng hạn như xóa nội dung của biểu mẫu, đặt lại trạng thái canvas hoặc làm mới giao diện người dùng.
- `400 BAD_REQUEST`: Mã trạng thái phản hồi cho biết máy chủ không thể hoặc sẽ không xử lý yêu cầu do điều gì đó được coi là lỗi máy khách
- `401 UNAUTHORIZED`: Mã trạng thái phản hồi cho biết yêu cầu của khách hàng chưa được hoàn thành vì thiếu thông tin xác thực hợp lệ cho tài nguyên được yêu cầu.
- `403 FORBIDDEN`: Mã trạng thái phản hồi cho biết máy chủ hiểu yêu cầu nhưng từ chối ủy quyền.
- `404 NOT_FOUND`: Mã trạng thái phản hồi cho biết máy chủ không thể tìm thấy tài nguyên được yêu cầu.
- `406 NOT_ACCEPTABLE`: Mã phản hồi lỗi máy khách cho biết rằng máy chủ không thể tạo phản hồi khớp với danh sách các giá trị có thể chấp nhận được xác định trong tiêu đề đàm phán nội dung chủ động của yêu cầu và máy chủ không sẵn sàng cung cấp đại diện mặc định.
- `409 CONFLICT`: Mã trạng thái phản hồi xung đột HTTP 409 cho biết xung đột yêu cầu với trạng thái hiện tại của tài nguyên đích.
- `500 INTERNAL_SERVER_ERROR`: Mã phản hồi lỗi máy chủ cho biết máy chủ gặp phải tình trạng không mong muốn khiến máy chủ không thể thực hiện yêu cầu.
- `502 BAD_GATEWAY`: Mã phản hồi lỗi máy chủ cho biết rằng máy chủ, trong khi hoạt động như một cổng hoặc proxy, đã nhận được phản hồi không hợp lệ từ máy chủ ngược dòng.
- `503 SERVICE_UNAVAILABLE`: Mã phản hồi lỗi máy chủ cho biết máy chủ chưa sẵn sàng xử lý yêu cầu.
- `504 GATEWAY_TIMEOUT`: Mã phản hồi lỗi máy chủ cho biết rằng máy chủ, trong khi đóng vai trò là cổng hoặc proxy, đã không nhận được phản hồi kịp thời từ máy chủ ngược tuyến mà nó cần để hoàn thành yêu cầu.

**2.7 Tài liệu hóa API (API document)**

Hiện nay có rất nhiều công cụ khác nhau để tài liệu hóa các API như Postman, Swagger, OpenAPI, Excel. Việc tài liệu hóa các API là rất cần thiết cho việc chuyển giao giữa các team Backend và Frontend. Đặc biệt là chúng ta cần bàn giao lại tài liệu API cho khách hàng. Sau đây là một công cụ được các lập trình viên sử dụng rất phổ biến vì một số lý do: dễ dàng tích hợp, dễ dàng cấu hình, dễ sử dụng và có thể đồng bộ lập tức sau khi các APi được tạo mới hoặc cập nhật

<p align="center">
  <img src="/assets/images/tieu-chuan-backend/image.png" alt="Image title_1" />
</p>

**2.8. Đường dẫn API (Path/Routes)**

Việc không định nghĩa đường dẫn cho các API một cách rõ ràng có thể gây nên những bối rối khi sử dụng và điều hướng sai luồng cái API

- GET api/v1/user
- GET api/v1/user/{userId}
- GET api/v1/user/{userId}/orders
- GET api/v1/user/{userId}/orders?orderId=1
- POST api/v1/user
- PUT api/v1/user/{userId}
- PATCH api/v1/user/{userId}
- DELETE api/v1/user/{userId}

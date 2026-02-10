---
title: "So sánh POST, PATCH và PUT trong RESTful API"
date: 2026-02-02 21:38:00 +0700
categories: [Programming, Backend]
tags: [REST API, HTTP Methods, Web Development, Backend, API Design]
---



## Giới thiệu

Khi phát triển RESTful API, việc lựa chọn đúng HTTP method là vô cùng quan trọng. Ba phương thức thường gây nhầm lẫn nhất là **POST**, **PATCH** và **PUT**. Bài viết này sẽ giúp bạn hiểu rõ sự khác biệt và cách sử dụng chúng một cách chính xác.

## Tổng quan nhanh

| Method | Mục đích | Idempotent | Sử dụng khi |
|--------|----------|------------|-------------|
| **POST** | Tạo mới resource | Không | Tạo resource mới |
| **PUT** | Thay thế toàn bộ resource | Có | Cập nhật toàn bộ resource |
| **PATCH** | Cập nhật một phần resource | Có thể | Cập nhật một phần resource |

## 1. POST - Tạo mới resource

### Đặc điểm
- Dùng để **tạo mới** một resource
- **Không idempotent**: gọi nhiều lần sẽ tạo ra nhiều resource khác nhau
- Server quyết định URI của resource mới
- Response thường trả về status code **201 Created**

### Ví dụ

```http
POST /api/users
Content-Type: application/json

{
  "name": "Nguyễn Văn A",
  "email": "vana@example.com",
  "age": 25
}
```

**Response:**
```http
HTTP/1.1 201 Created
Location: /api/users/123

{
  "id": 123,
  "name": "Nguyễn Văn A",
  "email": "vana@example.com",
  "age": 25,
  "createdAt": "2026-02-03T10:00:00Z"
}
```

### Khi nào dùng POST?
- Tạo user mới
- Đăng bài viết mới
- Thêm sản phẩm vào giỏ hàng
- Upload file
- Các thao tác không idempotent

## 2. PUT - Thay thế toàn bộ resource

### Đặc điểm
- Dùng để **thay thế hoàn toàn** một resource hiện có
- **Idempotent**: gọi nhiều lần với cùng data cho kết quả giống nhau
- Client phải gửi **toàn bộ** thông tin của resource
- Nếu resource chưa tồn tại, có thể tạo mới (tùy thiết kế)
- Response thường trả về status code **200 OK** hoặc **204 No Content**

### Ví dụ

```http
PUT /api/users/123
Content-Type: application/json

{
  "name": "Nguyễn Văn A Updated",
  "email": "vana.new@example.com",
  "age": 26
}
```

**Response:**
```http
HTTP/1.1 200 OK

{
  "id": 123,
  "name": "Nguyễn Văn A Updated",
  "email": "vana.new@example.com",
  "age": 26,
  "updatedAt": "2026-02-03T11:00:00Z"
}
```

### Lưu ý quan trọng

Nếu bạn chỉ gửi một phần thông tin với PUT:

```http
PUT /api/users/123
Content-Type: application/json

{
  "name": "Nguyễn Văn A Updated"
}
```

Kết quả sẽ là:
```json
{
  "id": 123,
  "name": "Nguyễn Văn A Updated",
  "email": null,  // Bị mất
  "age": null     // Bị mất
}
```

### Khi nào dùng PUT?
- Cập nhật toàn bộ thông tin profile user
- Thay thế cấu hình hệ thống
- Cập nhật toàn bộ thông tin sản phẩm
- Khi bạn muốn đảm bảo tính nhất quán của toàn bộ resource

## 3. PATCH - Cập nhật một phần resource

### Đặc điểm
- Dùng để **cập nhật một phần** của resource
- **Có thể idempotent** (tùy implementation)
- Client chỉ cần gửi các field muốn thay đổi
- Response thường trả về status code **200 OK**
- Linh hoạt và hiệu quả hơn PUT trong nhiều trường hợp

### Ví dụ

```http
PATCH /api/users/123
Content-Type: application/json

{
  "name": "Nguyễn Văn A Updated"
}
```

**Response:**
```http
HTTP/1.1 200 OK

{
  "id": 123,
  "name": "Nguyễn Văn A Updated",  // Đã cập nhật
  "email": "vana@example.com",     // Giữ nguyên
  "age": 25,                        // Giữ nguyên
  "updatedAt": "2026-02-03T11:30:00Z"
}
```

### PATCH với JSON Patch

PATCH cũng hỗ trợ format **JSON Patch** (RFC 6902) cho các thao tác phức tạp:

```http
PATCH /api/users/123
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/name", "value": "Nguyễn Văn A Updated" },
  { "op": "add", "path": "/phone", "value": "0123456789" },
  { "op": "remove", "path": "/age" }
]
```

### Khi nào dùng PATCH?
- Cập nhật một vài field của user
- Đánh dấu trạng thái task (chỉ update status)
- Cập nhật số lượng sản phẩm trong giỏ hàng
- Toggle settings (bật/tắt chức năng)
- Khi bạn chỉ muốn thay đổi một phần resource

## So sánh chi tiết

### 1. Idempotency (Tính bất biến)

**Idempotent** có nghĩa là gọi request nhiều lần cho kết quả giống như gọi 1 lần.

```javascript
// PUT - Idempotent
// Gọi 10 lần với cùng data → kết quả giống nhau
PUT /api/users/123 { "name": "A", "age": 25 }
PUT /api/users/123 { "name": "A", "age": 25 }
PUT /api/users/123 { "name": "A", "age": 25 }
// → Luôn cho kết quả: { id: 123, name: "A", age: 25 }

// POST - Không Idempotent
// Gọi 10 lần → tạo ra 10 users khác nhau
POST /api/users { "name": "A", "age": 25 }
POST /api/users { "name": "A", "age": 25 }
POST /api/users { "name": "A", "age": 25 }
// → Tạo ra: user#1, user#2, user#3, ...

// PATCH - Có thể Idempotent
// Tùy thuộc vào implementation
PATCH /api/users/123 { "name": "A" }
PATCH /api/users/123 { "name": "A" }
// → Nếu chỉ update name: Idempotent
// → Nếu có increment counter: Không Idempotent
```

### 2. Payload size

```javascript
// User hiện tại
{
  "id": 123,
  "name": "Nguyễn Văn A",
  "email": "vana@example.com",
  "age": 25,
  "address": "123 Đường ABC, Hà Nội",
  "phone": "0123456789",
  "avatar": "https://example.com/avatar.jpg"
}

// Chỉ muốn đổi tên:

// PUT - Phải gửi TOÀN BỘ
PUT /api/users/123
{
  "name": "Nguyễn Văn B",           // Cần update
  "email": "vana@example.com",      // Không đổi nhưng phải gửi
  "age": 25,                         // Không đổi nhưng phải gửi
  "address": "123 Đường ABC, Hà Nội",// Không đổi nhưng phải gửi
  "phone": "0123456789",             // Không đổi nhưng phải gửi
  "avatar": "https://example.com/avatar.jpg" // Không đổi nhưng phải gửi
}
// Payload: ~200 bytes

// PATCH - Chỉ gửi phần thay đổi
PATCH /api/users/123
{
  "name": "Nguyễn Văn B"
}
// Payload: ~30 bytes
```

### 3. Error handling

```javascript
// PUT - Validate toàn bộ
PUT /api/users/123
{
  "name": "A"
  // Missing required fields: email, age
}
// → 400 Bad Request: "email is required, age is required"

// PATCH - Validate chỉ field được gửi
PATCH /api/users/123
{
  "name": "A"
}
// → 200 OK (chỉ validate name)

PATCH /api/users/123
{
  "email": "invalid-email"
}
// → 400 Bad Request: "email format invalid"
```

## Các trường hợp thực tế

### Case 1: Cập nhật profile user

```javascript
// Scenario: User muốn đổi tên và email

// Không nên dùng POST
POST /api/users/123/update
// → POST dùng để tạo mới, không phải update

// Có thể dùng PUT (nhưng không tối ưu)
PUT /api/users/123
{
  "name": "New Name",
  "email": "new@email.com",
  "age": 25,              // Phải gửi kèm
  "address": "...",       // Phải gửi kèm
  "phone": "..."          // Phải gửi kèm
}

// Nên dùng PATCH
PATCH /api/users/123
{
  "name": "New Name",
  "email": "new@email.com"
}
```

### Case 2: Toggle feature flag

```javascript
// Scenario: Bật/tắt chế độ Dark Mode

// Tốt nhất: PATCH
PATCH /api/users/123/settings
{
  "darkMode": true
}

// Không nên: PUT
PUT /api/users/123/settings
{
  "darkMode": true,
  "notifications": true,
  "language": "vi",
  // ... phải gửi tất cả settings khác
}
```

### Case 3: Increment counter

```javascript
// Scenario: Tăng view count cho bài viết

// Không Idempotent với PATCH
PATCH /api/posts/456
{
  "views": currentViews + 1
}
// → Gọi 10 lần = tăng 10 lần (race condition)

// Nên dùng POST cho action
POST /api/posts/456/view
// → Server xử lý increment an toàn
```

### Case 4: Thay thế document hoàn toàn

```javascript
// Scenario: Upload file cấu hình mới

// Nên dùng PUT
PUT /api/config/settings.json
{
  "version": "2.0",
  "features": { ... },
  "theme": { ... }
}
// → Thay thế toàn bộ file config cũ
```

## Best Practices

### 1. Đặt tên endpoint rõ ràng

```javascript
// Tốt
POST   /api/users              // Tạo user mới
GET    /api/users/123          // Lấy thông tin user
PUT    /api/users/123          // Thay thế toàn bộ user
PATCH  /api/users/123          // Cập nhật một phần user
DELETE /api/users/123          // Xóa user

// Không nên
POST   /api/createUser
POST   /api/updateUser
POST   /api/users/123/update
GET    /api/getUserById?id=123
```

### 2. Validate đúng cách

```javascript
// PUT - Validate như create mới
app.put('/api/users/:id', (req, res) => {
  const schema = Joi.object({
    name: Joi.string().required(),
    email: Joi.string().email().required(),
    age: Joi.number().required()
  });
  // Validate toàn bộ
});

// PATCH - Validate chỉ field có mặt
app.patch('/api/users/:id', (req, res) => {
  const schema = Joi.object({
    name: Joi.string().optional(),
    email: Joi.string().email().optional(),
    age: Joi.number().optional()
  });
  // Chỉ validate field được gửi
});
```

### 3. Response phù hợp

```javascript
// POST - Trả về resource mới + Location header
app.post('/api/users', (req, res) => {
  const user = createUser(req.body);
  res.status(201)
     .location(`/api/users/${user.id}`)
     .json(user);
});

// PUT - Trả về resource đã update hoặc 204 No Content
app.put('/api/users/:id', (req, res) => {
  const user = replaceUser(req.params.id, req.body);
  res.status(200).json(user);
  // Hoặc: res.status(204).send();
});

// PATCH - Trả về resource đã update
app.patch('/api/users/:id', (req, res) => {
  const user = updateUser(req.params.id, req.body);
  res.status(200).json(user);
});
```

### 4. Xử lý error nhất quán

```javascript
// Resource không tồn tại
PUT /api/users/999    → 404 Not Found
PATCH /api/users/999  → 404 Not Found

// Validation error
PUT /api/users/123    → 400 Bad Request
PATCH /api/users/123  → 400 Bad Request

// Unauthorized
PUT /api/users/123    → 401 Unauthorized
PATCH /api/users/123  → 401 Unauthorized

// Forbidden (không có quyền)
PUT /api/users/123    → 403 Forbidden
PATCH /api/users/123  → 403 Forbidden
```

## Ví dụ hoàn chỉnh với Express.js

```javascript
const express = require('express');
const app = express();
app.use(express.json());

// Database giả lập
let users = [
  { id: 1, name: "User 1", email: "user1@example.com", age: 25 }
];

// POST - Tạo mới user
app.post('/api/users', (req, res) => {
  const { name, email, age } = req.body;
  
  // Validate required fields
  if (!name || !email || !age) {
    return res.status(400).json({ 
      error: "name, email, age are required" 
    });
  }
  
  const newUser = {
    id: users.length + 1,
    name,
    email,
    age,
    createdAt: new Date().toISOString()
  };
  
  users.push(newUser);
  
  res.status(201)
     .location(`/api/users/${newUser.id}`)
     .json(newUser);
});

// PUT - Thay thế toàn bộ user
app.put('/api/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const { name, email, age } = req.body;
  
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    return res.status(404).json({ error: "User not found" });
  }
  
  // Validate required fields (giống như tạo mới)
  if (!name || !email || !age) {
    return res.status(400).json({ 
      error: "name, email, age are required" 
    });
  }
  
  // Thay thế toàn bộ (giữ lại id và createdAt)
  users[userIndex] = {
    id: userId,
    name,
    email,
    age,
    createdAt: users[userIndex].createdAt,
    updatedAt: new Date().toISOString()
  };
  
  res.status(200).json(users[userIndex]);
});

// PATCH - Cập nhật một phần user
app.patch('/api/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const updates = req.body;
  
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    return res.status(404).json({ error: "User not found" });
  }
  
  // Validate chỉ các field được gửi
  if (updates.email && !isValidEmail(updates.email)) {
    return res.status(400).json({ 
      error: "Invalid email format" 
    });
  }
  
  // Merge changes
  users[userIndex] = {
    ...users[userIndex],
    ...updates,
    id: userId, // Không cho phép thay đổi id
    updatedAt: new Date().toISOString()
  };
  
  res.status(200).json(users[userIndex]);
});

// DELETE - Xóa user
app.delete('/api/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    return res.status(404).json({ error: "User not found" });
  }
  
  users.splice(userIndex, 1);
  res.status(204).send();
});

// Helper function
function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

## Kết luận

### Chọn method nào?

**Dùng POST khi:**
- Tạo mới resource
- Thực hiện action (không phải CRUD thuần túy)
- Không biết URI của resource trước

**Dùng PUT khi:**
- Muốn thay thế TOÀN BỘ resource
- Cần đảm bảo tính idempotent
- Client biết chính xác URI của resource

**Dùng PATCH khi:**
- Chỉ cần cập nhật MỘT PHẦN resource
- Muốn tiết kiệm bandwidth
- Cập nhật linh hoạt các field


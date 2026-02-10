---
title: "REST Client VS Code - Complete Guide"
date: 2025-07-07 01:17:00  +0700
categories: [Tools, VSCode, API]
tags: [tools,vscode,api]
---

---


📋 Tổng quan
REST Client là extension mạnh mẽ cho VS Code giúp test API trực tiếp trong editor mà không cần tools bên ngoài như Postman.
✅ Ưu điểm so với Postman:

- Tích hợp sâu với VS Code: Không cần chuyển đổi app
- Version control: Có thể commit file .http cùng với code
- Lightweight: Không tốn tài nguyên như Postman
- Syntax highlighting: Đẹp mắt và dễ đọc
- Variables & Environment: Quản lý linh hoạt

❌ Nhược điểm:

- UI đơn giản: Không có GUI phong phú như Postman
- Team collaboration: Không có workspace sharing như Postman
- Advanced features: Thiếu một số tính năng nâng cao

🚀 Cài đặt

1. Mở VS Code
2. Vào Extensions (Ctrl+Shift+X)
3. Tìm "REST Client" by Huachao Mao
4. Install

📝 Cú pháp cơ bản

1. Phân cách requests

```
### Đây là comment để mô tả request
GET https://api.example.com/users

### Request khác - BẮT BUỘC có ### để phân cách
POST https://api.example.com/users
Content-Type: application/json

{
  "name": "John Doe"
}
```

⚠️ Lưu ý: `###` là BẮT BUỘC để phân cách các request. Không có `###` thì không hiện nút **Send Request**.

2. Cấu trúc request cơ bản

```
### [Mô tả request]
[METHOD] [URL]
[Headers]

[Body - nếu có]
```

## 🔧 Các loại request

`GET` Request

```
### Get all users
GET https://jsonplaceholder.typicode.com/users

### Get user by ID
GET https://jsonplaceholder.typicode.com/users/1
```

`POST` Request với JSON

```
### Create new post
POST https://jsonplaceholder.typicode.com/posts
Content-Type: application/json

{
  "title": "My New Post",
  "body": "This is the content of my post",
  "userId": 1
}
```

`PUT` Request

```
### Update post
PUT https://jsonplaceholder.typicode.com/posts/1
Content-Type: application/json

{
  "id": 1,
  "title": "Updated Title",
  "body": "Updated content",
  "userId": 1
}
```

`DELETE` Request

```
### Delete post
DELETE https://jsonplaceholder.typicode.com/posts/1
```

`PATCH` Request

```
### Partial update
PATCH https://jsonplaceholder.typicode.com/posts/1
Content-Type: application/json

{
  "title": "Only update title"
}
```

## 📄 Các loại Body

1. JSON Body

```
### JSON request
POST https://api.example.com/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}
```

2. Form Data

```
### Form data
POST https://httpbin.org/post
Content-Type: application/x-www-form-urlencoded

name=John&email=john@example.com&age=30
```

3. Multipart Form Data

```
### File upload
POST https://httpbin.org/post
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="name"

John Doe
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="test.txt"
Content-Type: text/plain

File content here
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

4. Raw Text

```
### Send raw text
POST https://httpbin.org/post
Content-Type: text/plain

This is raw text content
```

5. XML Body

```
### XML request
POST https://api.example.com/xml
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<user>
  <name>John Doe</name>
  <email>john@example.com</email>
</user>
```

## 🔐 Headers và Authentication

**Basic Headers**

```
### Request with headers
GET https://api.example.com/users
Content-Type: application/json
User-Agent: My App v1.0
X-Custom-Header: custom-value
```

**Bearer Token**

```
### With Bearer token
GET https://api.example.com/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Basic Authentication**

```
### Basic auth
GET https://api.example.com/secure
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

**API Key**

```
### API Key in header
GET https://api.example.com/data
X-API-Key: your-api-key-here
```

**Cookie**

```
### With cookies
GET https://api.example.com/dashboard
Cookie: sessionId=abc123; userId=456
```

## 🔧 Variables

1. File Variables

```
### Định nghĩa variables
@baseUrl = https://api.example.com
@token = your-jwt-token
@userId = 123

### Sử dụng variables
GET {{baseUrl}}/users/{{userId}}
Authorization: Bearer {{token}}

### Variables có thể dùng trong body
POST {{baseUrl}}/users
Content-Type: application/json

{
  "name": "User {{userId}}",
  "token": "{{token}}"
}
```

2. Environment Variables
   **Tạo file http-client.env.json trong thư mục root:**

```
{
  "development": {
    "baseUrl": "http://localhost:3000",
    "token": "dev-token-123",
    "apiKey": "dev-api-key"
  },
  "staging": {
    "baseUrl": "https://staging-api.example.com",
    "token": "staging-token-456",
    "apiKey": "staging-api-key"
  },
  "production": {
    "baseUrl": "https://api.example.com",
    "token": "prod-token-789",
    "apiKey": "prod-api-key"
  }
}
```

**Sử dụng:**

```
### Chọn environment từ status bar
GET {{baseUrl}}/users
Authorization: Bearer {{token}}
X-API-Key: {{apiKey}}
```

3. Response Variables

```
### Login và lấy token
POST {{baseUrl}}/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password"
}

### Sử dụng response từ request trước
GET {{baseUrl}}/profile
Authorization: Bearer {{login.response.body.token}}
```

## 🎯 Best Practices

1. Tổ chức file

```
### ========================================
### AUTH ENDPOINTS
### ========================================

### Login
POST {{baseUrl}}/auth/login
Content-Type: application/json

{
  "username": "{{username}}",
  "password": "{{password}}"
}

### ========================================
### USER ENDPOINTS
### ========================================

### Get all users
GET {{baseUrl}}/users
Authorization: Bearer {{token}}

### Get user by ID
GET {{baseUrl}}/users/{{userId}}
Authorization: Bearer {{token}}

### ========================================
### POST ENDPOINTS
### ========================================

### Create post
POST {{baseUrl}}/posts
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "title": "New Post",
  "content": "Post content"
}
```

2. Sử dụng Comments

```
### 📝 Create new user
### Description: This endpoint creates a new user
### Required: name, email
### Optional: age, phone
POST {{baseUrl}}/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}
```

3. Error Testing

```
### ✅ Success case
POST {{baseUrl}}/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com"
}

### ❌ Error case - Missing email
POST {{baseUrl}}/users
Content-Type: application/json

{
  "name": "John Doe"
}

### ❌ Error case - Invalid email
POST {{baseUrl}}/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "invalid-email"
}
```

### ⚡ Shortcuts

- Send Request: `Ctrl+Alt+R` (Windows/Linux) hoặc Cmd+Alt+R (Mac)
- Send All Requests: `Ctrl+Shift+P` → "Rest Client: Send All Requests"
- Cancel Request: `Ctrl+Alt+K`
- Switch Environment: Click vào environment ở status bar
- Rerun Last Request: `Ctrl+Alt+L`

### 🔧 Settings

Thêm vào settings.json:

```
{
  "rest-client.requestTimeout": 30000,
  "rest-client.followRedirect": true,
  "rest-client.defaultHeaders": {
    "User-Agent": "REST Client"
  },
  "rest-client.showResponseInDifferentTab": true,
  "rest-client.rememberCookiesForSubsequentRequests": true
}
```

### 🏆 Kết luận

REST Client phù hợp khi:

- Bạn đã làm việc trong VS Code
- Cần test API nhanh trong quá trình development
- Muốn version control API tests
- Team nhỏ, không cần collaboration phức tạp

Postman phù hợp khi:

- Cần GUI phong phú
- Team collaboration mạnh
- Cần features nâng cao (monitoring, mock server, etc.)
- Không quan tâm đến việc tích hợp với editor

Recommendation: Dùng REST Client cho development hàng ngày, Postman cho testing và documentation chính thức.

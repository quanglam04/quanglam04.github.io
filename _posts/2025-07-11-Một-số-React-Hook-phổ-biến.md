---
title: "Một số React Hook phổ biến"
date: 2025-07-11 21:38:00  +0700
categories: [web, Frontend]
tags: [web,frontend]
---


React Hooks là những function đặc biệt cho phép "hook into" (kết nối với) các tính năng của React trong function component. Hooks được giới thiệu từ React 16.8 và đã thay đổi cách viết React component.

---

## useState

### Khái niệm

#### State là gì?
Trước hết, để hiểu `useState`, cần hiểu **state** là gì. State (trạng thái) là dữ liệu có thể thay đổi trong component. Ví dụ:
- Giá trị trong ô input mà user đang gõ
- Danh sách sản phẩm trong giỏ hàng
- Trạng thái đăng nhập của user (đã login hay chưa)
- Số lượt like của một bài post

State quan trọng vì khi state thay đổi, React sẽ **tự động render lại** component để hiển thị dữ liệu mới lên màn hình.

#### Tại sao cần useState?
Có thể nghĩ: "Tại sao không dùng biến thông thường?"

```javascript
// ❌ CODE SAI - Không hoạt động như mong đợi
function Counter() {
  let count = 0; // Biến thông thường
  
  const handleClick = () => {
    count = count + 1;
    console.log(count); // In ra 1, 2, 3...
  };
  
  return (
    <div>
      <p>Count: {count}</p> {/* Luôn hiển thị 0 trên màn hình! */}
      <button onClick={handleClick}>Tăng</button>
    </div>
  );
}
```

**Vấn đề**: Mặc dù biến `count` có tăng lên (thấy trong console.log), nhưng màn hình KHÔNG cập nhật! Tại sao?

Vì React KHÔNG BIẾT biến `count` đã thay đổi. React chỉ render lại component khi:
1. Props thay đổi
2. **State thay đổi** (thông qua useState)
3. Parent component render lại

Hơn nữa, mỗi lần component render lại, function của component sẽ chạy lại từ đầu, và biến `count` sẽ bị reset về 0!

#### useState giải quyết vấn đề như thế nào?

`useState` làm 2 việc quan trọng:
1. **Lưu trữ giá trị giữa các lần render**: React ghi nhớ giá trị state, không bị reset mỗi lần render
2. **Trigger re-render khi state thay đổi**: Khi gọi setState, React biết cần render lại component

```javascript
// ✅ CODE ĐÚNG
function Counter() {
  const [count, setCount] = useState(0);
  // count: giá trị hiện tại (được React ghi nhớ)
  // setCount: function để thay đổi count VÀ báo React render lại
  
  const handleClick = () => {
    setCount(count + 1); // React sẽ render lại component với count mới
  };
  
  return (
    <div>
      <p>Count: {count}</p> {/* Hiển thị giá trị mới! */}
      <button onClick={handleClick}>Tăng</button>
    </div>
  );
}
```

#### Cơ chế hoạt động
1. Lần render đầu tiên: `useState(0)` tạo state với giá trị 0
2. User click button → `setCount(1)` được gọi
3. React đánh dấu component cần render lại
4. Component render lại, `useState(0)` giờ trả về giá trị 1 (không phải 0!)
5. Màn hình cập nhật hiển thị 1

### Cú pháp
```javascript
const [state, setState] = useState(initialValue);
```

- `state`: giá trị hiện tại của state
- `setState`: function để cập nhật state
- `initialValue`: giá trị khởi tạo ban đầu

### Ví dụ cơ bản

```javascript
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Bạn đã click {count} lần</p>
      <button onClick={() => setCount(count + 1)}>
        Tăng
      </button>
      <button onClick={() => setCount(count - 1)}>
        Giảm
      </button>
      <button onClick={() => setCount(0)}>
        Reset
      </button>
    </div>
  );
}
```

### Ví dụ với Object

```javascript
function Form() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setUser(prevUser => ({
      ...prevUser,
      [name]: value
    }));
  };

  return (
    <form>
      <input
        name="name"
        value={user.name}
        onChange={handleChange}
        placeholder="Tên"
      />
      <input
        name="email"
        value={user.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        name="age"
        value={user.age}
        onChange={handleChange}
        placeholder="Tuổi"
      />
    </form>
  );
}
```

### Lưu ý quan trọng
- State update có thể bất đồng bộ
- Khi update state dựa trên giá trị cũ, nên dùng functional update: `setState(prev => prev + 1)`
- State chỉ được khởi tạo lần đầu tiên component render

---

## useEffect

### Khái niệm

#### Side Effect là gì?
**Side effect** (tác động phụ) là những hành động ảnh hưởng đến thứ gì đó BÊN NGOÀI component, hoặc những hành động không liên quan trực tiếp đến việc render UI.

Hãy tưởng tượng React component như một cỗ máy:
- **Input**: Props và State
- **Output**: JSX (UI hiển thị)
- **Side effects**: Mọi thứ khác ngoài việc tính toán JSX

Ví dụ về side effects:
1. **Fetch data từ API**: Gọi server để lấy dữ liệu
2. **Thao tác với DOM trực tiếp**: Thay đổi title trang, focus vào input
3. **Subscribe/Unsubscribe**: Đăng ký nhận thông báo từ WebSocket, event listener
4. **Timer**: setTimeout, setInterval
5. **Ghi log**: console.log, gửi analytics
6. **Lưu vào localStorage**: Lưu data vào trình duyệt
7. **Kết nối external service**: Google Maps, payment gateway

#### Tại sao cần useEffect?

**Vấn đề 1: Không thể thực hiện side effect trực tiếp trong component body**

```javascript
// ❌ CODE SAI - ĐỪNG LÀM NHƯ NÀY!
function BadComponent() {
  const [data, setData] = useState(null);
  
  // Fetch ngay trong body của component
  fetch('https://api.example.com/data')
    .then(res => res.json())
    .then(data => setData(data)); // Gọi setData → trigger render
    // → Component render lại → fetch lại → setData lại → vòng lặp vô hạn! 💥
  
  return <div>{data?.name}</div>;
}
```

**Vấn đề**: Mỗi lần component render, fetch API được gọi lại → setData → trigger render mới → fetch lại → **vòng lặp vô tận**! Trình duyệt bị treo, API server bị spam.

**Vấn đề 2: Không biết khi nào component "đã render xong"**

Nhiều side effect cần thực hiện SAU KHI component đã render xong lên màn hình (ví dụ: focus vào input, đo kích thước DOM element). Nhưng làm sao biết được component đã render xong?

**Vấn đề 3: Không có cách cleanup khi component bị unmount**

Ví dụ subscribe vào WebSocket. Khi user rời khỏi trang, component bị xóa (unmount), nhưng connection WebSocket vẫn còn → **memory leak** (rò rỉ bộ nhớ).

#### useEffect giải quyết như thế nào?

`useEffect` cung cấp một "vùng an toàn" để thực hiện side effects với 3 tính năng quan trọng:

**1. Chạy SAU KHI render hoàn thành**
```javascript
function Component() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    // Code này chạy SAU KHI JSX đã được render lên màn hình
    document.title = `Bạn đã click ${count} lần`;
  });
  
  return <button onClick={() => setCount(count + 1)}>Click</button>;
}
```

Flow:
1. Component render → JSX trả về
2. React cập nhật DOM → Màn hình hiển thị button
3. **Sau đó** useEffect mới chạy → Cập nhật title

**2. Kiểm soát khi nào effect chạy (dependencies)**
```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // Chỉ chạy khi userId thay đổi, không chạy khi component render vì lý do khác
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]); // Dependency array
  
  return <div>{user?.name}</div>;
}
```

**Dependencies array giải thích chi tiết**:
- **Không có array**: Effect chạy sau MỌI lần render
  ```javascript
  useEffect(() => {
    console.log('Chạy sau mỗi render');
  }); // Không có dependencies
  ```

- **Empty array `[]`**: Effect chỉ chạy 1 lần sau lần render đầu tiên
  ```javascript
  useEffect(() => {
    console.log('Chỉ chạy 1 lần khi component mount');
  }, []); // Empty dependencies
  ```

- **Có dependencies `[a, b]`**: Effect chạy khi `a` hoặc `b` thay đổi
  ```javascript
  useEffect(() => {
    console.log('Chạy khi count hoặc userId thay đổi');
  }, [count, userId]);
  ```

**Tại sao cần dependencies?**
- **Tối ưu performance**: Không chạy effect không cần thiết
- **Tránh infinite loop**: Nếu effect gọi setState mà không có dependencies, sẽ tạo vòng lặp vô hạn
- **Logic rõ ràng**: Dễ hiểu effect phụ thuộc vào giá trị nào

**3. Cleanup function để "dọn dẹp"**
```javascript
function ChatRoom({ roomId }) {
  useEffect(() => {
    // Setup: Kết nối WebSocket
    const socket = connectToRoom(roomId);
    socket.on('message', handleMessage);
    
    // Cleanup: Ngắt kết nối khi component unmount hoặc roomId thay đổi
    return () => {
      socket.disconnect();
    };
  }, [roomId]);
}
```

**Khi nào cleanup chạy?**
- Khi component **unmount** (bị xóa khỏi màn hình)
- TRƯỚC KHI effect chạy lại (khi dependencies thay đổi)

**Tại sao cần cleanup?**

Ví dụ thực tế:
```javascript
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    // ❌ Nếu không cleanup
    // Khi component unmount, interval vẫn chạy
    // → gọi setSeconds trên component không tồn tại → ERROR!
    // → Memory leak
    
    // ✅ Cleanup đúng cách
    return () => clearInterval(interval);
  }, []);
}
```

#### Lifecycle của useEffect

Hiểu lifecycle giúp debug và viết code đúng:

```javascript
function Example({ prop }) {
  const [state, setState] = useState(0);
  
  useEffect(() => {
    console.log('1. Effect chạy');
    
    return () => {
      console.log('2. Cleanup chạy');
    };
  }, [prop]);
  
  return <div>...</div>;
}
```

**Kịch bản 1: Component mount lần đầu**
```
1. Component render
2. React cập nhật DOM
3. "1. Effect chạy" (không có cleanup vì lần đầu)
```

**Kịch bản 2: prop thay đổi (effect chạy lại)**
```
1. Component render với prop mới
2. React cập nhật DOM
3. "2. Cleanup chạy" (cleanup của effect trước đó)
4. "1. Effect chạy" (effect mới với prop mới)
```

**Kịch bản 3: Component unmount**
```
1. "2. Cleanup chạy"
2. Component bị xóa
```

#### Các trường hợp sử dụng phổ biến

**1. Fetch data khi component mount**
```javascript
useEffect(() => {
  fetch('/api/data')
    .then(res => res.json())
    .then(setData);
}, []); // Chỉ fetch 1 lần
```

**2. Fetch data khi ID thay đổi**
```javascript
useEffect(() => {
  fetch(`/api/users/${userId}`)
    .then(res => res.json())
    .then(setUser);
}, [userId]); // Fetch lại khi userId thay đổi
```

**3. Subscribe/Unsubscribe**
```javascript
useEffect(() => {
  const subscription = eventEmitter.on('update', handleUpdate);
  return () => subscription.unsubscribe();
}, []);
```

**4. Sync với external system**
```javascript
useEffect(() => {
  // Sync title với count
  document.title = `Count: ${count}`;
  // Không cần cleanup vì chỉ update giá trị
}, [count]);
```

### Cú pháp
```javascript
useEffect(() => {
  // Code thực thi side effect
  
  return () => {
    // Cleanup function (optional)
  };
}, [dependencies]);
```

### Ví dụ 1: Fetch data khi component mount

```javascript
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    fetch(`https://api.example.com/users/${userId}`)
      .then(response => response.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]); // Chỉ chạy lại khi userId thay đổi

  if (loading) return <p>Đang tải...</p>;
  return <div>{user?.name}</div>;
}
```

### Ví dụ 2: Subscription và Cleanup

```javascript
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    // Subscribe
    const socket = connectToRoom(roomId);
    socket.on('message', (message) => {
      setMessages(prev => [...prev, message]);
    });

    // Cleanup function
    return () => {
      socket.disconnect();
    };
  }, [roomId]);

  return (
    <div>
      {messages.map(msg => <p key={msg.id}>{msg.text}</p>)}
    </div>
  );
}
```

### Ví dụ 3: Timer

```javascript
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // Cleanup: Clear interval khi component unmount
    return () => clearInterval(interval);
  }, []); // [] nghĩa là chỉ chạy một lần khi mount

  return <p>Đã trôi qua {seconds} giây</p>;
}
```

### Các trường hợp dependency array

```javascript
// 1. Không có dependency array - chạy sau mỗi render
useEffect(() => {
  console.log('Chạy sau mỗi render');
});

// 2. Empty array [] - chỉ chạy một lần khi mount
useEffect(() => {
  console.log('Chỉ chạy một lần');
}, []);

// 3. Có dependencies - chạy khi dependencies thay đổi
useEffect(() => {
  console.log('Chạy khi count thay đổi');
}, [count]);
```

---

## useContext

### Khái niệm

#### Vấn đề: Prop Drilling (Địa ngục truyền Props)

Trước khi hiểu `useContext`, cần hiểu vấn đề mà nó giải quyết: **Prop Drilling**.

Hãy tưởng tượng ứng dụng có cấu trúc như này:

```
App (có thông tin user)
  └── Header
      └── Navigation
          └── UserMenu (cần thông tin user)
```

Để `UserMenu` có được thông tin `user` phải truyền props qua 3 cấp:

```javascript
// ❌ Prop Drilling - Mệt mỏi và dễ lỗi
function App() {
  const [user, setUser] = useState({ name: 'John', avatar: 'pic.jpg' });
  
  return <Header user={user} />; // Truyền xuống Header
}

function Header({ user }) {
  // Header không dùng user, chỉ truyền tiếp
  return <Navigation user={user} />; // Truyền xuống Navigation
}

function Navigation({ user }) {
  // Navigation cũng không dùng user, chỉ truyền tiếp
  return <UserMenu user={user} />; // Truyền xuống UserMenu
}

function UserMenu({ user }) {
  // Cuối cùng mới dùng user!
  return <div>{user.name}</div>;
}
```

**Vấn đề của Prop Drilling**:
1. **Component trung gian không cần data nhưng phải nhận props**: `Header` và `Navigation` không dùng `user` nhưng phải khai báo prop
2. **Khó maintain**: Nếu thêm field mới vào `user`, phải sửa tất cả component trung gian
3. **Coupling cao**: Component con phụ thuộc vào cấu trúc component cha
4. **Code dài dòng**: Phải viết đi viết lại `user={user}` nhiều lần

#### Context giải quyết như thế nào?

**Context** giống như một "kho dữ liệu chung" mà mọi component trong cây component đều có thể truy cập TRỰC TIẾP, không cần truyền props qua từng cấp.

Tưởng tượng như thế này:
- **Không có Context**: Muốn gửi thư cho người ở tầng 10, phải đưa thư cho người tầng 1, họ đưa cho tầng 2, ... tầng 9, rồi mới đến tầng 10
- **Có Context**: Gọi điện thoại trực tiếp cho người tầng 10 - KHÔNG cần qua trung gian!

```javascript
// ✅ Dùng Context - Sạch sẽ và đơn giản
// 1. Tạo Context
const UserContext = createContext();

function App() {
  const [user, setUser] = useState({ name: 'John', avatar: 'pic.jpg' });
  
  // 2. Bọc component trong Provider, cung cấp giá trị
  return (
    <UserContext.Provider value={user}>
      <Header /> {/* Không cần truyền user prop! */}
    </UserContext.Provider>
  );
}

function Header() {
  return <Navigation />; // Không cần truyền user prop!
}

function Navigation() {
  return <UserMenu />; // Không cần truyền user prop!
}

function UserMenu() {
  // 3. Lấy giá trị TRỰC TIẾP từ Context
  const user = useContext(UserContext);
  return <div>{user.name}</div>;
}
```

**Lợi ích**:
- `Header` và `Navigation` không cần biết gì về `user`
- `UserMenu` lấy data trực tiếp từ Context
- Dễ thêm/sửa data mà không ảnh hưởng component trung gian

#### Cách Context hoạt động

Context hoạt động theo mô hình **Provider-Consumer**:

**1. Provider (Nhà cung cấp)**:
- Là component "chứa" data
- Cung cấp data cho tất cả component con bên trong nó

```javascript
<UserContext.Provider value={userData}>
  {/* Tất cả component con đều có thể truy cập userData */}
  <App />
</UserContext.Provider>
```

**2. Consumer (Người tiêu dùng)**:
- Là component cần dùng data từ Context
- Dùng `useContext()` để "lấy" data

```javascript
const userData = useContext(UserContext);
```

**3. Context Value**:
- Có thể là bất cứ kiểu dữ liệu nào: object, array, string, function...
- Thường là object chứa cả data và function để update data

```javascript
// Value phức tạp với data và functions
<UserContext.Provider value={{
  user: user,
  login: loginFunction,
  logout: logoutFunction
}}>
```

#### Khi nào nên dùng Context?

**✅ Nên dùng Context khi**:
1. **Dữ liệu global**: Theme, ngôn ngữ, thông tin user đăng nhập
2. **Nhiều component cần dữ liệu**: Ít nhất 3-4 cấp component
3. **Dữ liệu ít thay đổi**: Không update liên tục (vì mỗi lần update, tất cả consumer đều re-render)

**❌ KHÔNG nên dùng Context khi**:
1. **Chỉ 1-2 cấp component**: Prop drilling đơn giản hơn
2. **Dữ liệu thay đổi thường xuyên**: Gây re-render nhiều, performance kém
3. **Dữ liệu local**: Chỉ 1-2 component cần

#### Context vs Props

| Context | Props |
|---------|-------|
| Data "nhảy" qua nhiều cấp | Data truyền từng cấp một |
| Tất cả component con đều truy cập được | Chỉ component nhận props mới có |
| Khó trace data flow | Dễ trace: nhìn props là biết data từ đâu |
| Dùng cho global state | Dùng cho local state |
| Coupling thấp giữa component | Coupling cao giữa cha-con |

#### Multiple Contexts

Có thể dùng nhiều Context cùng lúc:

```javascript
function App() {
  return (
    <ThemeContext.Provider value={theme}>
      <UserContext.Provider value={user}>
        <LanguageContext.Provider value={language}>
          <MainApp />
        </LanguageContext.Provider>
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

function SomeComponent() {
  const theme = useContext(ThemeContext);
  const user = useContext(UserContext);
  const language = useContext(LanguageContext);
  // Sử dụng cả 3 context
}
```

#### Performance Consideration

**Vấn đề**: Khi Context value thay đổi, TẤT CẢ component dùng useContext đều re-render.

```javascript
function App() {
  const [user, setUser] = useState({ name: 'John' });
  const [count, setCount] = useState(0);
  
  return (
    <UserContext.Provider value={{ user, count }}>
      {/* Khi count thay đổi, tất cả component dùng UserContext đều re-render
          ngay cả khi chúng chỉ dùng user, không dùng count! */}
      <ChildComponents />
    </UserContext.Provider>
  );
}
```

**Giải pháp**: Tách Context hoặc dùng useMemo

```javascript
// Tách thành 2 Context riêng
<UserContext.Provider value={user}>
  <CountContext.Provider value={count}>
    <ChildComponents />
  </CountContext.Provider>
</UserContext.Provider>

// Hoặc dùng useMemo
const contextValue = useMemo(() => ({ user, count }), [user, count]);
<UserContext.Provider value={contextValue}>
```

### Cú pháp
```javascript
const value = useContext(MyContext);
```

### Ví dụ đầy đủ

```javascript
import React, { createContext, useContext, useState } from 'react';

// 1. Tạo Context
const ThemeContext = createContext();

// 2. Tạo Provider Component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Component con sử dụng context
function Header() {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <header style={{ 
      background: theme === 'light' ? '#fff' : '#333',
      color: theme === 'light' ? '#000' : '#fff'
    }}>
      <h1>Website của tôi</h1>
      <button onClick={toggleTheme}>
        Chuyển sang {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}

// 4. App component
function App() {
  return (
    <ThemeProvider>
      <Header />
    </ThemeProvider>
  );
}
```

### Ví dụ với Authentication

```javascript
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (username, password) => {
    // Logic đăng nhập
    setUser({ username, id: 1 });
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

function UserMenu() {
  const { user, logout } = useContext(AuthContext);

  if (!user) {
    return <button>Đăng nhập</button>;
  }

  return (
    <div>
      <span>Xin chào, {user.username}</span>
      <button onClick={logout}>Đăng xuất</button>
    </div>
  );
}
```

---

## useRef

### Khái niệm

#### useRef khác useState như thế nào?

Để hiểu `useRef`, Cần hiểu sự khác biệt cơ bản giữa nó và `useState`:

```javascript
// useState: Khi thay đổi → Component RE-RENDER
const [count, setCount] = useState(0);
setCount(1); // → Component render lại

// useRef: Khi thay đổi → Component KHÔNG RE-RENDER
const countRef = useRef(0);
countRef.current = 1; // → Component KHÔNG render lại
```

**Bảng so sánh**:

| useState | useRef |
|----------|--------|
| Thay đổi → Re-render | Thay đổi → KHÔNG re-render |
| Dùng cho UI data (hiển thị lên màn hình) | Dùng cho non-UI data (background data) |
| Immutable (phải dùng setState) | Mutable (thay đổi trực tiếp .current) |
| Reset về initial value mỗi render | Giữ giá trị giữa các render |
| Async update | Sync update (thay đổi ngay lập tức) |

#### Tại sao cần useRef?

**Vấn đề 1: Cần lưu giá trị KHÔNG trigger re-render**

```javascript
// ❌ SAI: Dùng useState cho ID của interval
function Timer() {
  const [count, setCount] = useState(0);
  const [intervalId, setIntervalId] = useState(null);
  
  const start = () => {
    const id = setInterval(() => setCount(c => c + 1), 1000);
    setIntervalId(id); // ← Gây re-render không cần thiết!
  };
  
  const stop = () => {
    clearInterval(intervalId);
  };
  
  // Mỗi lần count tăng → setCount → render
  // → intervalId không thay đổi nhưng component vẫn render lại do setIntervalId ban đầu
}
```

**Vấn đề**: `intervalId` chỉ là "metadata" (dữ liệu phụ), không cần hiển thị lên UI. Nhưng vì dùng `useState`, mỗi lần set lại gây re-render không cần thiết.

```javascript
// ✅ ĐÚNG: Dùng useRef cho ID của interval
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null); // Không gây re-render khi thay đổi
  
  const start = () => {
    intervalRef.current = setInterval(() => setCount(c => c + 1), 1000);
    // Không gây re-render!
  };
  
  const stop = () => {
    clearInterval(intervalRef.current);
  };
}
```

**Vấn đề 2: Cần truy cập DOM element trực tiếp**

React là declarative (khai báo) - bạn mô tả UI BẠN MUỐN, React lo phần còn lại. Nhưng đôi khi bạn cần imperative (mệnh lệnh) - truy cập và điều khiển DOM trực tiếp:

```javascript
// Tôi MUỐN: Input này tự động focus khi component hiển thị
function SearchBox() {
  const inputRef = useRef(null);
  
  useEffect(() => {
    // Truy cập DOM element và focus
    inputRef.current.focus();
  }, []);
  
  return <input ref={inputRef} />;
}
```

Không có `useRef`, bạn không thể truy cập DOM element trong React.

**Vấn đề 3: Muốn lưu giá trị "trước đó" (previous value)**

```javascript
function Component({ value }) {
  // Làm sao lưu giá trị value của lần render trước?
  
  // ❌ SAI: Biến thông thường bị reset mỗi render
  let prevValue = value; // Luôn bằng value hiện tại
  
  // ❌ SAI: useState sẽ gây re-render và logic phức tạp
  const [prevValue, setPrevValue] = useState(value);
  
  // ✅ ĐÚNG: useRef lưu giá trị giữa các render
  const prevValueRef = useRef();
  
  useEffect(() => {
    prevValueRef.current = value; // Cập nhật sau mỗi render
  }, [value]);
  
  const prevValue = prevValueRef.current;
  
  return (
    <div>
      <p>Hiện tại: {value}</p>
      <p>Trước đó: {prevValue}</p>
    </div>
  );
}
```

#### useRef hoạt động như thế nào?

`useRef` trả về một object với 1 property duy nhất là `.current`:

```javascript
const myRef = useRef(initialValue);
// myRef = { current: initialValue }

// Đọc giá trị
console.log(myRef.current);

// Ghi giá trị (mutable - thay đổi trực tiếp)
myRef.current = newValue;
```

**Đặc điểm quan trọng**:
1. **Persistent (Bền vững)**: Object ref giữ nguyên giữa các lần render
2. **Mutable (Có thể thay đổi)**: Có thể gán `myRef.current = ...` trực tiếp
3. **No re-render**: Thay đổi `.current` KHÔNG trigger re-render
4. **Synchronous**: Thay đổi có hiệu lực ngay lập tức (không như setState)

#### Hai use case chính của useRef

**1. Truy cập DOM element**

```javascript
function FileUpload() {
  const fileInputRef = useRef(null);
  
  const handleClick = () => {
    // Trigger file browser dialog
    fileInputRef.current.click();
  };
  
  const handleReset = () => {
    // Reset file input
    fileInputRef.current.value = '';
  };
  
  return (
    <div>
      <input
        ref={fileInputRef}
        type="file"
        style={{ display: 'none' }}
      />
      <button onClick={handleClick}>Chọn file</button>
      <button onClick={handleReset}>Reset</button>
    </div>
  );
}
```

**Các thao tác DOM phổ biến**:
- `inputRef.current.focus()` - Focus vào input
- `inputRef.current.blur()` - Bỏ focus
- `videoRef.current.play()` - Play video
- `canvasRef.current.getContext('2d')` - Lấy canvas context
- `divRef.current.scrollIntoView()` - Scroll đến element

**2. Lưu mutable value (giá trị có thể thay đổi)**

```javascript
function Stopwatch() {
  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const intervalRef = useRef(null);
  const startTimeRef = useRef(null);
  
  const start = () => {
    setIsRunning(true);
    startTimeRef.current = Date.now() - time;
    
    intervalRef.current = setInterval(() => {
      setTime(Date.now() - startTimeRef.current);
    }, 10);
  };
  
  const stop = () => {
    setIsRunning(false);
    clearInterval(intervalRef.current);
  };
  
  const reset = () => {
    setTime(0);
    setIsRunning(false);
    clearInterval(intervalRef.current);
  };
  
  // Cleanup khi unmount
  useEffect(() => {
    return () => clearInterval(intervalRef.current);
  }, []);
  
  return (
    <div>
      <p>Time: {(time / 1000).toFixed(2)}s</p>
      {!isRunning ? (
        <button onClick={start}>Start</button>
      ) : (
        <button onClick={stop}>Stop</button>
      )}
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

**Các giá trị thường lưu trong ref**:
- Timer IDs (setTimeout, setInterval, requestAnimationFrame)
- Previous values (giá trị của lần render trước)
- DOM references
- Any value that changes but shouldn't trigger re-render

#### useRef vs Variable thông thường

```javascript
function Component() {
  let normalVar = 0; // ❌ Reset về 0 mỗi lần render
  const refVar = useRef(0); // ✅ Giữ giá trị qua các render
  
  const increment = () => {
    normalVar += 1;
    refVar.current += 1;
    
    console.log(normalVar); // Luôn là 1
    console.log(refVar.current); // Tăng: 1, 2, 3, 4...
  };
  
  return <button onClick={increment}>Increment</button>;
}
```

#### Anti-patterns và lỗi thường gặp

**❌ Đừng dùng ref.current trong JSX để render**
```javascript
// ❌ SAI
function Counter() {
  const countRef = useRef(0);
  
  return (
    <div>
      <p>{countRef.current}</p> {/* Không update khi click! */}
      <button onClick={() => countRef.current += 1}>
        Tăng
      </button>
    </div>
  );
}

// ✅ ĐÚNG: Dùng useState cho UI data
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>
        Tăng
      </button>
    </div>
  );
}
```

**❌ Đừng đọc/ghi ref.current trong render**
```javascript
// ❌ SAI
function Component() {
  const ref = useRef(0);
  ref.current += 1; // Ghi trong render → BUG!
  
  return <div>{ref.current}</div>;
}

// ✅ ĐÚNG: Ghi trong event handler hoặc useEffect
function Component() {
  const ref = useRef(0);
  
  const handleClick = () => {
    ref.current += 1; // OK: Ghi trong event handler
  };
  
  useEffect(() => {
    ref.current = someValue; // OK: Ghi trong effect
  });
}
```

### Cú pháp
```javascript
const refContainer = useRef(initialValue);
```

### Ví dụ 1: Truy cập DOM element

```javascript
import React, { useRef, useEffect } from 'react';

function TextInputWithFocusButton() {
  const inputRef = useRef(null);

  const handleFocus = () => {
    inputRef.current.focus();
  };

  useEffect(() => {
    // Auto focus khi component mount
    inputRef.current.focus();
  }, []);

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={handleFocus}>Focus vào input</button>
    </div>
  );
}
```

### Ví dụ 2: Lưu giá trị không trigger re-render

```javascript
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);

  const startTimer = () => {
    if (intervalRef.current) return; // Đã chạy rồi
    
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };

  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  useEffect(() => {
    return () => stopTimer(); // Cleanup
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={startTimer}>Bắt đầu</button>
      <button onClick={stopTimer}>Dừng</button>
    </div>
  );
}
```

### Ví dụ 3: Lưu giá trị trước đó

```javascript
function UsePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = UsePrevious(count);

  return (
    <div>
      <p>Hiện tại: {count}</p>
      <p>Trước đó: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Tăng</button>
    </div>
  );
}
```

---

## useMemo

### Khái niệm

#### Memoization là gì?

**Memoization** là kỹ thuật lưu trữ (cache) kết quả của một phép tính đắt đỏ, để không phải tính lại khi input giống nhau.

Ví dụ trong đời thực:
- Bạn tính `2 + 2 = 4` và ghi nhớ kết quả
- Lần sau gặp `2 + 2`, bạn KHÔNG tính lại, chỉ lấy kết quả đã ghi nhớ: `4`
- Nếu gặp `3 + 3`, bạn mới tính vì input khác

```javascript
// Không có memoization
function expensiveCalculation(a, b) {
  console.log('Đang tính toán...');
  let result = 0;
  for (let i = 0; i < 1000000000; i++) {
    result += a + b;
  }
  return result;
}

// Mỗi lần gọi đều phải tính lại, mất thời gian
expensiveCalculation(2, 3); // Tính... → 5
expensiveCalculation(2, 3); // Tính lại... → 5 (lãng phí!)

// Có memoization
const memoizedCalc = memoize(expensiveCalculation);
memoizedCalc(2, 3); // Tính... → 5
memoizedCalc(2, 3); // Lấy từ cache → 5 (nhanh!)
```

#### Vấn đề: Component re-render không cần thiết

Trong React, mỗi khi component re-render, TẤT CẢ code trong function component đều chạy lại:

```javascript
function Component() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([1, 2, 3, 4, 5]);
  
  // Phép tính phức tạp
  const sum = items.reduce((total, item) => total + item, 0);
  // ↑ Code này chạy lại MỖI LẦN component render!
  
  return (
    <div>
      <p>Sum: {sum}</p>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Tăng Count
      </button>
    </div>
  );
}
```

**Vấn đề**: Khi `count` thay đổi → component render lại → `sum` được tính lại, mặc dù `items` KHÔNG thay đổi!

Nếu phép tính `sum` rất phức tạp (ví dụ: filter/map một array 10,000 phần tử), mỗi lần render sẽ chậm, gây lag UI.

#### useMemo giải quyết như thế nào?

`useMemo` "ghi nhớ" kết quả và chỉ tính lại khi dependencies thay đổi:

```javascript
function Component() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([1, 2, 3, 4, 5]);
  
  // useMemo: Chỉ tính lại khi items thay đổi
  const sum = useMemo(() => {
    console.log('Tính sum...');
    return items.reduce((total, item) => total + item, 0);
  }, [items]); // Dependencies: [items]
  
  return (
    <div>
      <p>Sum: {sum}</p>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Tăng Count {/* sum KHÔNG được tính lại */}
      </button>
      <button onClick={() => setItems([...items, items.length + 1])}>
        Thêm Item {/* sum MỚI được tính lại */}
      </button>
    </div>
  );
}
```

**Flow hoạt động**:
1. Render lần 1: `items = [1,2,3,4,5]` → Tính sum → Lưu kết quả `15` và `items`
2. Click "Tăng Count" → `count` thay đổi → Render lại
3. useMemo kiểm tra: `items` có thay đổi không? KHÔNG
4. → Trả về kết quả đã lưu: `15` (không tính lại)
5. Click "Thêm Item" → `items` thay đổi → Render lại
6. useMemo kiểm tra: `items` có thay đổi không? CÓ
7. → Tính lại sum với `items` mới

#### Cơ chế hoạt động chi tiết

```javascript
const memoizedValue = useMemo(() => {
  // Function này chỉ chạy khi dependencies thay đổi
  return expensiveComputation(a, b);
}, [a, b]); // Dependencies
```

React lưu trữ:
1. **Giá trị trả về** của function
2. **Dependencies** để so sánh

Mỗi lần render:
1. So sánh dependencies mới với dependencies cũ
2. Nếu GIỐNG NHAU → Trả về giá trị đã lưu (không chạy function)
3. Nếu KHÁC NHAU → Chạy function, lưu giá trị mới

```javascript
// Ví dụ minh họa
useMemo(() => computeValue(a, b), [a, b]);

// Render 1: a=1, b=2
// → Chạy computeValue(1, 2) → Lưu result = 3 và [a=1, b=2]

// Render 2: a=1, b=2 (không đổi)
// → So sánh [1, 2] === [1, 2] ? YES
// → Trả về 3 (không gọi computeValue)

// Render 3: a=1, b=3 (b thay đổi)
// → So sánh [1, 3] === [1, 2] ? NO
// → Chạy computeValue(1, 3) → Lưu result mới
```

#### Khi nào NÊN dùng useMemo?

**1. Phép tính phức tạp, tốn thời gian**

```javascript
// ✅ NÊN: Filter/map array lớn
const filteredItems = useMemo(() => {
  return hugeArray.filter(item => 
    item.name.toLowerCase().includes(searchTerm.toLowerCase())
  );
}, [hugeArray, searchTerm]);

// ✅ NÊN: Tính toán toán học phức tạp
const result = useMemo(() => {
  return complexMathOperation(data);
}, [data]);
```

**2. Tránh tạo object/array mới (referential equality)**

```javascript
function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ Không dùng useMemo: options là object MỚI mỗi lần render
  const options = { color: 'red', size: 'large' };
  
  return <Child options={options} />;
  // Mỗi lần Parent render, Child nhận options "mới" (reference khác)
  // → Child re-render ngay cả khi options giống nhau!
}

function Parent() {
  const [count, setCount] = useState(0);
  
  // ✅ Dùng useMemo: options GIỮ NGUYÊN reference
  const options = useMemo(() => ({
    color: 'red',
    size: 'large'
  }), []); // Không có dependencies → object tạo 1 lần
  
  return <Child options={options} />;
  // Child chỉ re-render khi options thực sự thay đổi
}
```

**Tại sao?** JavaScript so sánh object/array bằng reference:

```javascript
const obj1 = { a: 1 };
const obj2 = { a: 1 };
obj1 === obj2; // false (reference khác nhau!)

const obj3 = obj1;
obj1 === obj3; // true (cùng reference)
```

**3. Dùng với React.memo để tối ưu child component**

```javascript
const ExpensiveChild = React.memo(({ data }) => {
  console.log('ExpensiveChild render');
  return <div>{/* Render phức tạp */}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([1, 2, 3]);
  
  // ❌ Không useMemo: processedItems là array mới mỗi render
  const processedItems = items.map(i => i * 2);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ExpensiveChild data={processedItems} />
      {/* Mỗi lần count thay đổi, ExpensiveChild re-render vì data "mới" */}
    </div>
  );
}

function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([1, 2, 3]);
  
  // ✅ Dùng useMemo: processedItems chỉ tạo mới khi items thay đổi
  const processedItems = useMemo(() => {
    return items.map(i => i * 2);
  }, [items]);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ExpensiveChild data={processedItems} />
      {/* ExpensiveChild KHÔNG re-render khi count thay đổi */}
    </div>
  );
}
```

#### Khi nào KHÔNG NÊN dùng useMemo?

**1. Phép tính đơn giản, nhanh**

```javascript
// ❌ KHÔNG CẦN useMemo cho phép tính đơn giản
const sum = useMemo(() => a + b, [a, b]);
const isEven = useMemo(() => count % 2 === 0, [count]);

// ✅ Tính trực tiếp nhanh hơn
const sum = a + b;
const isEven = count % 2 === 0;
```

useMemo có overhead (chi phí) - React phải so sánh dependencies. Với phép tính đơn giản, overhead này LỚN HƠN lợi ích!

**2. Primitive values (string, number, boolean)**

```javascript
// ❌ VÔ NGHĨA: Primitive values không có vấn đề referential equality
const greeting = useMemo(() => `Hello ${name}`, [name]);

// ✅ Không cần useMemo
const greeting = `Hello ${name}`;
```

**3. Dependencies thay đổi liên tục**

```javascript
// ❌ VÔ DỤNG: timestamp thay đổi mỗi render → useMemo luôn tính lại
const result = useMemo(() => {
  return complexCalc(timestamp);
}, [timestamp]);

// Không khác gì không dùng useMemo!
```

#### Premature Optimization (Tối ưu sớm)

> "Premature optimization is the root of all evil" - Donald Knuth

**Quy tắc vàng**:
1. **Viết code đơn giản trước**
2. **Đo performance** (dùng React DevTools Profiler)
3. **Chỉ tối ưu nếu có vấn đề performance thực sự**

```javascript
// ❌ Quá nhiều useMemo không cần thiết
function Component({ a, b, c }) {
  const sum = useMemo(() => a + b, [a, b]);
  const product = useMemo(() => a * b, [a, b]);
  const greeting = useMemo(() => `Hello ${c}`, [c]);
  
  // Code phức tạp, khó đọc, performance không cải thiện
}

// ✅ Đơn giản, dễ đọc
function Component({ a, b, c }) {
  const sum = a + b;
  const product = a * b;
  const greeting = `Hello ${c}`;
  
  // Nếu SAU NÀY có vấn đề performance, mới thêm useMemo
}
```

#### useMemo vs useEffect

Nhiều người nhầm lẫn giữa `useMemo` và `useEffect`. Hãy phân biệt:

```javascript
// useMemo: Tính giá trị TRONG quá trình render
const value = useMemo(() => {
  return computeValue(deps); // Trả về giá trị
}, [deps]);

// useEffect: Chạy side effect SAU khi render
useEffect(() => {
  doSomething(deps); // Không trả về gì (hoặc cleanup function)
}, [deps]);
```

**useMemo**: "Tôi cần giá trị này để render"
**useEffect**: "Sau khi render xong, làm việc này"

### Cú pháp
```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

### Ví dụ 1: Tính toán phức tạp

```javascript
import React, { useState, useMemo } from 'react';

function ExpensiveCalculation() {
  const [numbers, setNumbers] = useState([1, 2, 3, 4, 5]);
  const [count, setCount] = useState(0);

  // Hàm tính toán "đắt đỏ"
  const sum = useMemo(() => {
    console.log('Đang tính tổng...');
    return numbers.reduce((total, num) => total + num, 0);
  }, [numbers]); // Chỉ tính lại khi numbers thay đổi

  // Khi count thay đổi, sum KHÔNG được tính lại
  return (
    <div>
      <p>Tổng: {sum}</p>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Tăng Count
      </button>
      <button onClick={() => setNumbers([...numbers, numbers.length + 1])}>
        Thêm số
      </button>
    </div>
  );
}
```

### Ví dụ 2: Filter danh sách lớn

```javascript
function SearchList() {
  const [searchTerm, setSearchTerm] = useState('');
  const [items, setItems] = useState([
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Orange' },
    // ... hàng nghìn items
  ]);

  const filteredItems = useMemo(() => {
    console.log('Đang filter...');
    return items.filter(item =>
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [items, searchTerm]);

  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Tìm kiếm..."
      />
      <ul>
        {filteredItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Khi nào nên dùng useMemo?
- Phép tính phức tạp, tốn nhiều thời gian
- Render danh sách lớn
- Tránh re-render không cần thiết cho child component

### Khi nào KHÔNG nên dùng?
- Phép tính đơn giản, nhanh
- Premature optimization (tối ưu quá sớm)

---

## useCallback

### Khái niệm

#### useCallback là useMemo cho functions

`useCallback` về bản chất giống `useMemo`, nhưng dành riêng cho **functions**:

```javascript
// useMemo: Memoize GIÁ TRỊ
const memoizedValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);

// useCallback: Memoize FUNCTION
const memoizedFunction = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

// Thực ra useCallback chỉ là shorthand:
const memoizedFunction = useMemo(() => {
  return () => doSomething(a, b);
}, [a, b]);
```

#### Vấn đề: Functions tạo mới mỗi lần render

Trong JavaScript, mỗi function là một object, và mỗi lần tạo function là một reference mới:

```javascript
function Component() {
  // Mỗi lần render, function này được TẠO MỚI
  const handleClick = () => {
    console.log('Clicked');
  };
  
  // Mỗi lần render, handleClick có reference KHÁC NHAU
  // handleClick render 1 !== handleClick render 2
}
```

**Minh họa**:
```javascript
const func1 = () => console.log('hello');
const func2 = () => console.log('hello');
func1 === func2; // false! Khác reference

const func3 = func1;
func1 === func3; // true! Cùng reference
```

**Tại sao đây là vấn đề?**

**Vấn đề 1: Child component re-render không cần thiết**

```javascript
// Child component được bọc React.memo để tối ưu
const Button = React.memo(({ onClick, children }) => {
  console.log(`Render button: ${children}`);
  return <button onClick={onClick}>{children}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  // ❌ Function mới mỗi render
  const handleClick = () => {
    setCount(count + 1);
  };
  
  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      <Button onClick={handleClick}>Count: {count}</Button>
      {/* 
        Mỗi lần gõ vào input:
        1. text thay đổi → Parent render lại
        2. handleClick được TẠO MỚI (reference mới)
        3. Button nhận prop onClick với reference mới
        4. React.memo so sánh: onClick cũ !== onClick mới
        5. → Button re-render (mặc dù logic giống hệt!)
      */}
    </div>
  );
}
```

**Giải thích chi tiết**:
- `React.memo` kiểm tra props bằng **reference equality** (`===`)
- Function mới = reference mới → React.memo nghĩ props thay đổi → re-render
- Ngay cả khi logic của function y hệt, nhưng reference khác → re-render

**Vấn đề 2: useEffect chạy lại không mong muốn**

```javascript
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  // ❌ Function mới mỗi render
  const fetchResults = async () => {
    const res = await fetch(`/api/search?q=${query}`);
    const data = await res.json();
    setResults(data);
  };
  
  useEffect(() => {
    fetchResults();
  }, [fetchResults]); // fetchResults là dependency
  
  /*
    Flow vấn đề:
    1. User gõ query → setQuery → Component render
    2. fetchResults được TẠO MỚI
    3. useEffect thấy fetchResults thay đổi (reference mới)
    4. → Chạy fetchResults() → fetch API
    5. setResults → Component render lại
    6. fetchResults lại được TẠO MỚI
    7. useEffect lại chạy → fetch lại
    8. Vòng lặp vô tận! 💥
  */
}
```

#### useCallback giải quyết như thế nào?

`useCallback` "ghi nhớ" function và chỉ tạo mới khi dependencies thay đổi:

```javascript
// ✅ Giải quyết vấn đề 1: Child component re-render
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  // Function chỉ tạo MỘT LẦN (dependencies [])
  const handleClick = useCallback(() => {
    setCount(c => c + 1); // Dùng functional update
  }, []); // Không phụ thuộc vào count
  
  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      <Button onClick={handleClick}>Count: {count}</Button>
      {/*
        Khi gõ input:
        1. text thay đổi → Parent render
        2. handleClick GIỮ NGUYÊN reference (useCallback)
        3. Button nhận prop onClick với reference KHÔNG ĐỔI
        4. React.memo: onClick cũ === onClick mới
        5. → Button KHÔNG re-render ✅
      */}
    </div>
  );
}

// ✅ Giải quyết vấn đề 2: useEffect ổn định
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  // Function chỉ tạo lại khi query thay đổi
  const fetchResults = useCallback(async () => {
    const res = await fetch(`/api/search?q=${query}`);
    const data = await res.json();
    setResults(data);
  }, [query]); // Dependencies: [query]
  
  useEffect(() => {
    fetchResults();
  }, [fetchResults]);
  
  /*
    Flow chính xác:
    1. User gõ query → setQuery → query thay đổi
    2. fetchResults được tạo LẠI (vì query thay đổi)
    3. useEffect thấy fetchResults thay đổi
    4. → Chạy fetchResults() → fetch API với query MỚI ✅
    5. setResults → Component render
    6. fetchResults GIỮ NGUYÊN (query không đổi)
    7. useEffect KHÔNG chạy lại (fetchResults không đổi)
    8. Hoàn hảo! ✅
  */
}
```

#### Cơ chế hoạt động chi tiết

```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b] // Dependencies
);
```

React lưu trữ:
1. **Function** (function instance)
2. **Dependencies** để so sánh

Mỗi lần render:
1. So sánh dependencies mới với dependencies cũ
2. Nếu GIỐNG → Trả về function đã lưu (reference cũ)
3. Nếu KHÁC → Tạo function mới, lưu lại

```javascript
// Ví dụ cụ thể
const handleClick = useCallback(() => {
  console.log(count);
}, [count]);

// Render 1: count = 0
// → Tạo function A: () => console.log(0)
// → Lưu function A và [count=0]
// → Trả về function A

// Render 2: count = 0 (không đổi)
// → So sánh [0] === [0] ? YES
// → Trả về function A (cùng reference!)

// Render 3: count = 1 (thay đổi)
// → So sánh [1] === [0] ? NO
// → Tạo function B: () => console.log(1)
// → Lưu function B và [count=1]
// → Trả về function B (reference mới)
```

#### Functional Update Pattern

Một pattern quan trọng khi dùng `useCallback`:

```javascript
// ❌ Phụ thuộc vào count → phải thêm count vào dependencies
const increment = useCallback(() => {
  setCount(count + 1);
}, [count]); // Phải có count → mỗi lần count đổi, function tạo mới

// ✅ Dùng functional update → KHÔNG phụ thuộc count
const increment = useCallback(() => {
  setCount(c => c + 1); // c là giá trị hiện tại
}, []); // Không dependencies → function tạo 1 lần duy nhất!
```

**Tại sao functional update tốt hơn?**
- Function chỉ tạo **một lần duy nhất**
- Reference **không bao giờ thay đổi**
- Child component **không bao giờ re-render** vì callback
- Code **đơn giản hơn** (không cần track dependencies)

#### Khi nào NÊN dùng useCallback?

**1. Truyền callback xuống child component được bọc React.memo**

```javascript
const ChildComponent = React.memo(({ onAction }) => {
  // Expensive render
});

function Parent() {
  // ✅ NÊN: Dùng useCallback để tránh re-render child
  const handleAction = useCallback(() => {
    doSomething();
  }, []);
  
  return <ChildComponent onAction={handleAction} />;
}
```

**2. Callback là dependency của useEffect/useMemo/useCallback khác**

```javascript
function Component() {
  // ✅ NÊN: Dùng useCallback khi callback là dependency
  const fetchData = useCallback(async () => {
    const data = await fetch('/api');
    return data.json();
  }, []);
  
  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // fetchData là dependency
}
```

**3. Callback được truyền vào custom hook với dependencies**

```javascript
function useCustomHook(callback) {
  useEffect(() => {
    callback();
  }, [callback]); // callback là dependency
}

function Component() {
  // ✅ NÊN: Dùng useCallback khi truyền vào hook
  const handleSomething = useCallback(() => {
    doSomething();
  }, []);
  
  useCustomHook(handleSomething);
}
```

#### Khi nào KHÔNG NÊN dùng useCallback?

**1. Callback chỉ dùng trong component, không truyền đi**

```javascript
function Component() {
  // ❌ KHÔNG CẦN: Callback chỉ dùng nội bộ
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  
  return <button onClick={handleClick}>Click</button>;
  
  // ✅ Đơn giản hơn
  return <button onClick={() => console.log('clicked')}>Click</button>;
}
```

**2. Child component KHÔNG được memo**

```javascript
function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
}

function Parent() {
  // ❌ KHÔNG CẦN: Child không memo, sẽ re-render anyway
  const handleClick = useCallback(() => {
    doSomething();
  }, []);
  
  return <Child onClick={handleClick} />;
}
```

**3. Dependencies thay đổi liên tục**

```javascript
function Component({ dynamicValue }) {
  // ❌ VÔ DỤNG: dynamicValue đổi mỗi render
  // → useCallback luôn tạo function mới
  const handleClick = useCallback(() => {
    doSomething(dynamicValue);
  }, [dynamicValue]);
  
  // Không khác gì không dùng useCallback
}
```

#### useCallback vs useMemo

```javascript
// useCallback: Trả về FUNCTION
const memoizedCallback = useCallback(
  () => doSomething(a, b),
  [a, b]
);

// useMemo: Trả về VALUE
const memoizedValue = useMemo(
  () => computeValue(a, b),
  [a, b]
);

// useCallback tương đương với:
const memoizedCallback = useMemo(
  () => () => doSomething(a, b), // Trả về function
  [a, b]
);
```

**Khi nào dùng cái nào?**
- **useCallback**: Khi bạn cần memoize **function** (callback)
- **useMemo**: Khi bạn cần memoize **giá trị** (kết quả tính toán)

#### Anti-patterns

**❌ Dùng useCallback cho mọi function**
```javascript
// Quá tải useCallback - code phức tạp không cần thiết
function Component() {
  const handleClick = useCallback(() => {}, []);
  const handleChange = useCallback(() => {}, []);
  const handleSubmit = useCallback(() => {}, []);
  const handleReset = useCallback(() => {}, []);
  // ... 20 callbacks khác
}
```

**✅ Chỉ dùng khi thực sự cần**
```javascript
function Component() {
  // Chỉ callback được truyền xuống child memo mới cần useCallback
  const handleImportantAction = useCallback(() => {}, []);
  
  // Các callback khác không cần
  const handleClick = () => {};
  const handleChange = () => {};
}
```

### Cú pháp
```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b]
);
```

### Ví dụ 1: Tránh re-render child component

```javascript
import React, { useState, useCallback, memo } from 'react';

// Child component được memo
const Button = memo(({ onClick, children }) => {
  console.log(`Rendering button: ${children}`);
  return <button onClick={onClick}>{children}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [otherState, setOtherState] = useState(0);

  // Không dùng useCallback - Button sẽ re-render mỗi lần Parent render
  // const increment = () => setCount(count + 1);

  // Dùng useCallback - Button chỉ re-render khi cần thiết
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []); // Không có dependencies vì dùng functional update

  return (
    <div>
      <p>Count: {count}</p>
      <p>Other: {otherState}</p>
      <Button onClick={increment}>Tăng Count</Button>
      <button onClick={() => setOtherState(otherState + 1)}>
        Tăng Other
      </button>
    </div>
  );
}
```

### Ví dụ 2: Truyền callback vào useEffect

```javascript
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const fetchResults = useCallback(async () => {
    if (!query) return;
    
    const response = await fetch(`/api/search?q=${query}`);
    const data = await response.json();
    setResults(data);
  }, [query]);

  useEffect(() => {
    fetchResults();
  }, [fetchResults]); // fetchResults được memoized

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <ul>
        {results.map(item => <li key={item.id}>{item.name}</li>)}
      </ul>
    </div>
  );
}
```

### So sánh useMemo vs useCallback

```javascript
// useCallback trả về function
const memoizedCallback = useCallback(() => {
  return a + b;
}, [a, b]);

// useMemo trả về giá trị
const memoizedValue = useMemo(() => {
  return a + b;
}, [a, b]);

// Thực tế useCallback tương đương:
const memoizedCallback = useMemo(() => {
  return () => {
    return a + b;
  };
}, [a, b]);
```

---

## useReducer

### Khái niệm

#### State Management: useState vs useReducer

Hãy bắt đầu với câu hỏi: **Khi nào state trở nên "phức tạp"?**

**useState - Phù hợp cho state đơn giản**:
```javascript
const [count, setCount] = useState(0);
const [name, setName] = useState('');
const [isOpen, setIsOpen] = useState(false);
```

**Vấn đề xuất hiện khi state phức tạp hơn**:

```javascript
// ❌ Nhiều state liên quan nhau
function Form() {
  const [username, setUsername] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitCount, setSubmitCount] = useState(0);
  
  // Logic phức tạp, phải update nhiều state cùng lúc
  const handleSubmit = () => {
    setIsSubmitting(true);
    setErrors({});
    setSubmitCount(c => c + 1);
    
    // Validation logic...
    if (!username) {
      setErrors(prev => ({ ...prev, username: 'Required' }));
      setIsSubmitting(false);
      return;
    }
    // ... nhiều logic khác
  };
}
```

**Vấn đề**:
1. **Nhiều state liên quan**: username, email, password, errors đều là phần của "form state"
2. **Logic update rải rác**: Phải gọi nhiều setState ở nhiều chỗ
3. **Khó maintain**: Sửa logic phải tìm tất cả chỗ setState
4. **Race condition**: Nhiều setState async có thể gây bugs
5. **Khó test**: Phải mock nhiều state và setState

#### useReducer - Quản lý state phức tạp

`useReducer` lấy ý tưởng từ pattern **Redux**: tập trung tất cả logic update state vào **MỘT NƠI DUY NHẤT** (reducer function).

**Triết lý**:
- **State**: "Dữ liệu là GÌ" (What)
- **Action**: "Muốn làm GÌ" (What to do)
- **Reducer**: "Làm NHƯ THẾ NÀO" (How to do)

```javascript
// useState: Bạn nói "LÀM NHƯ THẾ NÀO"
setCount(count + 1);           // "Tăng count lên 1"
setCount(0);                   // "Set count về 0"

// useReducer: Bạn nói "MUỐN LÀM GÌ", reducer lo "LÀM NHƯ THẾ NÀO"
dispatch({ type: 'INCREMENT' }); // "Tôi muốn tăng count"
dispatch({ type: 'RESET' });     // "Tôi muốn reset count"
// Reducer sẽ quyết định cách tăng, cách reset
```

#### Các thành phần của useReducer

**1. State**: Dữ liệu hiện tại
```javascript
const state = {
  count: 0,
  history: [],
  isLoading: false
};
```

**2. Action**: Object mô tả "hành động muốn làm"
```javascript
// Action có type (bắt buộc) và payload (optional)
{ type: 'INCREMENT' }
{ type: 'SET_COUNT', payload: 10 }
{ type: 'ADD_TODO', payload: { id: 1, text: 'Learn React' } }
```

**3. Reducer**: Function quyết định state mới dựa trên state cũ và action
```javascript
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'SET_COUNT':
      return { ...state, count: action.payload };
    default:
      return state; // Không thay đổi state
  }
}
```

**4. Dispatch**: Function để "gửi" action đến reducer
```javascript
dispatch({ type: 'INCREMENT' });
```

#### Flow hoạt động chi tiết

```javascript
const [state, dispatch] = useReducer(reducer, initialState);

// 1. User click button
dispatch({ type: 'INCREMENT' });

// 2. React gọi reducer với state hiện tại và action
reducer(currentState, { type: 'INCREMENT' });

// 3. Reducer trả về state MỚI
return { ...state, count: state.count + 1 };

// 4. React cập nhật state và re-render component
// Component nhận state mới
```

**Minh họa cụ thể**:
```javascript
function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  
  console.log('State hiện tại:', state); // { count: 0 }
  
  // User click "Increment"
  dispatch({ type: 'INCREMENT' });
  
  // React gọi: reducer({ count: 0 }, { type: 'INCREMENT' })
  // Reducer trả về: { count: 1 }
  // React render lại với state = { count: 1 }
}
```

#### Tại sao useReducer tốt hơn useState cho state phức tạp?

**1. Tập trung logic ở một chỗ**

```javascript
// ❌ useState: Logic rải rác khắp nơi
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');
  
  const addTodo = (text) => {
    setTodos([...todos, { id: Date.now(), text, completed: false }]);
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  const clearCompleted = () => {
    setTodos(todos.filter(todo => !todo.completed));
  };
  // ... 10 functions nữa
}

// ✅ useReducer: Tất cả logic trong reducer
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, { id: Date.now(), text: action.text, completed: false }];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.id ? { ...todo, completed: !todo.completed } : todo
      );
    case 'DELETE_TODO':
      return state.filter(todo => todo.id !== action.id);
    case 'CLEAR_COMPLETED':
      return state.filter(todo => !todo.completed);
    default:
      return state;
  }
}

function TodoApp() {
  const [todos, dispatch] = useReducer(todoReducer, []);
  
  // Component chỉ dispatch actions, không quan tâm logic
  const addTodo = (text) => dispatch({ type: 'ADD_TODO', text });
  const toggleTodo = (id) => dispatch({ type: 'TOGGLE_TODO', id });
  // Đơn giản, rõ ràng!
}
```

**2. Dễ test**

```javascript
// Test reducer - PURE FUNCTION, dễ test
test('INCREMENT tăng count lên 1', () => {
  const state = { count: 0 };
  const action = { type: 'INCREMENT' };
  const newState = reducer(state, action);
  
  expect(newState.count).toBe(1);
});

// Không cần mock React, không cần render component
// Chỉ cần gọi reducer như function thông thường
```

**3. Dễ debug**

```javascript
function reducer(state, action) {
  console.log('Action:', action.type);
  console.log('State trước:', state);
  
  const newState = /* ... */;
  
  console.log('State sau:', newState);
  return newState;
}

// Bạn thấy CHÍNH XÁC:
// - Action nào được dispatch
// - State trước và sau mỗi action
// - Có thể log tất cả actions để replay
```

**4. State transitions rõ ràng**

```javascript
// useState: Không rõ state có thể thay đổi như thế nào
const [user, setUser] = useState(null);
// setUser có thể nhận BẤT KỲ giá trị nào ở BẤT KỲ đâu

// useReducer: Chỉ có thể thay đổi qua các actions định nghĩa sẵn
const [user, dispatch] = useReducer(userReducer, null);
// Chỉ có thể: LOGIN, LOGOUT, UPDATE_PROFILE
// Không thể có state transition không hợp lệ
```

**5. Multiple state updates thành một**

```javascript
// ❌ useState: Nhiều updates → nhiều re-renders
const handleSubmit = () => {
  setIsSubmitting(true);    // Re-render 1
  setErrors({});            // Re-render 2
  setSubmitCount(c => c+1); // Re-render 3
};

// ✅ useReducer: Một action → một re-render
const handleSubmit = () => {
  dispatch({ type: 'SUBMIT_START' });
  // Reducer update tất cả cùng lúc → chỉ 1 re-render
};
```

#### Khi nào nên dùng useReducer?

**✅ Nên dùng useReducer khi**:

1. **State là object/array phức tạp với nhiều sub-values**
```javascript
const state = {
  user: { name, email, avatar },
  settings: { theme, language, notifications },
  ui: { isLoading, errors, modal }
};
```

2. **State tiếp theo phụ thuộc vào state trước**
```javascript
// Tính total dựa trên items và discount
const newTotal = calculateTotal(state.items, state.discount);
```

3. **Nhiều actions khác nhau update state**
```javascript
// ADD, DELETE, UPDATE, SORT, FILTER, CLEAR...
```

4. **Logic update phức tạp**
```javascript
case 'ADD_ITEM':
  // Kiểm tra duplicate
  // Validate
  // Update nhiều fields
  // Recalculate totals
```

5. **Cần share logic giữa nhiều components**
```javascript
// Nhiều components dùng chung reducer
import { todoReducer } from './reducers';
```

**❌ Không nên dùng useReducer khi**:

1. **State đơn giản (primitive values)**
```javascript
const [count, setCount] = useState(0); // Đủ rồi
```

2. **Chỉ có 1-2 cách update state**
```javascript
const [isOpen, setIsOpen] = useState(false);
// Chỉ cần toggle → useState đơn giản hơn
```

3. **State độc lập, không liên quan**
```javascript
const [name, setName] = useState('');
const [age, setAge] = useState(0);
// Không liên quan → không cần reducer
```

#### Patterns và Best Practices

**1. Action Types - Dùng constants**

```javascript
// ❌ Magic strings - dễ typo
dispatch({ type: 'INCREMENT' });
dispatch({ type: 'INCREMNET' }); // Lỗi typo, khó phát hiện!

// ✅ Constants - TypeScript/IDE hỗ trợ
const INCREMENT = 'INCREMENT';
const DECREMENT = 'DECREMENT';

dispatch({ type: INCREMENT }); // Auto-complete, không typo
```

**2. Action Creators**

```javascript
// ❌ Dispatch trực tiếp - dễ nhầm payload structure
dispatch({ type: 'ADD_TODO', text: 'Learn', id: 1 });
dispatch({ type: 'ADD_TODO', text: 'React' }); // Thiếu id!

// ✅ Action creators - consistent payload
const addTodo = (text) => ({
  type: 'ADD_TODO',
  payload: { id: Date.now(), text, completed: false }
});

dispatch(addTodo('Learn React')); // Luôn đúng structure
```

**3. Immutable Updates**

```javascript
// ❌ SAI: Mutate state trực tiếp
function reducer(state, action) {
  state.count += 1; // MUTATE!
  return state;     // React không phát hiện thay đổi!
}

// ✅ ĐÚNG: Tạo object/array mới
function reducer(state, action) {
  return { ...state, count: state.count + 1 };
}
```

**4. Organize by Feature**

```javascript
// Reducer lớn, dễ loạn
function appReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO': // ...
    case 'TOGGLE_TODO': // ...
    case 'LOGIN': // ...
    case 'LOGOUT': // ...
    case 'SET_THEME': // ...
    // 50 cases khác...
  }
}

// ✅ Tách thành sub-reducers
function todoReducer(state, action) { /* ... */ }
function authReducer(state, action) { /* ... */ }
function uiReducer(state, action) { /* ... */ }

function appReducer(state, action) {
  return {
    todos: todoReducer(state.todos, action),
    auth: authReducer(state.auth, action),
    ui: uiReducer(state.ui, action)
  };
}
```

#### useReducer vs useState - Tổng kết

| Tiêu chí | useState | useReducer |
|----------|----------|------------|
| **Độ phức tạp state** | Đơn giản | Phức tạp |
| **Số lượng updates** | Ít | Nhiều |
| **Logic update** | Đơn giản | Phức tạp |
| **Testability** | Khó test | Dễ test |
| **Code organization** | Rải rác | Tập trung |
| **Learning curve** | Dễ | Khó hơn |
| **Boilerplate** | Ít | Nhiều hơn |

**Kết luận**: Bắt đầu với `useState`, chuyển sang `useReducer` khi state phức tạp!

### Cú pháp
```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

- `reducer`: function nhận (state, action) và trả về state mới
- `initialState`: giá trị khởi tạo
- `state`: state hiện tại
- `dispatch`: function để gửi action

### Ví dụ 1: Counter với useReducer

```javascript
import React, { useReducer } from 'react';

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    case 'SET':
      return { count: action.payload };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>
        +1
      </button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>
        -1
      </button>
      <button onClick={() => dispatch({ type: 'RESET' })}>
        Reset
      </button>
      <button onClick={() => dispatch({ type: 'SET', payload: 10 })}>
        Set to 10
      </button>
    </div>
  );
}
```

### Ví dụ 2: Form phức tạp

```javascript
const initialState = {
  username: '',
  email: '',
  password: '',
  confirmPassword: '',
  errors: {}
};

function formReducer(state, action) {
  switch (action.type) {
    case 'UPDATE_FIELD':
      return {
        ...state,
        [action.field]: action.value,
        errors: {
          ...state.errors,
          [action.field]: null // Clear error khi user nhập
        }
      };
    case 'SET_ERRORS':
      return {
        ...state,
        errors: action.errors
      };
    case 'RESET':
      return initialState;
    default:
      return state;
  }
}

function RegistrationForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  const handleChange = (e) => {
    dispatch({
      type: 'UPDATE_FIELD',
      field: e.target.name,
      value: e.target.value
    });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Validation
    const errors = {};
    if (!state.username) errors.username = 'Username là bắt buộc';
    if (!state.email) errors.email = 'Email là bắt buộc';
    if (state.password !== state.confirmPassword) {
      errors.confirmPassword = 'Mật khẩu không khớp';
    }

    if (Object.keys(errors).length > 0) {
      dispatch({ type: 'SET_ERRORS', errors });
      return;
    }

    // Submit form
    console.log('Form submitted:', state);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="username"
          value={state.username}
          onChange={handleChange}
          placeholder="Username"
        />
        {state.errors.username && <span>{state.errors.username}</span>}
      </div>
      
      <div>
        <input
          name="email"
          value={state.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {state.errors.email && <span>{state.errors.email}</span>}
      </div>
      
      <div>
        <input
          name="password"
          type="password"
          value={state.password}
          onChange={handleChange}
          placeholder="Password"
        />
      </div>
      
      <div>
        <input
          name="confirmPassword"
          type="password"
          value={state.confirmPassword}
          onChange={handleChange}
          placeholder="Confirm Password"
        />
        {state.errors.confirmPassword && <span>{state.errors.confirmPassword}</span>}
      </div>
      
      <button type="submit">Đăng ký</button>
      <button type="button" onClick={() => dispatch({ type: 'RESET' })}>
        Reset
      </button>
    </form>
  );
}
```

### Ví dụ 3: Todo List

```javascript
const todoReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, {
        id: Date.now(),
        text: action.text,
        completed: false
      }];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.id
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    case 'DELETE_TODO':
      return state.filter(todo => todo.id !== action.id);
    case 'EDIT_TODO':
      return state.map(todo =>
        todo.id === action.id
          ? { ...todo, text: action.text }
          : todo
      );
    default:
      return state;
  }
};

function TodoList() {
  const [todos, dispatch] = useReducer(todoReducer, []);
  const [input, setInput] = useState('');

  const handleAdd = () => {
    if (input.trim()) {
      dispatch({ type: 'ADD_TODO', text: input });
      setInput('');
    }
  };

  return (
    <div>
      <div>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Thêm todo..."
        />
        <button onClick={handleAdd}>Thêm</button>
      </div>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'TOGGLE_TODO', id: todo.id })}
            />
            <span style={{
              textDecoration: todo.completed ? 'line-through' : 'none'
            }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'DELETE_TODO', id: todo.id })}>
              Xóa
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### useState vs useReducer

Dùng **useState** khi:
- State đơn giản (string, number, boolean)
- Không có logic phức tạp
- Update độc lập

Dùng **useReducer** khi:
- State phức tạp (nested objects, arrays)
- Logic update phức tạp
- Nhiều sub-values liên quan
- State tiếp theo phụ thuộc vào state trước

---

## Quy Tắc Của Hooks

### Rules of Hooks

1. **Chỉ gọi Hooks ở top level**
   - Không gọi trong loops, conditions, hoặc nested functions
   - Đảm bảo Hooks được gọi theo thứ tự giống nhau mỗi lần render

2. **Chỉ gọi Hooks trong React Functions**
   - Gọi trong function components
   - Gọi trong custom Hooks

### Ví dụ SAI:

```javascript
// ❌ SAI: Hook trong condition
function Component({ condition }) {
  if (condition) {
    const [state, setState] = useState(0); // SAI!
  }
  // ...
}

// ❌ SAI: Hook trong loop
function Component({ items }) {
  items.forEach(item => {
    const [state, setState] = useState(item); // SAI!
  });
  // ...
}
```

### Ví dụ ĐÚNG:

```javascript
// ✅ ĐÚNG: Hook ở top level
function Component({ condition }) {
  const [state, setState] = useState(0);
  
  if (condition) {
    // Sử dụng state ở đây
  }
  // ...
}
```

---

## Custom Hooks

Bạn có thể tạo custom hooks để tái sử dụng logic giữa các component.

### Ví dụ: useLocalStorage

```javascript
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// Sử dụng
function App() {
  const [name, setName] = useLocalStorage('name', '');

  return (
    <input
      value={name}
      onChange={(e) => setName(e.target.value)}
      placeholder="Nhập tên của bạn"
    />
  );
}
```

### Ví dụ: useFetch

```javascript
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        const json = await response.json();
        setData(json);
        setError(null);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// Sử dụng
function UserProfile({ userId }) {
  const { data, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  return <div>{data?.name}</div>;
}
```

---

## Kết Luận

React Hooks đã thay đổi cách chúng ta viết React component, làm code ngắn gọn, dễ hiểu và dễ tái sử dụng hơn. Các hooks cơ bản bạn cần nắm vững là:

- **useState**: Quản lý state cơ bản
- **useEffect**: Xử lý side effects
- **useContext**: Chia sẻ data giữa components
- **useRef**: Truy cập DOM và lưu giá trị mutable
- **useMemo**: Tối ưu phép tính đắt đỏ
- **useCallback**: Tối ưu callback functions
- **useReducer**: Quản lý state phức tạp

Hãy thực hành nhiều để hiểu rõ cách hoạt động và biết khi nào nên dùng hook nào!

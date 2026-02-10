---
title: "Next.js App Router - Hướng dẫn cho Người mới bắt đầu"
date: 2025-12-20 01:17:00  +0700
categories: [nextjs, react]
tags: [nextjs, react]
---




## 1. Khởi tạo Project

### Cài đặt Next.js với App Router

```bash
npx create-next-app@latest my-app
```

### Trong quá trình cài đặt, chọn:
```
✔ Would you like to use TypeScript? … Yes
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like to use `src/` directory? … No
✔ Would you like to use App Router? … Yes ← QUAN TRỌNG!
✔ Would you like to customize the default import alias? … No
```

### Chạy project:
```bash
cd my-app
npm run dev
```

Truy cập: `http://localhost:3000`

### Cấu trúc thư mục ban đầu:
```
my-app/
├── app/                    ← Thư mục chính cho App Router
│   ├── layout.tsx         ← Root layout
│   ├── page.tsx           ← Trang chủ (/)
│   └── globals.css        ← CSS toàn cục
├── public/                ← Static files (images, fonts...)
├── node_modules/
├── package.json
├── tsconfig.json
└── next.config.js
```

---

## 2. Quy tắc Chia Folder và Define Routes

### 2.1. Nguyên tắc cơ bản

**📁 Mỗi folder = 1 route segment**

```
app/
├── page.tsx              → /
├── about/
│   └── page.tsx          → /about
├── blog/
│   └── page.tsx          → /blog
└── contact/
    └── page.tsx          → /contact
```

### 2.2. Các file đặc biệt trong App Router

| File | Mục đích | Bắt buộc? |
|------|----------|-----------|
| `layout.tsx` | Layout bao bọc các page con | ✅ (root) |
| `page.tsx` | Nội dung trang, tạo route | ✅ |
| `loading.tsx` | UI loading khi page đang load | ❌ |
| `error.tsx` | UI hiển thị khi có lỗi | ❌ |
| `not-found.tsx` | UI cho trang 404 | ❌ |
| `route.ts` | API endpoint (backend) | ❌ |

### 2.3. Dynamic Routes (Route động)

#### Tạo route với tham số:
```
app/
└── blog/
    ├── page.tsx                    → /blog
    └── [slug]/
        └── page.tsx                → /blog/bai-viet-1
```

**File: `app/blog/[slug]/page.tsx`**
```typescript
export default function BlogPost({ 
  params 
}: { 
  params: { slug: string } 
}) {
  return <h1>Bài viết: {params.slug}</h1>
}
```

#### Catch-all routes:
```
app/
└── docs/
    └── [...slug]/
        └── page.tsx                → /docs/a/b/c
```

```typescript
export default function DocsPage({ 
  params 
}: { 
  params: { slug: string[] } 
}) {
  return <h1>Docs: {params.slug.join('/')}</h1>
}
```

### 2.4. Route Groups (Nhóm routes không ảnh hưởng URL)

```
app/
├── (marketing)/              ← Không xuất hiện trong URL
│   ├── layout.tsx           ← Layout riêng cho marketing
│   ├── about/
│   │   └── page.tsx         → /about
│   └── contact/
│       └── page.tsx         → /contact
├── (shop)/
│   ├── layout.tsx           ← Layout riêng cho shop
│   ├── products/
│   │   └── page.tsx         → /products
│   └── cart/
│       └── page.tsx         → /cart
└── page.tsx                 → /
```

**Lợi ích:** Tổ chức code tốt hơn, mỗi nhóm có layout riêng mà không làm URL dài dòng.

### 2.5. Parallel Routes & Intercepting Routes (Nâng cao)

#### Parallel Routes:
```
app/
├── @team/
│   └── page.tsx
├── @analytics/
│   └── page.tsx
└── layout.tsx
```

#### Intercepting Routes (Modal):
```
app/
├── photos/
│   ├── page.tsx             → /photos
│   └── [id]/
│       └── page.tsx         → /photos/123
└── @modal/
    └── (..)photos/
        └── [id]/
            └── page.tsx     → Hiển thị modal khi click
```

---

## 3. Viết API Routes và Thao tác với Frontend

### 3.1. Tạo API Endpoint đơn giản

**File: `app/api/hello/route.ts`**
```typescript
import { NextResponse } from 'next/server'

// GET /api/hello
export async function GET() {
  return NextResponse.json({ message: 'Hello World!' })
}
```

### 3.2. CRUD API hoàn chỉnh

#### Structure:
```
app/
└── api/
    └── todos/
        ├── route.ts              → GET, POST /api/todos
        └── [id]/
            └── route.ts          → GET, PATCH, DELETE /api/todos/:id
```

#### File: `app/api/todos/route.ts`
```typescript
import { NextRequest, NextResponse } from 'next/server'

// Giả lập database (trong thực tế dùng Prisma, MongoDB...)
let todos = [
  { id: 1, title: 'Học Next.js', completed: false },
  { id: 2, title: 'Build API', completed: true }
]

// GET /api/todos - Lấy tất cả todos
export async function GET() {
  return NextResponse.json(todos)
}

// POST /api/todos - Tạo todo mới
export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    
    // Validation
    if (!body.title || body.title.trim() === '') {
      return NextResponse.json(
        { error: 'Title is required' },
        { status: 400 }
      )
    }
    
    const newTodo = {
      id: Date.now(),
      title: body.title,
      completed: false
    }
    
    todos.push(newTodo)
    
    return NextResponse.json(newTodo, { status: 201 })
  } catch (error) {
    return NextResponse.json(
      { error: 'Invalid request' },
      { status: 400 }
    )
  }
}
```

#### File: `app/api/todos/[id]/route.ts`
```typescript
import { NextRequest, NextResponse } from 'next/server'

// Giả lập database
let todos = [
  { id: 1, title: 'Học Next.js', completed: false }
]

// GET /api/todos/:id - Lấy 1 todo
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const id = parseInt(params.id)
  const todo = todos.find(t => t.id === id)
  
  if (!todo) {
    return NextResponse.json(
      { error: 'Todo not found' },
      { status: 404 }
    )
  }
  
  return NextResponse.json(todo)
}

// PATCH /api/todos/:id - Cập nhật todo
export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const id = parseInt(params.id)
  const body = await request.json()
  
  const todoIndex = todos.findIndex(t => t.id === id)
  
  if (todoIndex === -1) {
    return NextResponse.json(
      { error: 'Todo not found' },
      { status: 404 }
    )
  }
  
  // Update fields
  todos[todoIndex] = {
    ...todos[todoIndex],
    ...body
  }
  
  return NextResponse.json(todos[todoIndex])
}

// DELETE /api/todos/:id - Xóa todo
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const id = parseInt(params.id)
  const initialLength = todos.length
  
  todos = todos.filter(t => t.id !== id)
  
  if (todos.length === initialLength) {
    return NextResponse.json(
      { error: 'Todo not found' },
      { status: 404 }
    )
  }
  
  return NextResponse.json({ message: 'Deleted successfully' })
}
```

### 3.3. Gọi API từ Frontend

#### Client Component (với useState, useEffect)

**File: `app/todos/page.tsx`**
```typescript
'use client' // Bắt buộc khi dùng hooks

import { useState, useEffect } from 'react'

interface Todo {
  id: number
  title: string
  completed: boolean
}

export default function TodosPage() {
  const [todos, setTodos] = useState<Todo[]>([])
  const [newTodo, setNewTodo] = useState('')
  const [loading, setLoading] = useState(true)

  // Fetch todos khi component mount
  useEffect(() => {
    fetchTodos()
  }, [])

  const fetchTodos = async () => {
    try {
      const res = await fetch('/api/todos')
      const data = await res.json()
      setTodos(data)
    } catch (error) {
      console.error('Error fetching todos:', error)
    } finally {
      setLoading(false)
    }
  }

  const addTodo = async (e: React.FormEvent) => {
    e.preventDefault()
    if (!newTodo.trim()) return

    try {
      const res = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ title: newTodo })
      })
      
      const data = await res.json()
      setTodos([...todos, data])
      setNewTodo('')
    } catch (error) {
      console.error('Error adding todo:', error)
    }
  }

  const toggleTodo = async (id: number) => {
    const todo = todos.find(t => t.id === id)
    if (!todo) return

    try {
      const res = await fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed: !todo.completed })
      })
      
      const data = await res.json()
      setTodos(todos.map(t => t.id === id ? data : t))
    } catch (error) {
      console.error('Error updating todo:', error)
    }
  }

  const deleteTodo = async (id: number) => {
    try {
      await fetch(`/api/todos/${id}`, {
        method: 'DELETE'
      })
      
      setTodos(todos.filter(t => t.id !== id))
    } catch (error) {
      console.error('Error deleting todo:', error)
    }
  }

  if (loading) return <div>Loading...</div>

  return (
    <div className="max-w-2xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-8">Todo List</h1>
      
      <form onSubmit={addTodo} className="mb-8">
        <input
          type="text"
          value={newTodo}
          onChange={(e) => setNewTodo(e.target.value)}
          placeholder="Add new todo..."
          className="border p-2 mr-2"
        />
        <button 
          type="submit"
          className="bg-blue-500 text-white px-4 py-2 rounded"
        >
          Add
        </button>
      </form>

      <ul className="space-y-2">
        {todos.map(todo => (
          <li key={todo.id} className="flex items-center gap-2 p-4 border">
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span className={todo.completed ? 'line-through' : ''}>
              {todo.title}
            </span>
            <button
              onClick={() => deleteTodo(todo.id)}
              className="ml-auto text-red-500"
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

#### Server Component (fetch trực tiếp, không cần useState)

**File: `app/posts/page.tsx`**
```typescript
// Không cần 'use client' - đây là Server Component

interface Post {
  id: number
  title: string
  body: string
}

async function getPosts() {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts', {
    cache: 'no-store' // Hoặc: next: { revalidate: 60 }
  })
  
  if (!res.ok) throw new Error('Failed to fetch posts')
  
  return res.json()
}

export default async function PostsPage() {
  const posts: Post[] = await getPosts()

  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-8">Posts</h1>
      <div className="space-y-4">
        {posts.map(post => (
          <div key={post.id} className="border p-4">
            <h2 className="font-bold">{post.title}</h2>
            <p>{post.body}</p>
          </div>
        ))}
      </div>
    </div>
  )
}
```

### 3.4. Request với Headers, Cookies, Search Params

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { cookies, headers } from 'next/headers'

export async function GET(request: NextRequest) {
  // 1. Lấy Search Params
  const searchParams = request.nextUrl.searchParams
  const query = searchParams.get('q')
  
  // 2. Lấy Headers
  const authorization = request.headers.get('authorization')
  
  // 3. Lấy Cookies
  const cookieStore = cookies()
  const token = cookieStore.get('token')
  
  // 4. Set Cookies trong response
  const response = NextResponse.json({ data: 'success' })
  response.cookies.set('session', 'abc123', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 60 * 60 * 24 // 1 day
  })
  
  return response
}
```

---

## 4. Lưu ý và Best Practices

### 4.1. Server Components vs Client Components

#### 📊 So sánh:

| Tiêu chí | Server Component | Client Component |
|----------|------------------|------------------|
| **Khai báo** | Mặc định | `'use client'` ở đầu file |
| **Chạy ở** | Server | Browser |
| **Hooks** | ❌ Không dùng được | ✅ useState, useEffect... |
| **Event handlers** | ❌ onClick, onChange... | ✅ Tất cả events |
| **Browser APIs** | ❌ window, localStorage... | ✅ Tất cả APIs |
| **Fetch data** | ✅ Trực tiếp async/await | ⚠️ Dùng useEffect |
| **Bundle size** | ✅ Không gửi JS về client | ❌ Tăng bundle size |

#### ✅ Khi nào dùng Server Component:
- Fetch data từ database/API
- Truy cập backend resources
- Giữ sensitive info ở server (API keys, tokens)
- Giảm JavaScript gửi về client

#### ✅ Khi nào dùng Client Component:
- Cần interactivity (onClick, onChange...)
- Dùng React hooks (useState, useEffect...)
- Dùng browser APIs (localStorage, geolocation...)
- Dùng các thư viện chỉ chạy trên client

### 4.2. Data Fetching Strategies

#### Caching và Revalidation:

```typescript
// 1. Force cache (mặc định)
fetch('https://api.example.com/data')

// 2. No cache - luôn fresh
fetch('https://api.example.com/data', { cache: 'no-store' })

// 3. Revalidate sau mỗi 60 giây
fetch('https://api.example.com/data', { 
  next: { revalidate: 60 } 
})

// 4. Revalidate theo tag
fetch('https://api.example.com/data', { 
  next: { tags: ['posts'] } 
})
// Sau đó revalidate bằng: revalidateTag('posts')
```

### 4.3. Error Handling

#### File: `app/error.tsx`
```typescript
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-2xl font-bold mb-4">Something went wrong!</h2>
      <p className="mb-4">{error.message}</p>
      <button
        onClick={reset}
        className="bg-blue-500 text-white px-4 py-2 rounded"
      >
        Try again
      </button>
    </div>
  )
}
```

#### File: `app/not-found.tsx`
```typescript
import Link from 'next/link'

export default function NotFound() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-4xl font-bold mb-4">404 - Not Found</h2>
      <p className="mb-4">Could not find requested resource</p>
      <Link 
        href="/"
        className="text-blue-500 underline"
      >
        Return Home
      </Link>
    </div>
  )
}
```

### 4.4. Loading States

#### File: `app/dashboard/loading.tsx`
```typescript
export default function Loading() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="animate-spin rounded-full h-32 w-32 border-b-2 border-gray-900"></div>
    </div>
  )
}
```

### 4.5. Metadata và SEO

#### Static Metadata:
```typescript
// app/about/page.tsx
import { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn more about our company',
}

export default function AboutPage() {
  return <h1>About Us</h1>
}
```

#### Dynamic Metadata:
```typescript
// app/blog/[slug]/page.tsx
import { Metadata } from 'next'

export async function generateMetadata({ 
  params 
}: { 
  params: { slug: string } 
}): Promise<Metadata> {
  const post = await getPost(params.slug)
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.image],
    },
  }
}
```

### 4.6. Environment Variables

#### File: `.env.local`
```env
# Public (accessible từ browser)
NEXT_PUBLIC_API_URL=https://api.example.com

# Private (chỉ trên server)
DATABASE_URL=postgresql://...
API_SECRET_KEY=abc123xyz
```

#### Sử dụng:
```typescript
// Client Component
const apiUrl = process.env.NEXT_PUBLIC_API_URL

// Server Component hoặc API Route
const dbUrl = process.env.DATABASE_URL
```

### 4.7. Middleware

#### File: `middleware.ts` (root level)
```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // Check authentication
  const token = request.cookies.get('token')
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  return NextResponse.next()
}

// Config: chỉ chạy middleware cho các route này
export const config = {
  matcher: ['/dashboard/:path*', '/admin/:path*']
}
```

### 4.8. TypeScript Tips

#### Định nghĩa types chung:
```typescript
// types/index.ts
export interface User {
  id: number
  name: string
  email: string
}

export interface Todo {
  id: number
  title: string
  completed: boolean
  userId: number
}

export interface ApiResponse<T> {
  data: T
  message?: string
  error?: string
}
```

### 4.9. Folder Structure Tốt nhất

```
my-app/
├── app/
│   ├── (auth)/                    # Route group cho authentication
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   ├── (dashboard)/               # Route group cho dashboard
│   │   ├── layout.tsx
│   │   ├── analytics/
│   │   │   └── page.tsx
│   │   └── settings/
│   │       └── page.tsx
│   ├── api/
│   │   ├── auth/
│   │   │   └── [...nextauth]/
│   │   │       └── route.ts
│   │   ├── todos/
│   │   │   ├── route.ts
│   │   │   └── [id]/
│   │   │       └── route.ts
│   │   └── users/
│   │       └── route.ts
│   ├── layout.tsx                 # Root layout
│   ├── page.tsx                   # Home page
│   ├── error.tsx                  # Global error
│   ├── loading.tsx                # Global loading
│   └── not-found.tsx              # 404 page
├── components/                    # Reusable components
│   ├── ui/                       # UI components
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── Card.tsx
│   └── features/                 # Feature-specific components
│       ├── TodoList.tsx
│       └── UserProfile.tsx
├── lib/                          # Utility functions
│   ├── db.ts                     # Database connection
│   ├── auth.ts                   # Auth helpers
│   └── utils.ts                  # Helper functions
├── types/                        # TypeScript types
│   └── index.ts
├── public/                       # Static assets
│   ├── images/
│   └── fonts/
├── .env.local                    # Environment variables
└── package.json
```

### 4.10. Performance Tips

#### 1. **Sử dụng Image Component:**
```typescript
import Image from 'next/image'

<Image
  src="/hero.jpg"
  alt="Hero"
  width={800}
  height={600}
  priority // Load ngay cho above-the-fold images
/>
```

#### 2. **Dynamic Import (Lazy Loading):**
```typescript
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('@/components/HeavyComponent'), {
  loading: () => <p>Loading...</p>,
  ssr: false // Không render trên server
})
```

#### 3. **Suspense Boundaries:**
```typescript
import { Suspense } from 'react'

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<LoadingSkeleton />}>
        <DataComponent />
      </Suspense>
    </div>
  )
}
```


## 📚 Tài nguyên học thêm

- [Next.js Official Documentation](https://nextjs.org/docs)
- [Next.js Learn Course](https://nextjs.org/learn)
- [Vercel Examples](https://github.com/vercel/next.js/tree/canary/examples)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

---


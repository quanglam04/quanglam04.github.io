---
title: "Thread vs Process vs CPU Core: Concurrent vs Parallel "
date: 2025-12-02 01:17:00  +0700
categories: [operating-system]
tags: [operating-system, process, thread, scheduling]
---

# Thread vs Process vs CPU Core: Concurrent vs Parallel - Hiểu đúng để tối ưu hiệu năng

## 1. Thread là gì?

**Thread** là một luồng xử lý độc lập trong chương trình, được sử dụng để chạy bất đồng bộ với các luồng khác, đặc biệt là với **main thread**. Điều này giúp chương trình không bị dừng (blocking) khi gặp các tác vụ tốn thời gian như I/O operations.

### Đặc điểm của Thread

**1. Chia sẻ bộ nhớ Heap**

```java
public class SharedMemoryExample {
    // Biến này nằm trong Heap - được chia sẻ giữa các thread
    private static int counter = 0;
    
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter++; // Truy cập biến chung
            }
        });
        
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter++; // Cùng truy cập biến đó
            }
        });
        
        t1.start();
        t2.start();
        
        // Kết quả có thể KHÔNG phải 2000 do race condition!
    }
}
```

**2. Có Stack riêng biệt**

Mỗi thread có vùng nhớ Stack riêng để lưu:
- Biến local
- Tham số hàm
- Return address

```java
public void processData() {
    int localVar = 10;  // Nằm trong Stack của thread hiện tại
    String name = "test"; // Cũng trong Stack
    
    // Mỗi thread gọi hàm này sẽ có bản sao localVar riêng
}
```

**3. Đồng bộ hóa với join()**

```java
Thread worker = new Thread(() -> {
    // Xử lý tốn thời gian
    processHeavyTask();
});

worker.start(); // Chạy bất đồng bộ

// Main thread tiếp tục làm việc khác
doSomethingElse();

// Đợi worker hoàn thành trước khi tiếp tục
worker.join(); // Chạy đồng bộ từ đây

System.out.println("Worker đã xong việc!");
```

### Vấn đề Race Condition và giải pháp

**Vấn đề:**
```java
// 2 thread cùng tăng counter
Thread 1: đọc counter = 5
Thread 2: đọc counter = 5
Thread 1: tính 5 + 1 = 6, ghi vào counter
Thread 2: tính 5 + 1 = 6, ghi vào counter
// Kết quả: counter = 6 thay vì 7!
```

**Giải pháp 1: synchronized**
```java
private static int counter = 0;
private static final Object lock = new Object();

synchronized(lock) {
    counter++; // Chỉ 1 thread được vào đoạn này tại 1 thời điểm
}
```

**Giải pháp 2: Atomic variables**
```java
private static AtomicInteger counter = new AtomicInteger(0);

// Thread-safe mà không cần lock
counter.incrementAndGet();
```

**Giải pháp 3: ReentrantLock**
```java
private static final ReentrantLock lock = new ReentrantLock();

lock.lock();
try {
    counter++;
} finally {
    lock.unlock();
}
```

---

## 2. Process là gì?

**Process** hiểu nôm na là một "chương trình" độc lập đang chạy trên hệ điều hành. Mỗi process có:
- Bộ nhớ riêng biệt (Heap, Stack, Code, Data segments)
- Process ID (PID) duy nhất
- Tài nguyên hệ thống riêng (file handles, network sockets, etc.)

### So sánh Thread vs Process

| Đặc điểm | Thread | Process |
|----------|--------|---------|
| **Bộ nhớ** | Chia sẻ Heap chung | Bộ nhớ hoàn toàn độc lập |
| **Giao tiếp** | Truy cập biến trực tiếp | Cần IPC (Inter-Process Communication) |
| **Tạo mới** | Nhanh (~microseconds) | Chậm (~milliseconds) |
| **Overhead** | Nhẹ (chỉ Stack riêng) | Nặng (copy toàn bộ process) |
| **Crash handling** | 1 thread crash → cả process chết | 1 process crash → các process khác vẫn sống |

### Giao tiếp giữa các Process

**1. Shared Memory**
```c
// Process A: Tạo shared memory
int shm_id = shmget(IPC_PRIVATE, 1024, IPC_CREAT | 0666);
char *shared = (char*) shmat(shm_id, NULL, 0);
strcpy(shared, "Hello from Process A");

// Process B: Đọc shared memory
char *shared = (char*) shmat(shm_id, NULL, 0);
printf("%s\n", shared); // "Hello from Process A"
```

**2. Message Queue**
```python
# Process A: Gửi message
import multiprocessing as mp

queue = mp.Queue()
queue.put({"user_id": 123, "action": "login"})

# Process B: Nhận message
data = queue.get()
print(data)  # {"user_id": 123, "action": "login"}
```

**3. Pipes & Sockets**
```javascript
// Node.js: Parent process giao tiếp với child process
const { fork } = require('child_process');

const child = fork('worker.js');

// Gửi message
child.send({ task: 'process_data', data: [1, 2, 3] });

// Nhận kết quả
child.on('message', (result) => {
    console.log('Result:', result);
});
```

### Tại sao Process an toàn hơn Thread?

```java
// Với Thread: 1 thread crash → toàn bộ app crash
Thread dangerous = new Thread(() -> {
    throw new RuntimeException("Oops!"); // Cả app chết!
});

dangerous.start();
```

```python
# Với Process: 1 process crash → app chính vẫn sống
import multiprocessing as mp

def dangerous_task():
    raise Exception("Oops!")  # Chỉ process con chết

process = mp.Process(target=dangerous_task)
process.start()
process.join()

print("Main process vẫn chạy bình thường!")
```

**Ứng dụng thực tế:**
- **Chrome Browser**: Mỗi tab là 1 process riêng → 1 tab crash không làm chết browser
- **Nginx**: Worker processes độc lập → 1 worker lỗi không ảnh hưởng workers khác
- **Kubernetes**: Mỗi container là 1 process → isolation tốt

---

## 3. CPU Core và cơ chế xử lý

### Single Core: Concurrent (Đồng thời - giả lập song song)

Trên CPU single core, **OS không thể chạy thực sự song song** nhiều thread. Thay vào đó, nó sử dụng **Time-Slicing** (chia thời gian):

```
Timeline trên 1 CPU core:

0ms   10ms  20ms  30ms  40ms  50ms  60ms
|--A--|--B--|--C--|--A--|--B--|--C--|--A--|
 
Thread A chạy → Context Switch → Thread B chạy → Thread C → lại Thread A...
```

**Context Switch** là gì?
1. Lưu trạng thái thread hiện tại (registers, program counter)
2. Chọn thread tiếp theo từ queue
3. Khôi phục trạng thái thread mới
4. Tiếp tục thực thi

```python
# Ví dụ: Concurrent trên single core
import threading
import time

def task(name):
    for i in range(3):
        print(f"{name} đang chạy lần {i+1}")
        time.sleep(0.1)

t1 = threading.Thread(target=task, args=("Thread-A",))
t2 = threading.Thread(target=task, args=("Thread-B",))

t1.start()
t2.start()

# Output (giả lập song song):
# Thread-A đang chạy lần 1
# Thread-B đang chạy lần 1
# Thread-A đang chạy lần 2
# Thread-B đang chạy lần 2
# ...
```

**Chi phí của Context Switch:**
- Thời gian: 1-10 microseconds
- CPU cache bị xóa (L1/L2 cache miss)
- Với hàng ngàn thread → overhead lớn

### Multi-Core: Parallel (Thực sự song song)

CPU hiện đại có nhiều cores (4, 8, 16+ cores), **mỗi core như một CPU độc lập**.

```
CPU với 4 cores:

Core 0: [Thread A] ────────────────────>
Core 1: [Thread B] ────────────────────>
Core 2: [Thread C] ────────────────────>
Core 3: [Thread D] ────────────────────>

→ 4 thread chạy THẬT SỰ ĐỒNG THỜI
```

**OS Scheduler phân bố thread lên các core:**

```java
// Java: Tạo 4 thread để xử lý song song
ExecutorService executor = Executors.newFixedThreadPool(4);

for (int i = 0; i < 4; i++) {
    final int taskId = i;
    executor.submit(() -> {
        // OS sẽ tự động phân bố task này lên 1 core trống
        processData(taskId);
    });
}

// Trên CPU 4 cores: 4 task chạy song song
// Trên CPU 2 cores: 2 task song song, 2 task concurrent
```

### Minh họa sinh động: Đầu bếp trong nhà hàng

**Concurrent (1 đầu bếp - 1 core):**
```
Đầu bếp làm nhiều món cùng lúc:
1. Cho thịt vào chảo (món A)
2. Trong khi thịt nướng, chạy sang thái rau (món B)
3. Quay lại lật thịt (món A)
4. Chạy sang cho nước vào nồi (món C)
5. Lại quay về kiểm tra thịt (món A)

→ Concurrent: Làm nhiều việc nhưng không cùng 1 thời điểm
```

**Parallel (4 đầu bếp - 4 cores):**
```
Đầu bếp 1: Nướng thịt (món A)
Đầu bếp 2: Thái rau (món B)
Đầu bếp 3: Nấu súp (món C)
Đầu bếp 4: Làm tráng miệng (món D)

→ Parallel: 4 công việc thực sự cùng lúc
```

---

## 4. Thread Affinity - Gắn thread vào core cụ thể

### Tại sao cần Thread Affinity?

**Vấn đề:** Thread nhảy qua nhảy lại giữa các core → **CPU Cache miss**

```
Thread A đang chạy trên Core 0:
- Data được cache trong L1/L2 của Core 0
- Rất nhanh truy cập (nanoseconds)

OS chuyển Thread A sang Core 1:
- Cache của Core 0 bị vô dụng
- Phải load lại data từ RAM (~100ns chậm hơn)
```

**Giải pháp:** Gắn cứng thread vào 1 core

```c
// Linux: Set thread affinity
#include <pthread.h>
#include <sched.h>

cpu_set_t cpuset;
CPU_ZERO(&cpuset);
CPU_SET(2, &cpuset); // Gắn vào Core 2

pthread_t thread = pthread_self();
pthread_setaffinity_np(thread, sizeof(cpu_set_t), &cpuset);

// Bây giờ thread này CHỈ chạy trên Core 2
```

```java
// Java: Sử dụng thư viện Java Thread Affinity
AffinityLock lock = AffinityLock.acquireCore();
try {
    // Code chạy trên core cố định
    // Tận dụng tối đa CPU cache
    performCPUIntensiveTask();
} finally {
    lock.release();
}
```

### Khi nào nên dùng Thread Affinity?

 **Nên dùng:**
- **CPU-bound tasks**: Tính toán nặng (video encoding, scientific computing)
- **Low-latency applications**: Trading systems, game engines
- **Cache-sensitive workloads**: Xử lý ma trận lớn, DSP

 **Không nên dùng:**
- **I/O-bound tasks**: Đọc file, network request (thread hay bị block)
- **General web applications**: Overhead không đáng kể
- **Khi có ít thread hơn số core**: OS đã tự phân bố tốt rồi

---

## 5. Virtual Thread và CPU-bound vs I/O-bound

### Virtual Thread là gì?

**Virtual Thread** (Java 21, Kotlin Coroutines, Goroutines) là thread "nhẹ" được quản lý bởi runtime thay vì OS.

```java
// Traditional Thread
Thread t = new Thread(() -> {
    // 1 OS thread = ~1MB stack
});

// Virtual Thread
Thread.startVirtualThread(() -> {
    // ~1KB overhead, có thể tạo hàng triệu virtual threads
});
```

### Tại sao Virtual Thread không tối ưu cho CPU-bound?

**Vấn đề: Context switch quá nhiều làm cache thrashing**

```java
// 1 triệu virtual threads cùng tính toán
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

for (int i = 0; i < 1_000_000; i++) {
    executor.submit(() -> {
        // CPU-intensive: Tính số fibonacci
        calculateFibonacci(40); 
    });
}

// Trên single core:
// - 1 triệu lần context switch
// - CPU cache liên tục bị xóa
// - Chậm hơn cả thread pool cố định!
```

**So sánh:**

| Loại Task | Thread Pool cố định | Virtual Thread |
|-----------|---------------------|----------------|
| **CPU-bound** |  Tốt (ít context switch) |  Tệ (nhiều context switch) |
| **I/O-bound** |  Tệ (thread bị block) |  Rất tốt (cheap to block) |

### Virtual Thread sinh ra cho I/O-bound

```java
// I/O-bound: Gọi 10,000 API đồng thời
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

for (int i = 0; i < 10_000; i++) {
    executor.submit(() -> {
        // Virtual thread bị block khi chờ response
        String response = httpClient.get("https://api.example.com");
        
        // OS thread thực tế vẫn tự do xử lý virtual thread khác
        // → Tận dụng CPU tốt hơn
    });
}

// Traditional thread pool với 10,000 threads:
// - 10,000 OS threads = ~10GB RAM
// - Context switch overhead khủng khiếp

// Virtual threads:
// - 10,000 virtual threads = ~10MB RAM
// - Chỉ cần vài chục OS threads
```

---

## 6. Lựa chọn công nghệ phù hợp

### Bảng chọn lựa nhanh

```
┌─────────────────────┬──────────────────┬─────────────────┐
│   Loại công việc    │  Công nghệ tốt   │   Lý do         │
├─────────────────────┼──────────────────┼─────────────────┤
│ CPU-bound           │ Thread Pool      │ Ít context      │
│ (tính toán nặng)    │ cố định          │ switch, tận     │
│                     │ (= số core)      │ dụng cache      │
├─────────────────────┼──────────────────┼─────────────────┤
│ I/O-bound           │ Virtual Thread   │ Nhẹ, scale tốt, │
│ (network, file IO)  │ Async/Await      │ không tốn RAM   │
├─────────────────────┼──────────────────┼─────────────────┤
│ Mixed workload      │ Separate pools   │ Tách biệt CPU   │
│                     │                  │ và I/O tasks    │
├─────────────────────┼──────────────────┼─────────────────┤
│ Ultra low latency   │ Thread Affinity  │ Tận dụng cache, │
│ (trading, gaming)   │ + Lock-free DS   │ giảm latency    │
├─────────────────────┼──────────────────┼─────────────────┤
│ Isolation cần cao   │ Multi-process    │ Crash safety,   │
│ (browser, plugins)  │                  │ security        │
└─────────────────────┴──────────────────┴─────────────────┘
```

### Ví dụ thực tế

**1. Web Server xử lý API**
```javascript
// I/O-bound: Đợi database, gọi external API
//  Dùng async/await (giống virtual thread)

app.get('/users/:id', async (req, res) => {
    const user = await db.query('SELECT * FROM users WHERE id = ?', [req.params.id]);
    const posts = await fetch(`https://api.posts.com/user/${user.id}`);
    res.json({ user, posts });
});
```

**2. Video Encoding Service**
```python
# CPU-bound: Encode video tốn CPU
#  Dùng multiprocessing (tận dụng đủ cores)

from multiprocessing import Pool

def encode_video(video_path):
    # CPU-intensive task
    return ffmpeg.encode(video_path)

with Pool(processes=8) as pool:  # 8 cores
    results = pool.map(encode_video, video_files)
```

**3. Real-time Trading System**
```c++
// Ultra low-latency
//  Thread affinity + lock-free data structures

// Pin thread vào core 0
cpu_set_t cpuset;
CPU_ZERO(&cpuset);
CPU_SET(0, &cpuset);
pthread_setaffinity_np(pthread_self(), sizeof(cpu_set_t), &cpuset);

// Lock-free queue cho low latency
boost::lockfree::queue<Order> order_queue;
```

---

## 7. Những lầm tưởng phổ biến

###  Lầm 1: "Càng nhiều thread càng nhanh"

```java
// SAI: Tạo 1000 threads cho CPU-bound task trên CPU 4 cores
ExecutorService executor = Executors.newFixedThreadPool(1000);

// ĐÚNG: Thread pool = số core
ExecutorService executor = Executors.newFixedThreadPool(
    Runtime.getRuntime().availableProcessors() // = 4
);
```

###  Lầm 2: "Thread và Process giống nhau"

```python
# Thread: Chia sẻ memory → dễ race condition
counter = 0
def increment():
    global counter
    counter += 1  # NOT thread-safe!

# Process: Memory độc lập → an toàn nhưng chậm hơn
from multiprocessing import Value
counter = Value('i', 0)  # Shared counter giữa processes
```

###  Lầm 3: "Virtual thread phù hợp mọi trường hợp"

```java
// SAI: Dùng virtual thread cho CPU-intensive
Thread.startVirtualThread(() -> {
    calculatePrimes(1_000_000); // CPU-bound → Chậm!
});

// ĐÚNG: Dùng cho I/O-intensive
Thread.startVirtualThread(() -> {
    httpClient.get(url); // I/O-bound → Rất nhanh!
});
```

---

## 8. Tổng kết

### Điểm chính cần nhớ

1. **Thread**: Luồng xử lý trong process, chia sẻ Heap, có Stack riêng
2. **Process**: Chương trình độc lập, bộ nhớ riêng biệt, an toàn hơn nhưng nặng hơn
3. **Concurrent**: Giả lập song song trên single core (context switch)
4. **Parallel**: Thực sự song song trên multi-core
5. **Thread Affinity**: Gắn thread vào core để tận dụng cache (chỉ cho CPU-bound)
6. **Virtual Thread**: Sinh ra cho I/O-bound, không tối ưu cho CPU-bound

### Công thức chọn lựa

```
IF (task là I/O-bound):
    → Dùng Virtual Thread / Async-Await
    → Tạo nhiều threads không sao (rẻ, nhẹ)

ELSE IF (task là CPU-bound):
    → Dùng Thread Pool = số cores
    → Cân nhắc Thread Affinity nếu cần ultra low-latency

ELSE IF (cần isolation cao):
    → Dùng Multi-process
    → Chấp nhận overhead để đổi lấy stability

ELSE IF (mixed workload):
    → Tách riêng thread pools cho CPU và I/O tasks
```

Hi vọng bài viết giúp bạn hiểu rõ bản chất của Thread, Process, và cách CPU xử lý chúng. Khi nắm vững kiến thức này, bạn sẽ thiết kế được hệ thống với hiệu năng và độ ổn định tối ưu! 🚀

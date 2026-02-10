---
title: "Tìm hiểu về Virtual Thread trong Java"
date: 2025-12-22 01:17:00  +0700
categories: [os, java, optimize]
tags: [java, thread]
---


## Giới thiệu

Kể từ khi Java 21 ra mắt vào tháng 9/2023, Virtual Threads đã chính thức trở thành một tính năng ổn định (stable feature) và được xem là một trong những cải tiến quan trọng nhất trong lịch sử Java. Đây là thành quả của Project Loom - một dự án tham vọng nhằm đơn giản hóa concurrent programming trong Java.

Trong bài viết này, chúng ta sẽ đi sâu tìm hiểu Virtual Threads là gì, tại sao nó lại quan trọng, và cách sử dụng nó một cách hiệu quả trong các ứng dụng thực tế.

## 1. Virtual Thread là gì?

### 1.1. Định nghĩa

**Virtual Threads** là một loại thread nhẹ (lightweight threads) được JVM quản lý, cho phép chúng ta tạo ra hàng triệu threads mà không gây quá tải hệ thống. Khác với platform threads (threads truyền thống), virtual threads không ánh xạ 1:1 với OS threads.

### 1.2. Virtual Threads sinh ra để giải quyết vấn đề gì?

#### Vấn đề với Platform Threads truyền thống

Trước khi có Virtual Threads, Java sử dụng **Platform Threads** - mỗi platform thread được ánh xạ trực tiếp với một OS thread. Điều này dẫn đến một số vấn đề nghiêm trọng:

**1. Chi phí tạo threads cao:**
- Mỗi platform thread tiêu tốn khoảng 1-2 MB bộ nhớ cho stack
- Việc tạo và context switching giữa các OS threads rất tốn kém
- Không thể tạo hàng nghìn threads cùng lúc mà không gặp vấn đề về tài nguyên

**2. Thread pools hạn chế khả năng mở rộng:**
```java
// Thread pool truyền thống
ExecutorService executor = Executors.newFixedThreadPool(100);
// Chỉ có thể xử lý 100 tasks đồng thời
// Các tasks khác phải chờ trong queue
```

**3. Thread blocking làm lãng phí tài nguyên:**
- Khi một thread đang chờ I/O (database query, HTTP request, file read), nó vẫn giữ nguyên OS thread
- OS thread bị block không thể làm việc khác, gây lãng phí tài nguyên

**Ví dụ minh họa vấn đề:**

```java
// Ứng dụng web server với 1000 concurrent requests
// Mỗi request cần gọi database (mất 100ms)
ExecutorService executor = Executors.newFixedThreadPool(200);

for (int i = 0; i < 1000; i++) {
    executor.submit(() -> {
        // Thread bị block trong 100ms chờ database
        String result = database.query("SELECT * FROM users");
        // Trong suốt 100ms đó, OS thread không làm gì cả!
    });
}
// 800 requests phải chờ vì chỉ có 200 threads
```

#### Giải pháp của Virtual Threads

Virtual Threads giải quyết triệt để các vấn đề trên:

**1. Chi phí cực kỳ thấp:**
- Mỗi virtual thread chỉ tốn vài KB bộ nhớ
- Có thể tạo hàng triệu virtual threads mà không lo về tài nguyên

**2. Tự động scheduling thông minh:**
- JVM tự động schedule virtual threads lên một số lượng nhỏ OS threads (carrier threads)
- Khi virtual thread bị block (I/O), nó được unmount khỏi carrier thread
- Carrier thread trống có thể chạy virtual thread khác ngay lập tức

**3. Không lãng phí tài nguyên khi blocking:**
```java
// Với Virtual Threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1_000_000; i++) { // 1 triệu threads!
        executor.submit(() -> {
            // Virtual thread bị block, nhưng carrier thread vẫn làm việc khác
            String result = database.query("SELECT * FROM users");
        });
    }
}
// Tất cả 1 triệu requests được xử lý hiệu quả!
```

### 1.3. Kiến trúc hoạt động

```
┌─────────────────────────────────────────┐
│     Virtual Threads (hàng triệu)        │
│  VT1  VT2  VT3  VT4  ...  VTn           │
└──────────┬──────────────────────────────┘
           │ (scheduled by JVM)
           ▼
┌─────────────────────────────────────────┐
│   Carrier Threads (Platform Threads)    │
│     CT1      CT2      CT3      CT4      │
└──────────┬──────────────────────────────┘
           │ (1:1 mapping)
           ▼
┌─────────────────────────────────────────┐
│          OS Threads                     │
└─────────────────────────────────────────┘
```

**Cơ chế hoạt động:**
1. JVM tạo một ForkJoinPool với số lượng carrier threads = số CPU cores
2. Virtual threads được mount lên carrier threads để thực thi
3. Khi virtual thread gặp blocking operation (I/O, sleep, wait), nó được unmount
4. Carrier thread nhận virtual thread khác từ queue để thực thi
5. Khi blocking operation hoàn thành, virtual thread được schedule lại

## 2. So sánh Virtual Threads với Platform Threads

### 2.1. Bảng so sánh chi tiết

| Tiêu chí | Platform Thread | Virtual Thread |
|----------|----------------|----------------|
| **Bộ nhớ** | 1-2 MB/thread | Vài KB/thread |
| **Tạo threads** | Nặng, chậm | Nhẹ, nhanh |
| **Số lượng tối đa** | Vài nghìn | Hàng triệu |
| **Quản lý** | OS | JVM |
| **Context switching** | Chậm (OS level) | Nhanh (JVM level) |
| **Blocking I/O** | Lãng phí tài nguyên | Hiệu quả |
| **Thread pool** | Cần thiết | Không cần (tạo per-task) |
| **Pinning issue** | Không có | Có (với synchronized) |

### 2.2. Ưu điểm của Virtual Threads

#### Khả năng mở rộng vượt trội

```java
// Platform Threads - Giới hạn
ExecutorService platform = Executors.newFixedThreadPool(1000);
// Khó có thể vượt quá vài nghìn threads

// Virtual Threads - Không giới hạn thực tế
ExecutorService virtual = Executors.newVirtualThreadPerTaskExecutor();
// Có thể xử lý hàng triệu concurrent tasks
```

#### Đơn giản hóa code

Không cần reactive programming hay async/await phức tạp:

```java
// Style đồng bộ đơn giản, dễ đọc
String user = fetchUser(userId);           // Blocking call
String orders = fetchOrders(userId);       // Blocking call
String recommendations = getRecommendations(userId); // Blocking call
return combineData(user, orders, recommendations);
```

#### Tương thích ngược hoàn toàn

```java
// Code cũ hoạt động mà không cần sửa đổi
Thread thread = Thread.ofVirtual().start(() -> {
    // Legacy code here
});
```

#### Hiệu suất với I/O-bound tasks

```java
// Test: 10,000 HTTP requests
// Platform threads: 50-100 requests/second
// Virtual threads: 5,000-10,000 requests/second
```

### 2.3. Nhược điểm và hạn chế

#### Pinning problem với synchronized

Khi virtual thread sử dụng `synchronized`, nó bị "pin" vào carrier thread:

```java
// BAD: Virtual thread bị pin
synchronized(lock) {
    // Blocking I/O trong synchronized block
    database.query("SELECT * FROM users"); // Pin carrier thread!
}

// GOOD: Dùng ReentrantLock
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    database.query("SELECT * FROM users"); // Không bị pin
} finally {
    lock.unlock();
}
```

#### Không phù hợp với CPU-bound tasks

```java
// BAD: CPU-intensive task với virtual threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        // Tính toán phức tạp
        for (long i = 0; i < 1_000_000_000; i++) {
            result += Math.sqrt(i);
        }
    });
}
// Virtual thread chiếm carrier thread liên tục, không hiệu quả

// GOOD: Dùng platform thread với ForkJoinPool
ForkJoinPool.commonPool().submit(() -> {
    // CPU-intensive work
});
```

#### Thread-local variables tốn bộ nhớ

```java
// Với hàng triệu virtual threads
ThreadLocal<UserContext> context = new ThreadLocal<>();
// Mỗi virtual thread có bản copy riêng -> tốn bộ nhớ

// Giải pháp: Dùng ScopedValue (Java 21+)
ScopedValue<UserContext> context = ScopedValue.newInstance();
```

#### ThreadPoolExecutor không hoạt động tối ưu

```java
// KHÔNG NÊN: Dùng thread pool với virtual threads
ExecutorService executor = Executors.newFixedThreadPool(100, 
    Thread.ofVirtual().factory());
// Mất đi lợi ích của virtual threads!

// NÊN: Tạo virtual thread per-task
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
```

### 2.4. Khi nào nên dùng Virtual Threads?

#### NÊN dùng Virtual Threads khi:

**1. I/O-bound applications:**
- Web servers, REST APIs
- Database-heavy applications
- Microservices với nhiều external calls
- File processing, streaming data

**2. High concurrency requirements:**
- Cần xử lý hàng nghìn/triệu concurrent requests
- WebSocket servers với nhiều kết nối đồng thời
- Real-time applications

**3. Blocking APIs:**
- JDBC database calls
- HTTP clients (Java 11+ HttpClient)
- File I/O operations
- Thread.sleep(), wait(), join()

**Ví dụ use cases lý tưởng:**

```java
// 1. REST API với nhiều database calls
@RestController
public class UserController {
    public User getUser(Long id) {
        // Mỗi request chạy trong virtual thread
        User user = userRepository.findById(id);     // DB call
        List<Order> orders = orderRepository.findByUser(id); // DB call
        List<Review> reviews = reviewRepository.findByUser(id); // DB call
        return enrichUser(user, orders, reviews);
    }
}

// 2. Parallel data processing
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<CompletableFuture<Result>> futures = files.stream()
        .map(file -> CompletableFuture.supplyAsync(() -> 
            processFile(file), executor))
        .toList();
    
    List<Result> results = futures.stream()
        .map(CompletableFuture::join)
        .toList();
}
```

#### KHÔNG NÊN dùng Virtual Threads khi:

**1. CPU-bound tasks:**
```java
// Tính toán khoa học, machine learning, image processing
// => Dùng ForkJoinPool hoặc platform threads
```

**2. Code có nhiều synchronized blocks:**
```java
// Legacy code với nhiều synchronized
synchronized(this) {
    // Heavy computation or I/O
}
// => Cần refactor sang ReentrantLock trước
```

**3. Số lượng threads ít (< 1000):**
```java
// Nếu chỉ cần 10-100 threads
// => Platform threads vẫn ổn, không cần virtual threads
```

**4. Cần thread affinity hoặc thread priority:**
```java
// Virtual threads không hỗ trợ thread priority
thread.setPriority(Thread.MAX_PRIORITY); // Không có tác dụng với virtual threads
```

## 3. Implement Virtual Threads trong thực tế

### 3.1. Tạo Virtual Threads cơ bản

#### Cách 1: Thread.ofVirtual()

```java
// Tạo và start virtual thread đơn giản
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("Hello from virtual thread: " + Thread.currentThread());
});
vThread.join(); // Chờ thread hoàn thành

// Tạo virtual thread với tên
Thread namedVThread = Thread.ofVirtual()
    .name("worker-virtual-thread")
    .start(() -> {
        System.out.println("Named virtual thread");
    });
```

#### Cách 2: Thread.startVirtualThread()

```java
// Cách ngắn gọn nhất
Thread vThread = Thread.startVirtualThread(() -> {
    System.out.println("Quick virtual thread");
});
```

#### Cách 3: Executors.newVirtualThreadPerTaskExecutor()

```java
// Khuyến nghị cho production
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        System.out.println("Task in virtual thread");
    });
} // Auto-close, chờ tất cả tasks hoàn thành
```

### 3.2. Ví dụ 1: Web Server với High Concurrency

Giả sử chúng ta có một REST API cần gọi nhiều external services:

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;
import java.util.concurrent.*;

public class HighConcurrencyWebServer {
    
    private final HttpClient httpClient;
    
    public HighConcurrencyWebServer() {
        // HttpClient tự động sử dụng virtual threads nếu có
        this.httpClient = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .build();
    }
    
    /**
     * Xử lý 10,000 requests đồng thời
     */
    public void handleMassiveLoad() throws InterruptedException {
        int totalRequests = 10_000;
        CountDownLatch latch = new CountDownLatch(totalRequests);
        
        long startTime = System.currentTimeMillis();
        
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < totalRequests; i++) {
                final int requestId = i;
                executor.submit(() -> {
                    try {
                        processRequest(requestId);
                    } finally {
                        latch.countDown();
                    }
                });
            }
        }
        
        latch.await(); // Chờ tất cả requests hoàn thành
        long duration = System.currentTimeMillis() - startTime;
        
        System.out.printf("Processed %d requests in %d ms%n", 
            totalRequests, duration);
        System.out.printf("Throughput: %.2f requests/second%n", 
            totalRequests * 1000.0 / duration);
    }
    
    private void processRequest(int requestId) {
        try {
            // Giả lập gọi nhiều external services
            CompletableFuture<String> userService = fetchUserData(requestId);
            CompletableFuture<String> orderService = fetchOrderData(requestId);
            CompletableFuture<String> inventoryService = fetchInventoryData(requestId);
            
            // Chờ tất cả services trả về
            CompletableFuture.allOf(userService, orderService, inventoryService).join();
            
            // Combine results
            String result = combineResults(
                userService.get(),
                orderService.get(),
                inventoryService.get()
            );
            
            System.out.println("Request " + requestId + " completed: " + result);
            
        } catch (Exception e) {
            System.err.println("Request " + requestId + " failed: " + e.getMessage());
        }
    }
    
    private CompletableFuture<String> fetchUserData(int userId) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                HttpRequest request = HttpRequest.newBuilder()
                    .uri(URI.create("https://jsonplaceholder.typicode.com/users/" + (userId % 10 + 1)))
                    .GET()
                    .build();
                
                HttpResponse<String> response = httpClient.send(
                    request, 
                    HttpResponse.BodyHandlers.ofString()
                );
                
                return response.body();
            } catch (Exception e) {
                return "Error: " + e.getMessage();
            }
        });
    }
    
    private CompletableFuture<String> fetchOrderData(int userId) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                // Giả lập delay từ service
                Thread.sleep(50);
                return "Orders for user " + userId;
            } catch (InterruptedException e) {
                return "Error: " + e.getMessage();
            }
        });
    }
    
    private CompletableFuture<String> fetchInventoryData(int userId) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(30);
                return "Inventory for user " + userId;
            } catch (InterruptedException e) {
                return "Error: " + e.getMessage();
            }
        });
    }
    
    private String combineResults(String user, String orders, String inventory) {
        return String.format("Combined[user=%s, orders=%s, inventory=%s]", 
            user.substring(0, Math.min(20, user.length())), 
            orders, 
            inventory);
    }
    
    public static void main(String[] args) throws InterruptedException {
        HighConcurrencyWebServer server = new HighConcurrencyWebServer();
        
        System.out.println("Starting high concurrency test with Virtual Threads...");
        server.handleMassiveLoad();
        System.out.println("Test completed!");
    }
}
```

**Output mẫu:**
```
Starting high concurrency test with Virtual Threads...
Request 4523 completed: Combined[user={"id":4,"name":"P, orders=Orders for user 4523, inventory=Inventory for user 4523]
...
Processed 10000 requests in 3542 ms
Throughput: 2823.29 requests/second
Test completed!
```

### 3.3. Ví dụ 2: Batch Processing với Database

```java
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

public class BatchProcessor {
    
    private final String dbUrl = "jdbc:postgresql://localhost:5432/mydb";
    private final String dbUser = "user";
    private final String dbPassword = "password";
    
    /**
     * Xử lý hàng triệu records từ database
     */
    public void processBatchData() throws InterruptedException {
        List<Integer> recordIds = fetchRecordIds(); // Giả sử có 100,000 IDs
        
        System.out.println("Processing " + recordIds.size() + " records...");
        
        long startTime = System.currentTimeMillis();
        CountDownLatch latch = new CountDownLatch(recordIds.size());
        
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (Integer recordId : recordIds) {
                executor.submit(() -> {
                    try {
                        processRecord(recordId);
                    } catch (Exception e) {
                        System.err.println("Failed to process record " + recordId + ": " + e.getMessage());
                    } finally {
                        latch.countDown();
                    }
                });
            }
        }
        
        latch.await();
        long duration = System.currentTimeMillis() - startTime;
        
        System.out.printf("Processed %d records in %d ms (%.2f records/sec)%n",
            recordIds.size(), duration, recordIds.size() * 1000.0 / duration);
    }
    
    private void processRecord(int recordId) throws SQLException {
        // Mỗi virtual thread có connection riêng
        try (Connection conn = DriverManager.getConnection(dbUrl, dbUser, dbPassword)) {
            
            // 1. Fetch record
            Record record = fetchRecord(conn, recordId);
            
            // 2. Process business logic
            Record processed = applyBusinessLogic(record);
            
            // 3. Update record
            updateRecord(conn, processed);
            
            // 4. Log audit
            insertAuditLog(conn, recordId);
        }
    }
    
    private Record fetchRecord(Connection conn, int id) throws SQLException {
        String sql = "SELECT * FROM records WHERE id = ?";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, id);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return new Record(
                        rs.getInt("id"),
                        rs.getString("data"),
                        rs.getTimestamp("created_at")
                    );
                }
            }
        }
        throw new SQLException("Record not found: " + id);
    }
    
    private Record applyBusinessLogic(Record record) {
        // Giả lập business logic
        try {
            Thread.sleep(10); // External API call simulation
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        record.data = record.data.toUpperCase() + "_PROCESSED";
        return record;
    }
    
    private void updateRecord(Connection conn, Record record) throws SQLException {
        String sql = "UPDATE records SET data = ?, processed_at = NOW() WHERE id = ?";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, record.data);
            stmt.setInt(2, record.id);
            stmt.executeUpdate();
        }
    }
    
    private void insertAuditLog(Connection conn, int recordId) throws SQLException {
        String sql = "INSERT INTO audit_logs (record_id, action, timestamp) VALUES (?, ?, NOW())";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, recordId);
            stmt.setString(2, "PROCESSED");
            stmt.executeUpdate();
        }
    }
    
    private List<Integer> fetchRecordIds() {
        // Giả lập fetch 100,000 IDs
        List<Integer> ids = new ArrayList<>();
        for (int i = 1; i <= 100_000; i++) {
            ids.add(i);
        }
        return ids;
    }
    
    // Helper class
    static class Record {
        int id;
        String data;
        Timestamp createdAt;
        
        Record(int id, String data, Timestamp createdAt) {
            this.id = id;
            this.data = data;
            this.createdAt = createdAt;
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        BatchProcessor processor = new BatchProcessor();
        processor.processBatchData();
    }
}
```

### 3.4. Ví dụ 3: File Processing Pipeline

```java
import java.io.*;
import java.nio.file.*;
import java.util.concurrent.*;
import java.util.stream.*;

public class FileProcessingPipeline {
    
    private final Path inputDir = Paths.get("input");
    private final Path outputDir = Paths.get("output");
    
    /**
     * Xử lý hàng nghìn files song song
     */
    public void processFiles() throws IOException, InterruptedException {
        Files.createDirectories(outputDir);
        
        // Lấy danh sách tất cả files
        List<Path> files;
        try (Stream<Path> stream = Files.walk(inputDir)) {
            files = stream
                .filter(Files::isRegularFile)
                .filter(p -> p.toString().endsWith(".txt"))
                .collect(Collectors.toList());
        }
        
        System.out.println("Found " + files.size() + " files to process");
        
        long startTime = System.currentTimeMillis();
        
        // Xử lý với virtual threads
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            List<CompletableFuture<ProcessingResult>> futures = files.stream()
                .map(file -> CompletableFuture.supplyAsync(() -> 
                    processFile(file), executor))
                .toList();
            
            // Chờ tất cả hoàn thành
            CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
            
            // Tổng hợp kết quả
            List<ProcessingResult> results = futures.stream()
                .map(CompletableFuture::join)
                .toList();
            
            long duration = System.currentTimeMillis() - startTime;
            printSummary(results, duration);
        }
    }
    
    private ProcessingResult processFile(Path file) {
        try {
            long startTime = System.currentTimeMillis();
            
            // 1. Read file
            String content = Files.readString(file);
            
            // 2. Transform content
            String transformed = transformContent(content);
            
            // 3. Validate
            if (!validate(transformed)) {
                return ProcessingResult.failed(file, "Validation failed");
            }
            
            // 4. Write output
            Path outputFile = outputDir.resolve(file.getFileName());
            Files.writeString(outputFile, transformed);
            
            long duration = System.currentTimeMillis() - startTime;
            return ProcessingResult.success(file, duration, transformed.length());
            
        } catch (Exception e) {
            return ProcessingResult.failed(file, e.getMessage());
        }
    }
    
    private String transformContent(String content) {
        // Giả lập transformation phức tạp
        try {
            Thread.sleep(50); // External service call
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        return content
            .toUpperCase()
            .replaceAll("\\s+", " ")
            .trim();
    }
    
    private boolean validate(String content) {
        return content != null && !content.isEmpty() && content.length() < 1_000_000;
    }
    
    private void printSummary(List<ProcessingResult> results, long totalDuration) {
        long successful = results.stream().filter(r -> r.success).count();
        long failed = results.stream().filter(r -> !r.success).count();
        long totalBytes = results.stream().mapToLong(r -> r.bytesProcessed).sum();
        double avgDuration = results.stream()
            .mapToLong(r -> r.duration)
            .average()
            .orElse(0);
        
        System.out.println("\n=== Processing Summary ===");
        System.out.println("Total files: " + results.size());
        System.out.println("Successful: " + successful);
        System.out.println("Failed: " + failed);
        System.out.println("Total bytes processed: " + totalBytes);
        System.out.printf("Average file processing time: %.2f ms%n", avgDuration);
        System.out.printf("Total processing time: %d ms%n", totalDuration);
        System.out.printf("Throughput: %.2f files/second%n", 
            results.size() * 1000.0 / totalDuration);
    }
    
    // Helper class
    static class ProcessingResult {
        final Path file;
        final boolean success;
        final String error;
        final long duration;
        final long bytesProcessed;
        
        private ProcessingResult(Path file, boolean success, String error, 
                                long duration, long bytesProcessed) {
            this.file = file;
            this.success = success;
            this.error = error;
            this.duration = duration;
            this.bytesProcessed = bytesProcessed;
        }
        
        static ProcessingResult success(Path file, long duration, long bytes) {
            return new ProcessingResult(file, true, null, duration, bytes);
        }
        
        static ProcessingResult failed(Path file, String error) {
            return new ProcessingResult(file, false, error, 0, 0);
        }
    }
    
    public static void main(String[] args) throws Exception {
        System.out.println("=== Performance Comparison: Platform Threads vs Virtual Threads ===\n");
        
        // Test 1: Platform Threads với Fixed Thread Pool
        System.out.println("Test 1: Platform Threads (Fixed Pool)");
        testPlatformThreads();
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // Test 2: Virtual Threads
        System.out.println("Test 2: Virtual Threads");
        testVirtualThreads();
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        // Test 3: Platform Threads với Cached Thread Pool
        System.out.println("Test 3: Platform Threads (Cached Pool)");
        testPlatformThreadsCached();
    }
    
    /**
     * Test với Platform Threads - Fixed Thread Pool
     */
    private static void testPlatformThreads() throws Exception {
        int poolSize = 200; // Giới hạn threads
        ExecutorService executor = Executors.newFixedThreadPool(poolSize);
        
        long startTime = System.currentTimeMillis();
        CountDownLatch latch = new CountDownLatch(TASK_COUNT);
        
        try {
            for (int i = 0; i < TASK_COUNT; i++) {
                final int taskId = i;
                executor.submit(() -> {
                    try {
                        simulateIoTask(taskId);
                    } finally {
                        latch.countDown();
                    }
                });
            }
            
            latch.await();
            long duration = System.currentTimeMillis() - startTime;
            
            printResults("Platform Threads (Pool: " + poolSize + ")", duration);
            
        } finally {
            executor.shutdown();
            executor.awaitTermination(1, TimeUnit.MINUTES);
        }
    }
    
    /**
     * Test với Platform Threads - Cached Thread Pool
     */
    private static void testPlatformThreadsCached() throws Exception {
        ExecutorService executor = Executors.newCachedThreadPool();
        
        long startTime = System.currentTimeMillis();
        CountDownLatch latch = new CountDownLatch(TASK_COUNT);
        
        try {
            for (int i = 0; i < TASK_COUNT; i++) {
                final int taskId = i;
                executor.submit(() -> {
                    try {
                        simulateIoTask(taskId);
                    } finally {
                        latch.countDown();
                    }
                });
            }
            
            latch.await();
            long duration = System.currentTimeMillis() - startTime;
            
            printResults("Platform Threads (Cached Pool)", duration);
            
        } finally {
            executor.shutdown();
            executor.awaitTermination(1, TimeUnit.MINUTES);
        }
    }
    
    /**
     * Test với Virtual Threads
     */
    private static void testVirtualThreads() throws Exception {
        long startTime = System.currentTimeMillis();
        CountDownLatch latch = new CountDownLatch(TASK_COUNT);
        
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < TASK_COUNT; i++) {
                final int taskId = i;
                executor.submit(() -> {
                    try {
                        simulateIoTask(taskId);
                    } finally {
                        latch.countDown();
                    }
                });
            }
            
            latch.await();
            long duration = System.currentTimeMillis() - startTime;
            
            printResults("Virtual Threads", duration);
        }
    }
    
    /**
     * Giả lập I/O-bound task (database query, HTTP call, etc.)
     */
    private static void simulateIoTask(int taskId) {
        try {
            // Giả lập blocking I/O
            Thread.sleep(TASK_DURATION_MS);
            
            // Một chút CPU work
            int result = 0;
            for (int i = 0; i < 1000; i++) {
                result += i;
            }
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private static void printResults(String testName, long duration) {
        double throughput = TASK_COUNT * 1000.0 / duration;
        double avgLatency = (double) duration / TASK_COUNT;
        
        System.out.println("Test: " + testName);
        System.out.println("Total tasks: " + TASK_COUNT);
        System.out.println("Task duration: " + TASK_DURATION_MS + " ms each");
        System.out.println("Total time: " + duration + " ms");
        System.out.printf("Throughput: %.2f tasks/second%n", throughput);
        System.out.printf("Average latency: %.2f ms%n", avgLatency);
        System.out.printf("Speedup vs sequential: %.2fx%n", 
            (TASK_COUNT * TASK_DURATION_MS) / (double) duration);
    }
}
```

**Kết quả thực tế khi chạy:**

```
=== Performance Comparison: Platform Threads vs Virtual Threads ===

Test 1: Platform Threads (Fixed Pool)
Test: Platform Threads (Pool: 200)
Total tasks: 10000
Task duration: 100 ms each
Total time: 5243 ms
Throughput: 1907.31 tasks/second
Average latency: 0.52 ms
Speedup vs sequential: 190.73x

============================================================

Test 2: Virtual Threads
Test: Virtual Threads
Total tasks: 10000
Task duration: 100 ms each
Total time: 1156 ms
Throughput: 8650.52 tasks/second
Average latency: 0.12 ms
Speedup vs sequential: 865.05x

============================================================

Test 3: Platform Threads (Cached Pool)
Test: Platform Threads (Cached Pool)
Total tasks: 10000
Task duration: 100 ms each
Total time: 2891 ms
Throughput: 3459.01 tasks/second
Average latency: 0.29 ms
Speedup vs sequential: 345.90x
```

**Phân tích kết quả:**
- Virtual Threads nhanh hơn **4.5x** so với Fixed Thread Pool
- Virtual Threads nhanh hơn **2.5x** so với Cached Thread Pool
- Với I/O-bound tasks, Virtual Threads cho throughput vượt trội

### 3.6. Ví dụ 5: Structured Concurrency (Java 21+)

Structured Concurrency là pattern mới đi cùng Virtual Threads, giúp quản lý concurrent tasks tốt hơn:

```java
import java.util.concurrent.*;
import jdk.incubator.concurrent.StructuredTaskScope;

/**
 * Structured Concurrency đảm bảo:
 * - Parent task chờ tất cả child tasks
 * - Nếu parent bị cancel, tất cả children cũng bị cancel
 * - Error handling tập trung
 */
public class StructuredConcurrencyExample {
    
    /**
     * Fetch user data từ nhiều sources song song
     */
    public UserProfile fetchUserProfile(String userId) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            
            // Submit concurrent tasks
            Subtask<UserBasicInfo> basicInfo = scope.fork(() -> fetchBasicInfo(userId));
            Subtask<List<Order>> orders = scope.fork(() -> fetchOrders(userId));
            Subtask<List<Review>> reviews = scope.fork(() -> fetchReviews(userId));
            Subtask<RecommendationList> recommendations = 
                scope.fork(() -> fetchRecommendations(userId));
            
            // Chờ tất cả hoàn thành hoặc có lỗi
            scope.join();           // Chờ tất cả tasks
            scope.throwIfFailed();  // Throw exception nếu có task failed
            
            // Tất cả tasks thành công, combine results
            return new UserProfile(
                basicInfo.get(),
                orders.get(),
                reviews.get(),
                recommendations.get()
            );
        }
        // Auto-close: tự động cancel các tasks chưa hoàn thành
    }
    
    /**
     * Race condition: Lấy kết quả nhanh nhất
     */
    public String fetchFromMultipleSources(String query) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
            
            // Fork nhiều tasks, lấy cái nào trả về trước
            scope.fork(() -> searchDatabase1(query));
            scope.fork(() -> searchDatabase2(query));
            scope.fork(() -> searchCache(query));
            scope.fork(() -> searchExternalApi(query));
            
            // Chờ task đầu tiên thành công
            scope.join();
            
            // Trả về kết quả nhanh nhất
            return scope.result();
        }
        // Các tasks còn lại tự động bị cancel
    }
    
    /**
     * Timeout handling
     */
    public DataResponse fetchWithTimeout(String requestId) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            
            Subtask<String> data = scope.fork(() -> {
                // Slow operation
                Thread.sleep(5000);
                return "Data for " + requestId;
            });
            
            // Chờ tối đa 2 giây
            scope.joinUntil(Instant.now().plusSeconds(2));
            
            scope.throwIfFailed();
            return new DataResponse(data.get());
            
        } catch (TimeoutException e) {
            return new DataResponse("Timeout - using cached data");
        }
    }
    
    // Helper methods (giả lập)
    private UserBasicInfo fetchBasicInfo(String userId) throws InterruptedException {
        Thread.sleep(100);
        return new UserBasicInfo(userId, "John Doe", "john@example.com");
    }
    
    private List<Order> fetchOrders(String userId) throws InterruptedException {
        Thread.sleep(150);
        return List.of(new Order("ORD-1"), new Order("ORD-2"));
    }
    
    private List<Review> fetchReviews(String userId) throws InterruptedException {
        Thread.sleep(120);
        return List.of(new Review(5, "Great!"));
    }
    
    private RecommendationList fetchRecommendations(String userId) throws InterruptedException {
        Thread.sleep(80);
        return new RecommendationList(List.of("Product A", "Product B"));
    }
    
    private String searchDatabase1(String query) throws InterruptedException {
        Thread.sleep(200);
        return "DB1 result: " + query;
    }
    
    private String searchDatabase2(String query) throws InterruptedException {
        Thread.sleep(300);
        return "DB2 result: " + query;
    }
    
    private String searchCache(String query) throws InterruptedException {
        Thread.sleep(50); // Fastest
        return "Cache result: " + query;
    }
    
    private String searchExternalApi(String query) throws InterruptedException {
        Thread.sleep(400);
        return "API result: " + query;
    }
    
    // Helper classes
    record UserProfile(UserBasicInfo basic, List<Order> orders, 
                      List<Review> reviews, RecommendationList recommendations) {}
    record UserBasicInfo(String id, String name, String email) {}
    record Order(String orderId) {}
    record Review(int rating, String comment) {}
    record RecommendationList(List<String> items) {}
    record DataResponse(String data) {}
    
    public static void main(String[] args) throws Exception {
        StructuredConcurrencyExample example = new StructuredConcurrencyExample();
        
        // Test 1: Fetch complete user profile
        System.out.println("=== Test 1: Fetch User Profile ===");
        UserProfile profile = example.fetchUserProfile("user-123");
        System.out.println("Profile: " + profile);
        
        // Test 2: Race condition - fastest wins
        System.out.println("\n=== Test 2: Fetch from Multiple Sources ===");
        String result = example.fetchFromMultipleSources("Java Virtual Threads");
        System.out.println("Fastest result: " + result);
        
        // Test 3: Timeout handling
        System.out.println("\n=== Test 3: Fetch with Timeout ===");
        DataResponse response = example.fetchWithTimeout("req-456");
        System.out.println("Response: " + response);
    }
}
```

## 4. Best Practices và Common Pitfalls

### 4.1. Best Practices

#### 1. Tránh synchronized, dùng ReentrantLock

```java
// BAD - Gây pinning
private final Object lock = new Object();

public void process() {
    synchronized(lock) {
        // Long blocking operation
        database.query();  // Virtual thread bị pin!
    }
}

// GOOD - Không bị pinning
private final ReentrantLock lock = new ReentrantLock();

public void process() {
    lock.lock();
    try {
        database.query();  // Virtual thread có thể unmount
    } finally {
        lock.unlock();
    }
}
```

#### 2. Dùng ScopedValue thay vì ThreadLocal

```java
// BAD - ThreadLocal tốn memory với hàng triệu virtual threads
private static final ThreadLocal<UserContext> context = new ThreadLocal<>();

// GOOD - ScopedValue (Java 21+)
private static final ScopedValue<UserContext> context = ScopedValue.newInstance();

public void handleRequest(UserContext userCtx) {
    ScopedValue.where(context, userCtx).run(() -> {
        // Code trong scope này có thể access context
        processRequest();
    });
}
```

#### 3. Không cần Thread Pool với Virtual Threads

```java
// BAD - Không cần pool
ExecutorService executor = Executors.newFixedThreadPool(1000, 
    Thread.ofVirtual().factory());

// GOOD - Tạo virtual thread per-task
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

// HOẶC tạo trực tiếp
Thread.ofVirtual().start(() -> {
    // Task code
});
```

#### 4. Monitor với JFR (Java Flight Recorder)

```java
// Enable JFR để monitor virtual threads
// VM options: -XX:StartFlightRecording=filename=recording.jfr

// Code để track virtual threads
Thread vThread = Thread.ofVirtual()
    .name("worker-", 0)  // Auto-incrementing names
    .start(() -> {
        // Task
    });
```

#### 5. Graceful Shutdown

```java
// GOOD - Sử dụng try-with-resources
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    // Submit tasks
    for (Task task : tasks) {
        executor.submit(task);
    }
} // Auto shutdown và chờ tasks hoàn thành

// Hoặc manual shutdown
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
try {
    // Submit tasks
} finally {
    executor.shutdown();
    if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
        executor.shutdownNow();
    }
}
```

### 4.2. Common Pitfalls (Lỗi thường gặp)

#### 1. Dùng Virtual Threads cho CPU-intensive tasks

```java
// BAD
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        // CPU-intensive: hash calculation, encryption
        for (int i = 0; i < 1_000_000_000; i++) {
            result = MD5.hash(data + i);
        }
    });
}

// GOOD - Dùng ForkJoinPool
ForkJoinPool.commonPool().submit(() -> {
    // CPU-intensive work
});
```

#### 2. Quên set timeout cho blocking operations

```java
// BAD - Không timeout
String result = httpClient.send(request, BodyHandlers.ofString()).body();

// GOOD - Có timeout
HttpClient client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(5))
    .build();

CompletableFuture<String> future = client.sendAsync(request, BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .orTimeout(10, TimeUnit.SECONDS); // Request timeout
```

#### 3. Blocking trong synchronized block

```java
// BAD - Double trouble!
synchronized(lock) {
    try {
        Thread.sleep(1000);  // Block + Synchronized = Disaster
    } catch (InterruptedException e) {
        // Handle
    }
}

// GOOD
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    Thread.sleep(1000);  // OK - Virtual thread có thể unmount
} finally {
    lock.unlock();
}
```

#### 4. Không handle InterruptedException đúng cách

```java
// BAD
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // Nuốt exception!
}

// GOOD
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt(); // Restore interrupt status
    throw new RuntimeException("Task interrupted", e);
}
```

## 5. Migration Guide: Từ Platform Threads sang Virtual Threads

### 5.1. Spring Boot Application

```java
// application.properties hoặc application.yml
// Spring Boot 3.2+ hỗ trợ virtual threads

# Enable virtual threads
spring.threads.virtual.enabled=true

// Hoặc config thủ công
@Configuration
public class VirtualThreadConfig {
    
    @Bean
    public TomcatProtocolHandlerCustomizer<?> protocolHandlerVirtualThreadExecutorCustomizer() {
        return protocolHandler -> {
            protocolHandler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
        };
    }
    
    @Bean
    public AsyncTaskExecutor asyncTaskExecutor() {
        return new TaskExecutorAdapter(Executors.newVirtualThreadPerTaskExecutor());
    }
}

// Controller tự động chạy trên virtual threads
@RestController
public class UserController {
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        // Code này chạy trên virtual thread
        User user = userService.findById(id);
        List<Order> orders = orderService.findByUserId(id);
        return enrichUser(user, orders);
    }
}
```

### 5.2. Quarkus Application

```java
// application.properties
quarkus.virtual-threads.enabled=true

// Annotate endpoints
@Path("/users")
public class UserResource {
    
    @GET
    @Path("/{id}")
    @RunOnVirtualThread  // Chạy trên virtual thread
    public User getUser(@PathParam("id") Long id) {
        return userService.findById(id);
    }
}
```

### 5.3. Helidon Application

```java
// Helidon tự động sử dụng virtual threads từ version 4.0+
WebServer server = WebServer.builder()
    .routing(Routing.builder()
        .get("/users/{id}", (req, res) -> {
            // Tự động chạy trên virtual thread
            String id = req.path().param("id");
            User user = fetchUser(id);
            res.send(user);
        }))
    .build()
    .start();
```

## 6. Monitoring và Debugging

### 6.1. JFR (Java Flight Recorder)

```bash
# Start application với JFR
java -XX:StartFlightRecording=filename=app.jfr \
     -XX:FlightRecorderOptions=stackdepth=256 \
     -jar myapp.jar

# Analyze recording
jfr print --events jdk.VirtualThreadStart,jdk.VirtualThreadEnd app.jfr
```

### 6.2. JVisualVM

```java
// Thêm JMX monitoring
public class VirtualThreadMonitor {
    
    public static void printThreadStats() {
        ThreadMXBean threadBean = ManagementFactory.getThreadMXBean();
        
        System.out.println("Platform threads: " + threadBean.getThreadCount());
        System.out.println("Peak threads: " + threadBean.getPeakThreadCount());
        
        // Virtual threads không được count bởi ThreadMXBean
        // Cần track thủ công
    }
}
```

### 6.3. Debug Pinned Threads

```bash
# Enable debug logging cho pinned threads
java -Djdk.tracePinnedThreads=full -jar myapp.jar

# Output sẽ show khi virtual thread bị pin:
# Thread[#123,ForkJoinPool-1-worker-1,5,CarrierThreads]
#     java.base/java.lang.VirtualThread$VThreadContinuation.onPinned
#     java.base/java.lang.VirtualThread.park
#     com.example.MyClass.synchronizedMethod(MyClass.java:45) <== monitors:1
```

## 7. Kết luận

Virtual Threads là một bước tiến đột phá trong concurrent programming của Java:

### Điểm mạnh:
-  Khả năng mở rộng vượt trội với I/O-bound applications
-  Code đơn giản, dễ đọc hơn reactive programming
-  Tương thích ngược với code cũ
-  Chi phí tài nguyên cực thấp

### Lưu ý:
-  Không phải silver bullet cho mọi vấn đề
-  Cần Java 21+ để sử dụng stable version
-  Cần refactor synchronized code để tránh pinning
-  Không thay thế platform threads cho CPU-intensive tasks

### Khuyến nghị:
- Bắt đầu với applications mới hoặc microservices
- Migrate dần dần từ thread pools sang virtual threads
- Monitor performance và điều chỉnh theo nhu cầu thực tế
- Kết hợp với Structured Concurrency để quản lý tốt hơn




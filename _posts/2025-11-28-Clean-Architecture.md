---
title: "TÌm hiểu về Kiến trúc Clean Architecture"
date: 2025-11-28 01:17:00  +0700
categories: [architecture, clean]
tags: [architecture]
---

# Kiến trúc Clean Architecture và Ứng dụng Thực Tế 

---

## 1. Giới thiệu

**Clean Architecture** là một kiến trúc phần mềm được đề xuất bởi **Robert C. Martin (Uncle Bob)** vào năm 2012, nhằm giải quyết các vấn đề về:
- **Khả năng bảo trì** (Maintainability)
- **Khả năng mở rộng** (Scalability)
- **Khả năng kiểm thử** (Testability)
- **Độc lập với công nghệ** (Technology Independence)

### Mục tiêu chính:
> **Tạo ra hệ thống phần mềm có khả năng thay đổi linh hoạt mà không ảnh hưởng đến nghiệp vụ cốt lõi**

---

## 2. Lịch sử và lý do ra đời

### 2.1. Bối cảnh lịch sử

Trước khi Clean Architecture ra đời, ngành phần mềm đã trải qua nhiều mô hình kiến trúc:

**Timeline:**
```
1970s - 1980s: Monolithic Architecture (Kiến trúc nguyên khối)
    ↓
1990s: Layered Architecture (Kiến trúc phân tầng 3-tier)
    ↓
2000s: MVC, MVP, MVVM (Tách biệt UI và Logic)
    ↓
2005: Hexagonal Architecture (Ports & Adapters) - Alistair Cockburn
    ↓
2008: Onion Architecture - Jeffrey Palermo
    ↓
2012: Clean Architecture - Robert C. Martin
    ↓
Hiện tại: Kết hợp nhiều pattern (Microservices, DDD, CQRS...)
```

### 2.2. Lý do sinh ra Clean Architecture

#### **Lý do 1: Sự phụ thuộc vào Framework**

**Vấn đề thực tế:**
```java
// Code dính chặt vào Spring Framework
@RestController
@RequestMapping("/api/hoadon")
public class HoaDonController {
    @Autowired
    private HoaDonService service;
    
    @GetMapping("/{id}")
    public ResponseEntity<HoaDon> getHoaDon(@PathVariable Long id) {
        // Logic nghiệp vụ dính chặt với Spring annotations
        return ResponseEntity.ok(service.findById(id));
    }
}
```

**Hậu quả:**
- Muốn chuyển từ Spring sang Jakarta EE? → Viết lại toàn bộ!
- Framework cập nhật breaking changes? → Phải sửa code khắp nơi!
- Test phải chạy cả Spring container → Chậm và phức tạp!

**Giải pháp Clean Architecture:**
```java
// Business logic hoàn toàn độc lập
public class GetHoaDonUseCase {
    private final HoaDonRepository repository;
    
    public GetHoaDonUseCase(HoaDonRepository repository) {
        this.repository = repository;
    }
    
    public HoaDon execute(Long id) {
        return repository.findById(id)
            .orElseThrow(() -> new HoaDonNotFoundException(id));
    }
}

// Adapter cho Spring (có thể thay thế bất cứ lúc nào)
@RestController
public class HoaDonSpringController {
    private final GetHoaDonUseCase useCase;
    
    @GetMapping("/api/hoadon/{id}")
    public ResponseEntity<HoaDon> getHoaDon(@PathVariable Long id) {
        return ResponseEntity.ok(useCase.execute(id));
    }
}
```

---

#### **Lý do 2: Thay đổi Database gây "thảm họa"**

**Câu chuyện thực tế:**

Một công ty bắt đầu với MySQL, sau 2 năm:
- **Vấn đề 1:** Dữ liệu văn bản tiếng Việt cần full-text search → Cần Elasticsearch
- **Vấn đề 2:** Dữ liệu log khổng lồ → Cần MongoDB
- **Vấn đề 3:** Real-time chat → Cần Redis
- **Vấn đề 4:** Data warehouse → Cần PostgreSQL với TimescaleDB

**Code kiểu cũ (Disaster!):**
```java
public class HoaDonService {
    public List<HoaDon> thongKeTheoThang(int thang) {
        // SQL MySQL dính chặt trong code
        String sql = "SELECT * FROM HoaDon " +
                     "WHERE MONTH(ngayLap) = ? " +
                     "AND YEAR(ngayLap) = YEAR(CURDATE())";
        
        try (Connection conn = DriverManager.getConnection("jdbc:mysql://...")) {
            PreparedStatement ps = conn.prepareStatement(sql);
            ps.setInt(1, thang);
            // ... xử lý ResultSet
        }
    }
    
    public void luuHoaDon(HoaDon hd) {
        String sql = "INSERT INTO HoaDon (ma, ngay, tong) VALUES (?, ?, ?)";
        // ... MySQL specific code
    }
    
    // 50 methods khác cũng dính MySQL...
}
```

**Muốn chuyển sang PostgreSQL?**
- Phải sửa hàng trăm methods!
- SQL syntax khác nhau (CURDATE() vs NOW())
- Connection string khác
- Driver khác
- Có thể có bug không lường trước

**Giải pháp Clean Architecture:**
```java
// 1. Domain Layer - Hoàn toàn độc lập
public class HoaDon {
    private String maHoaDon;
    private LocalDate ngayLap;
    private double tongTien;
    
    public boolean isTrongThang(int thang, int nam) {
        return ngayLap.getMonthValue() == thang 
            && ngayLap.getYear() == nam;
    }
}

// 2. Use Case Layer - Chỉ phụ thuộc vào interface
public interface HoaDonRepository {
    List<HoaDon> findByThangNam(int thang, int nam);
    void save(HoaDon hoaDon);
    Optional<HoaDon> findById(Long id);
}

public class ThongKeDoanhThuUseCase {
    private final HoaDonRepository repository;
    
    public ThongKeDoanhThuUseCase(HoaDonRepository repository) {
        this.repository = repository;
    }
    
    public ThongKeResult execute(int thang, int nam) {
        List<HoaDon> danhSach = repository.findByThangNam(thang, nam);
        double tongDoanhThu = danhSach.stream()
            .mapToDouble(HoaDon::getTongTien)
            .sum();
        return new ThongKeResult(danhSach, tongDoanhThu);
    }
}

// 3. Infrastructure Layer - MySQL Implementation
public class HoaDonRepositoryMySQL implements HoaDonRepository {
    @Override
    public List<HoaDon> findByThangNam(int thang, int nam) {
        String sql = "SELECT * FROM HoaDon " +
                     "WHERE MONTH(ngayLap) = ? AND YEAR(ngayLap) = ?";
        // MySQL specific implementation
    }
}

// 4. Muốn đổi sang PostgreSQL? Chỉ cần tạo class mới!
public class HoaDonRepositoryPostgreSQL implements HoaDonRepository {
    @Override
    public List<HoaDon> findByThangNam(int thang, int nam) {
        String sql = "SELECT * FROM HoaDon " +
                     "WHERE EXTRACT(MONTH FROM ngayLap) = ? " +
                     "AND EXTRACT(YEAR FROM ngayLap) = ?";
        // PostgreSQL specific implementation
    }
}

// 5. Muốn dùng MongoDB? Tạo thêm một class!
public class HoaDonRepositoryMongoDB implements HoaDonRepository {
    @Override
    public List<HoaDon> findByThangNam(int thang, int nam) {
        Query query = new Query();
        query.addCriteria(Criteria.where("ngayLap.month").is(thang)
                                 .and("ngayLap.year").is(nam));
        // MongoDB specific implementation
    }
}

// Use Case KHÔNG CẦN SỬA GÌ CẢ!
```

**Kết quả:**
-  Đổi database chỉ cần swap implementation
-  Có thể dùng NHIỀU database cùng lúc (MySQL + MongoDB + Redis)
-  Test use case với fake repository (không cần database thật)
-  Business logic không hề bị ảnh hưởng

---

#### **Lý do 3: Thay đổi UI làm "sập" toàn bộ hệ thống**

**Tình huống thực tế:**

Một ứng dụng desktop (Swing) cần chuyển sang:
- Web application (React/Angular)
- Mobile app (Android/iOS)
- REST API cho đối tác
- CLI tool cho admin

**Code kiểu cũ:**
```java
public class QuanLyHoaDonUI extends JFrame {
    private JTable table;
    private JTextField txtMaHD, txtTongTien;
    
    private void btnThemActionPerformed() {
        // Business logic dính chặt với Swing UI
        String ma = txtMaHD.getText();
        if (ma.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Vui lòng nhập mã!");
            return;
        }
        
        // Validation logic trong UI
        if (ma.length() < 5) {
            JOptionPane.showMessageDialog(this, "Mã phải >= 5 ký tự!");
            return;
        }
        
        // Database access trong UI
        try {
            Connection conn = DriverManager.getConnection("...");
            String sql = "INSERT INTO HoaDon VALUES (?, ?, ?)";
            PreparedStatement ps = conn.prepareStatement(sql);
            ps.setString(1, ma);
            // ...
            ps.executeUpdate();
            
            JOptionPane.showMessageDialog(this, "Thêm thành công!");
            loadData(); // Refresh table
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, "Lỗi: " + e.getMessage());
        }
    }
}
```

**Vấn đề:**
- Muốn làm Web app? → Toàn bộ logic phải viết lại!
- Muốn làm REST API? → Copy-paste logic ra, rồi sửa lại!
- Validation bị lặp ở nhiều nơi
- Không test được logic mà không chạy UI

**Giải pháp Clean Architecture:**
```java
// 1. Domain - Business Rules
public class HoaDon {
    private String maHoaDon;
    private LocalDate ngayLap;
    private double tongTien;
    
    public HoaDon(String maHoaDon, LocalDate ngayLap, double tongTien) {
        if (maHoaDon == null || maHoaDon.length() < 5) {
            throw new IllegalArgumentException("Mã hóa đơn phải >= 5 ký tự");
        }
        if (tongTien < 0) {
            throw new IllegalArgumentException("Tổng tiền không thể âm");
        }
        this.maHoaDon = maHoaDon;
        this.ngayLap = ngayLap;
        this.tongTien = tongTien;
    }
}

// 2. Use Case - Application Logic
public class TaoHoaDonUseCase {
    private final HoaDonRepository repository;
    
    public HoaDonResult execute(TaoHoaDonRequest request) {
        // Business validation
        if (repository.existsByMa(request.getMaHoaDon())) {
            throw new DuplicateHoaDonException("Mã hóa đơn đã tồn tại");
        }
        
        // Create entity
        HoaDon hoaDon = new HoaDon(
            request.getMaHoaDon(),
            request.getNgayLap(),
            request.getTongTien()
        );
        
        // Save
        repository.save(hoaDon);
        
        return HoaDonResult.success(hoaDon);
    }
}

// 3. Adapters - Multiple UIs share same Use Case

// 3a. Swing UI
public class HoaDonSwingController {
    private final TaoHoaDonUseCase useCase;
    private HoaDonView view;
    
    public void handleThemHoaDon() {
        try {
            TaoHoaDonRequest request = new TaoHoaDonRequest(
                view.getMaHoaDon(),
                view.getNgayLap(),
                view.getTongTien()
            );
            
            HoaDonResult result = useCase.execute(request);
            view.showSuccess("Thêm thành công!");
            view.refresh();
            
        } catch (IllegalArgumentException e) {
            view.showError(e.getMessage());
        } catch (DuplicateHoaDonException e) {
            view.showWarning(e.getMessage());
        }
    }
}

// 3b. REST API (cùng Use Case!)
@RestController
@RequestMapping("/api/hoadon")
public class HoaDonRestController {
    private final TaoHoaDonUseCase useCase;
    
    @PostMapping
    public ResponseEntity<?> taoHoaDon(@RequestBody TaoHoaDonRequest request) {
        try {
            HoaDonResult result = useCase.execute(request);
            return ResponseEntity.ok(result);
        } catch (IllegalArgumentException e) {
            return ResponseEntity.badRequest().body(e.getMessage());
        } catch (DuplicateHoaDonException e) {
            return ResponseEntity.status(409).body(e.getMessage());
        }
    }
}

// 3c. JavaFX UI (cùng Use Case!)
public class HoaDonFXController {
    private final TaoHoaDonUseCase useCase;
    
    @FXML
    private void handleThem() {
        try {
            TaoHoaDonRequest request = new TaoHoaDonRequest(
                txtMaHD.getText(),
                dpNgayLap.getValue(),
                Double.parseDouble(txtTongTien.getText())
            );
            
            useCase.execute(request);
            showAlert(AlertType.INFORMATION, "Thành công!");
        } catch (Exception e) {
            showAlert(AlertType.ERROR, e.getMessage());
        }
    }
}

// 3d. CLI Tool (cùng Use Case!)
public class HoaDonCLI {
    private final TaoHoaDonUseCase useCase;
    
    public void run(String[] args) {
        TaoHoaDonRequest request = new TaoHoaDonRequest(
            args[0], // ma
            LocalDate.parse(args[1]), // ngay
            Double.parseDouble(args[2]) // tong
        );
        
        try {
            useCase.execute(request);
            System.out.println("✓ Tạo hóa đơn thành công!");
        } catch (Exception e) {
            System.err.println("✗ Lỗi: " + e.getMessage());
        }
    }
}
```

**Kết quả:**
-  1 Use Case → Dùng cho Swing, Web, Mobile, CLI, API
-  Logic validation chỉ viết 1 lần
-  Thay đổi UI không ảnh hưởng business logic
-  Test use case dễ dàng (không cần UI)

---

#### **Lý do 4: Không thể Test được**

**Vấn đề:**
```java
public class HoaDonService {
    public double tinhTongDoanhThu(int thang) {
        // Dính database → Không test được nếu không có DB
        Connection conn = DatabaseConnection.getInstance();
        
        // Dính UI → Không test được nếu không có giao diện
        if (LoginUI.currentUser == null) {
            JOptionPane.showMessageDialog(null, "Chưa đăng nhập!");
            return 0;
        }
        
        // Dính thời gian hệ thống → Test không ổn định
        if (LocalDate.now().getMonthValue() != thang) {
            // Logic đặc biệt
        }
        
        // Business logic lẫn lộn
        // ...
    }
}
```

**Muốn test?**
-  Phải setup database
-  Phải có UI
-  Kết quả test thay đổi theo tháng hiện tại
-  Chạy chậm (database + UI)

**Giải pháp Clean Architecture:**
```java
// Use Case thuần túy, dễ test
public class TinhTongDoanhThuUseCase {
    private final HoaDonRepository repository;
    private final Clock clock; // Dependency injection cho thời gian
    
    public TinhTongDoanhThuUseCase(HoaDonRepository repository, Clock clock) {
        this.repository = repository;
        this.clock = clock;
    }
    
    public double execute(int thang, int nam) {
        List<HoaDon> danhSach = repository.findByThangNam(thang, nam);
        return danhSach.stream()
            .mapToDouble(HoaDon::getTongTien)
            .sum();
    }
}

// Test đơn giản, nhanh, ổn định
public class TinhTongDoanhThuUseCaseTest {
    @Test
    public void testTinhTong_CoHoaDon() {
        // Fake repository (không cần database thật)
        HoaDonRepository fakeRepo = new FakeHoaDonRepository(
            List.of(
                new HoaDon("HD001", LocalDate.of(2024, 12, 1), 1000000),
                new HoaDon("HD002", LocalDate.of(2024, 12, 15), 2000000)
            )
        );
        
        // Fake clock (thời gian cố định)
        Clock fixedClock = Clock.fixed(
            Instant.parse("2024-12-20T10:00:00Z"), 
            ZoneId.systemDefault()
        );
        
        TinhTongDoanhThuUseCase useCase = new TinhTongDoanhThuUseCase(fakeRepo, fixedClock);
        
        // Act
        double result = useCase.execute(12, 2024);
        
        // Assert
        assertEquals(3000000, result, 0.01);
    }
    
    @Test
    public void testTinhTong_KhongCoHoaDon() {
        HoaDonRepository emptyRepo = new FakeHoaDonRepository(List.of());
        Clock clock = Clock.systemDefaultZone();
        
        TinhTongDoanhThuUseCase useCase = new TinhTongDoanhThuUseCase(emptyRepo, clock);
        
        double result = useCase.execute(12, 2024);
        
        assertEquals(0, result, 0.01);
    }
}
```

**Kết quả:**
-  Test nhanh (không cần database, UI)
-  Test ổn định (không phụ thuộc thời gian thật)
-  Test độc lập (mỗi test tự setup data)
-  Coverage cao (dễ test edge cases)

---

#### **Lý do 5: Làm việc nhóm khó khăn**

**Tình huống:**

Team 5 người phát triển 1 feature:
- **Dev A:** Làm UI (Swing/JavaFX)
- **Dev B:** Làm Business Logic
- **Dev C:** Làm Database
- **Dev D:** Làm API
- **Dev E:** Làm Test

**Vấn đề với code không chia layer:**
```java
// File duy nhất: QuanLyHoaDon.java (2000 dòng)
public class QuanLyHoaDon extends JFrame {
    // UI components
    private JTable table;
    private JButton btnThem, btnXoa;
    
    // Database
    private Connection conn;
    
    // Business logic
    public void themHoaDon() { ... }
    public void xoaHoaDon() { ... }
    public double tinhTong() { ... }
    
    // Validation
    private boolean validate() { ... }
    
    // ...
}
```

**Hậu quả:**
-  5 người cùng sửa 1 file → Git conflict liên tục!
-  Dev A sửa UI → Breaking code của Dev B
-  Dev C thay đổi database → Dev D phải đợi
-  Code review khó (1 file quá dài)
-  Merge hell!

**Giải pháp Clean Architecture:**

```
project/
├── domain/                      # Dev B làm (Business Logic)
│   └── HoaDon.java
│
├── application/                 # Dev B làm (Use Cases)
│   ├── TaoHoaDonUseCase.java
│   ├── XoaHoaDonUseCase.java
│   └── ports/
│       └── HoaDonRepository.java  # Interface
│
├── adapters/
│   ├── ui/                      # Dev A làm (UI)
│   │   ├── swing/
│   │   │   └── HoaDonSwingView.java
│   │   └── controllers/
│   │       └── HoaDonController.java
│   │
│   ├── persistence/             # Dev C làm (Database)
│   │   └── HoaDonRepositoryMySQL.java
│   │
│   └── api/                     # Dev D làm (REST API)
│       └── HoaDonRestController.java
│
└── tests/                       # Dev E làm (Tests)
    └── TaoHoaDonUseCaseTest.java
```

**Kết quả:**
-  Mỗi người làm riêng folder → Ít conflict
-  Dev A sửa UI không ảnh hưởng Dev C (database)
-  Dev làm song song (không chờ đợi)
-  Code review dễ (mỗi PR nhỏ, rõ ràng)
-  Git workflow mượt mà

---

### 2.3. Tóm tắt lý do sinh ra Clean Architecture

| Vấn đề | Hậu quả | Giải pháp Clean Architecture |
|--------|---------|------------------------------|
| **Dính Framework** | Thay framework = Viết lại toàn bộ | Business logic độc lập |
| **Dính Database** | Đổi DB = Sửa hàng trăm chỗ | Repository pattern + Interface |
| **Dính UI** | Đổi UI = Viết lại logic | Use Case tách biệt UI |
| **Không test được** | Chạy test chậm, không ổn định | Dependency Injection, Mock |
| **Làm nhóm khó** | Git conflict, chờ đợi lẫn nhau | Chia layer rõ ràng |

---

## 3. Vấn đề của kiến trúc truyền thống

### 3.1. Monolithic Architecture (Kiến trúc nguyên khối)

```java
// Tất cả trong 1 class
public class Application {
    public static void main(String[] args) {
        // UI
        JFrame frame = new JFrame();
        JButton btn = new JButton("Click");
        
        btn.addActionListener(e -> {
            // Business Logic
            double total = 0;
            
            // Database
            try (Connection conn = DriverManager.getConnection("...")) {
                ResultSet rs = conn.createStatement()
                    .executeQuery("SELECT * FROM orders");
                while (rs.next()) {
                    total += rs.getDouble("amount");
                }
            }
            
            // UI
            JOptionPane.showMessageDialog(frame, "Total: " + total);
        });
        
        frame.add(btn);
        frame.setVisible(true);
    }
}
```

**Vấn đề:**
-  Không tái sử dụng được
-  Không test được
-  Khó bảo trì
-  Không làm việc nhóm được

### 3.2. Layered Architecture (3-tier)

```
┌─────────────────┐
│  Presentation   │  (UI Layer)
├─────────────────┤
│  Business Logic │  (Service Layer)
├─────────────────┤
│  Data Access    │  (DAO/Repository Layer)
└─────────────────┘
```

**Vấn đề:**
-  Business Logic vẫn phụ thuộc vào Database
-  Thay đổi database → Phải sửa Business Logic
-  Dependency đi từ trên xuống (UI → BL → DB)

**Ví dụ vấn đề:**
```java
// Business Layer
public class OrderService {
    private OrderDAO orderDAO; // Phụ thuộc vào DAO cụ thể
    
    public double calculateTotal() {
        List<Order> orders = orderDAO.findAll(); // Phải biết DAO
        return orders.stream()
            .mapToDouble(Order::getAmount)
            .sum();
    }
}

// Data Layer
public class OrderDAO {
    public List<Order> findAll() {
        // MySQL specific code
        String sql = "SELECT * FROM orders";
        // ...
    }
}
```

**Nếu đổi từ MySQL sang MongoDB:**
- OrderDAO thay đổi → OrderService cũng bị ảnh hưởng!
- Phải sửa code ở nhiều nơi
- Có thể gây bug

### 3.3. So sánh tổng quan

| Đặc điểm | Monolithic | Layered 3-tier | Clean Architecture |
|----------|------------|----------------|-------------------|
| **Tách biệt concerns** |  Không | ⚠️ Một phần |  Hoàn toàn |
| **Thay đổi Database** |  Rất khó | ⚠️ Khó |  Dễ dàng |
| **Thay đổi UI** |  Rất khó | ⚠️ Khó |  Dễ dàng |
| **Testability** |  Không test được | ⚠️ Khó test |  Dễ test |
| **Độc lập Framework** |  Không |  Không |  Có |
| **Làm việc nhóm** |  Rất khó | ⚠️ Khó |  Dễ dàng |
| **Độ phức tạp** |  Đơn giản | ⚠️ Vừa phải |  Phức tạp |

---

## 4. Nguyên tắc cốt lõi

### 4.1. SOLID Principles

Clean Architecture dựa trên 5 nguyên tắc SOLID:

#### **S - Single Responsibility Principle (Nguyên tắc Đơn Trách Nhiệm)**

> Một class chỉ nên có một lý do để thay đổi.

** Sai: Một class làm nhiều việc**
```java
public class HoaDon {
    private String ma;
    private double tong;
    
    // Lý do 1: Business logic thay đổi
    public double tinhThue() {
        return tong * 0.1;
    }
    
    // Lý do 2: Database thay đổi
    public void saveToDatabase() {
        Connection conn = DriverManager.getConnection("...");
        // Save logic
    }
    
    // Lý do 3: UI thay đổi
    public void displayOnUI() {
        JOptionPane.showMessageDialog(null, "Hóa đơn: " + ma);
    }
    
    // Lý do 4: Export format thay đổi
    public void exportToPDF() {
        // PDF export logic
    }
}
```

** Đúng: Mỗi class một trách nhiệm**
```java
// Chỉ chứa business logic
public class HoaDon {
    private String ma;
    private double tong;
    
    public double tinhThue() {
        return tong * 0.1;
    }
    
    public double getTongTien() {
        return tong + tinhThue();
    }
}

// Chỉ chịu trách nhiệm lưu database
public class HoaDonRepository {
    public void save(HoaDon hd) {
        // Database logic
    }
}

// Chỉ chịu trách nhiệm hiển thị UI
public class HoaDonPresenter {
    public void display(HoaDon hd) {
        // UI logic
    }
}

// Chỉ chịu trách nhiệm export
public class HoaDonPDFExporter {
    public void export(HoaDon hd) {
        // PDF export logic
    }
}
```

---

#### **O - Open/Closed Principle (Nguyên tắc Đóng/Mở)**

> Mở cho mở rộng, đóng cho sửa đổi.

** Sai: Phải sửa code khi thêm database mới**
```java
public class HoaDonService {
    private String databaseType;
    
    public void save(HoaDon hd) {
        if (databaseType.equals("MySQL")) {
            // MySQL logic
        } else if (databaseType.equals("PostgreSQL")) {
            // PostgreSQL logic
        } else if (databaseType.equals("MongoDB")) {
            // MongoDB logic
        }
        // Thêm database mới → Phải sửa code này!
    }
}
```

** Đúng: Thêm database mới không cần sửa Use Case**
```java
// Interface
public interface HoaDonRepository {
    void save(HoaDon hd);
    List<HoaDon> findAll();
}

// MySQL implementation
public class HoaDonRepositoryMySQL implements HoaDonRepository {
    public void save(HoaDon hd) {
        // MySQL specific code
    }
    
    public List<HoaDon> findAll() {
        // MySQL specific code
    }
}

// MongoDB implementation
public class HoaDonRepositoryMongoDB implements HoaDonRepository {
    public void save(HoaDon hd) {
        // MongoDB specific code
    }
    
    public List<HoaDon> findAll() {
        // MongoDB specific code
    }
}

// Use Case không cần sửa khi thêm database mới
public class TaoHoaDonUseCase {
    private HoaDonRepository repository; // Interface
    
    public void execute(HoaDon hd) {
        repository.save(hd); // Không quan tâm implementation
    }
}
```

---

#### **L - Liskov Substitution Principle (Nguyên tắc Thay Thế Liskov)**

> Subclass phải có thể thay thế được Superclass mà không làm thay đổi tính đúng đắn của chương trình.

```java
public interface PaymentMethod {
    boolean pay(double amount);
}

public class CreditCardPayment implements PaymentMethod {
    public boolean pay(double amount) {
        // Process credit card payment
        return true;
    }
}

public class CashPayment implements PaymentMethod {
    public boolean pay(double amount) {
        // Process cash payment
        return true;
    }
}

public class MoMoPayment implements PaymentMethod {
    public boolean pay(double amount) {
        // Process MoMo payment
        return true;
    }
}

// Use Case không cần biết payment method cụ thể
public class ThanhToanUseCase {
    private PaymentMethod paymentMethod;
    
    public ThanhToanUseCase(PaymentMethod paymentMethod) {
        this.paymentMethod = paymentMethod;
    }
    
    public void execute(double amount) {
        boolean success = paymentMethod.pay(amount);
        // Hoạt động với mọi payment method!
    }
}
```

---

#### **I - Interface Segregation Principle (Nguyên tắc Phân Tách Interface)**

> Không ép client phụ thuộc vào interface mà nó không sử dụng.

** Sai: Interface quá lớn**
```java
public interface Repository {
    void save();
    void update();
    void delete();
    List<Object> findAll();
    Optional<Object> findById(Long id);
    List<Object> search(String keyword);
    void exportToExcel();
    void importFromCSV();
    void backup();
    void restore();
}

// Client chỉ cần đọc, nhưng phải implement tất cả!
public class ReadOnlyRepository implements Repository {
    public List<Object> findAll() { /* OK */ }
    public Optional<Object> findById(Long id) { /* OK */ }
    
    // Phải implement những method không cần!
    public void save() { throw new UnsupportedOperationException(); }
    public void update() { throw new UnsupportedOperationException(); }
    public void delete() { throw new UnsupportedOperationException(); }
    public void exportToExcel() { throw new UnsupportedOperationException(); }
    public void importFromCSV() { throw new UnsupportedOperationException(); }
    public void backup() { throw new UnsupportedOperationException(); }
    public void restore() { throw new UnsupportedOperationException(); }
    public List<Object> search(String keyword) { throw new UnsupportedOperationException(); }
}
```

** Đúng: Tách thành nhiều interface nhỏ**
```java
// Interface nhỏ, tập trung
public interface ReadRepository {
    Optional<Object> findById(Long id);
    List<Object> findAll();
}

public interface WriteRepository {
    void save(Object obj);
    void update(Object obj);
    void delete(Long id);
}

public interface SearchRepository {
    List<Object> search(String keyword);
}

public interface ExportRepository {
    void exportToExcel();
    void importFromCSV();
}

// Client chỉ implement những gì cần
public class ReadOnlyRepository implements ReadRepository {
    public Optional<Object> findById(Long id) { /* Implementation */ }
    public List<Object> findAll() { /* Implementation */ }
}

public class FullRepository implements ReadRepository, WriteRepository, SearchRepository {
    // Implement tất cả
}
```

---

#### **D - Dependency Inversion Principle (Nguyên tắc Đảo Ngược Phụ Thuộc)**

> - High-level modules không nên phụ thuộc vào low-level modules. Cả hai nên phụ thuộc vào abstraction.
> - Abstraction không nên phụ thuộc vào details. Details nên phụ thuộc vào abstraction.

** Sai: Phụ thuộc vào concrete class**
```java
// Use Case phụ thuộc trực tiếp vào MySQL
public class OrderService {
    private MySQLOrderRepository repository = new MySQLOrderRepository();
    
    public List<Order> getAll() {
        return repository.findAll(); // Dính chặt MySQL
    }
}
```

** Đúng: Phụ thuộc vào interface (abstraction)**
```java
// 1. Define abstraction (interface)
public interface OrderRepository {
    List<Order> findAll();
    void save(Order order);
}

// 2. Use Case phụ thuộc vào interface
public class OrderService {
    private OrderRepository repository; // Interface, không phải concrete class
    
    public OrderService(OrderRepository repository) {
        this.repository = repository; // Dependency Injection
    }
    
    public List<Order> getAll() {
        return repository.findAll(); // Không biết implementation cụ thể
    }
}

// 3. Concrete implementation
public class MySQLOrderRepository implements OrderRepository {
    public List<Order> findAll() { /* MySQL code */ }
    public void save(Order order) { /* MySQL code */ }
}

public class MongoDBOrderRepository implements OrderRepository {
    public List<Order> findAll() { /* MongoDB code */ }
    public void save(Order order) { /* MongoDB code */ }
}

// 4. Wiring (trong Main hoặc Config)
public class Main {
    public static void main(String[] args) {
        // Dễ dàng swap implementation
        OrderRepository repo = new MySQLOrderRepository();
        // OrderRepository repo = new MongoDBOrderRepository();
        
        OrderService service = new OrderService(repo);
        service.getAll();
    }
}
```

---

### 4.2. Separation of Concerns (Tách biệt các mối quan tâm)

**Nguyên tắc:** Mỗi phần của hệ thống chỉ quan tâm đến một việc cụ thể.

```
Business Logic  ≠  Database  ≠  UI  ≠  Framework
```

**Ví dụ minh họa:**

```java
//  Domain - CHỈ quan tâm business logic
public class Order {
    private double amount;
    private OrderStatus status;
    
    public void approve() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Cannot approve order");
        }
        this.status = OrderStatus.APPROVED;
    }
    
    public double calculateTotal() {
        return amount * 1.1; // +10% tax
    }
}

//  Application - CHỈ quan tâm use case
public class ApproveOrderUseCase {
    private OrderRepository repository;
    
    public void execute(Long orderId) {
        Order order = repository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
        
        order.approve();
        repository.save(order);
    }
}

//  Infrastructure - CHỈ quan tâm database
public class OrderRepositoryJPA implements OrderRepository {
    private EntityManager em;
    
    public Optional<Order> findById(Long id) {
        return Optional.ofNullable(em.find(Order.class, id));
    }
    
    public void save(Order order) {
        em.persist(order);
    }
}

//  UI - CHỈ quan tâm hiển thị
public class OrderController {
    private ApproveOrderUseCase useCase;
    
    @PostMapping("/orders/{id}/approve")
    public ResponseEntity<?> approve(@PathVariable Long id) {
        useCase.execute(id);
        return ResponseEntity.ok().build();
    }
}
```

---

## 5. Các tầng trong Clean Architecture

### Sơ đồ tổng quan

```
┌─────────────────────────────────────────────────────────┐
│                Frameworks & Drivers (Tầng 4)            │
│         Database | UI | Web | External APIs             │
│                     Ngoài cùng                          │
└──────────────────────┬──────────────────────────────────┘
                       │ Phụ thuộc
┌──────────────────────▼──────────────────────────────────┐
│             Interface Adapters (Tầng 3)                 │
│  Controllers | Presenters | Gateways | Repositories     │
└──────────────────────┬──────────────────────────────────┘
                       │ Phụ thuộc
┌──────────────────────▼──────────────────────────────────┐
│                  Use Cases (Tầng 2)                     │
│           Application Business Rules                    │
└──────────────────────┬──────────────────────────────────┘
                       │ Phụ thuộc
┌──────────────────────▼──────────────────────────────────┐
│                   Entities (Tầng 1)                     │
│            Enterprise Business Rules                    │
│                    Trong cùng                           │
└─────────────────────────────────────────────────────────┘
```

---

### 5.1. Tầng 1: Entities (Domain Layer)

**Đặc điểm:**
- Tầng trong cùng, chứa business logic cốt lõi
- Đại diện cho các khái niệm nghiệp vụ
- **Hoàn toàn độc lập**, không phụ thuộc gì cả
- Ít thay đổi nhất

**Ví dụ:**
```java
package com.example.domain.entities;

public class HoaDon {
    private String maHoaDon;
    private LocalDate ngayLap;
    private double tongTien;
    private TrangThaiHoaDon trangThai;
    private KhachHang khachHang;
    private List<ChiTietHoaDon> chiTiet;
    
    public HoaDon(String maHoaDon, LocalDate ngayLap, KhachHang khachHang, List<ChiTietHoaDon> chiTiet) {
        // Validation - Business Rules
        if (maHoaDon == null || maHoaDon.isBlank()) {
            throw new ValidationException("Mã hóa đơn không được rỗng");
        }
        if (chiTiet == null || chiTiet.isEmpty()) {
            throw new ValidationException("Hóa đơn phải có ít nhất 1 sản phẩm");
        }
        
        this.maHoaDon = maHoaDon;
        this.ngayLap = ngayLap;
        this.khachHang = khachHang;
        this.chiTiet = new ArrayList<>(chiTiet);
        this.trangThai = TrangThaiHoaDon.CHO_THANH_TOAN;
    }
    
    // Business logic methods
    public void huy() {
        if (trangThai == TrangThaiHoaDon.DA_THANH_TOAN) {
            throw new HoaDonException("Không thể hủy hóa đơn đã thanh toán");
        }
        this.trangThai = TrangThaiHoaDon.DA_HUY;
    }
    
    public void thanhToan() {
        if (trangThai == TrangThaiHoaDon.DA_HUY) {
            throw new HoaDonException("Không thể thanh toán hóa đơn đã hủy");
        }
        this.trangThai = TrangThaiHoaDon.DA_THANH_TOAN;
    }
    
    public double tinhTongTien() {
        return chiTiet.stream()
            .mapToDouble(ct -> ct.getSoLuong() * ct.getDonGia())
            .sum();
    }
    
    public boolean isDaQuaHan(LocalDate ngayHienTai) {
        return trangThai == TrangThaiHoaDon.CHO_THANH_TOAN 
            && ngayHienTai.isAfter(ngayLap.plusDays(30));
    }
    
    // Getters (no setters - immutability where possible)
}

public class KhachHang {
    private String maKH;
    private String ten;
    private LoaiKhachHang loai;
    private double tongChiTieu;
    
    public void nangCapLoai() {
        if (tongChiTieu >= 100_000_000) {
            this.loai = LoaiKhachHang.VIP;
        } else if (tongChiTieu >= 50_000_000) {
            this.loai = LoaiKhachHang.THUONG_XUYEN;
        } else {
            this.loai = LoaiKhachHang.BINH_THUONG;
        }
    }
    
    public double getTyLeGiamGia() {
        return switch (loai) {
            case VIP -> 0.15;
            case THUONG_XUYEN -> 0.10;
            case BINH_THUONG -> 0.05;
        };
    }
}

public enum TrangThaiHoaDon {
    CHO_THANH_TOAN,
    DA_THANH_TOAN,
    DA_HUY
}
```

**Lưu ý:**
-  Không import javax.*, org.springframework.*, java.sql.*
-  Chỉ sử dụng Java core (java.time, java.util...)
-  Không biết Database, UI, Framework tồn tại

---

### 5.2. Tầng 2: Use Cases (Application Layer)

**Đặc điểm:**
- Chứa quy tắc nghiệp vụ của ứng dụng cụ thể
- Điều phối luồng dữ liệu giữa Entities và tầng ngoài
- Phụ thuộc vào Entities và Interface (không phụ thuộc concrete class)

**Ví dụ:**
```java
package com.example.application.usecases;

// Input DTO
public record TaoHoaDonRequest(
    String maKhachHang,
    List<ChiTietDTO> chiTiet
) {}

public record ChiTietDTO(
    String maSanPham,
    int soLuong
) {}

// Output DTO
public record TaoHoaDonResponse(
    String maHoaDon,
    double tongTien,
    double giamGia,
    String thongBao
) {}

// Use Case
public class TaoHoaDonUseCase {
    private final HoaDonRepository hoaDonRepository;
    private final KhachHangRepository khachHangRepository;
    private final SanPhamRepository sanPhamRepository;
    private final MaHoaDonGenerator maGenerator;
    
    public TaoHoaDonUseCase(
        HoaDonRepository hoaDonRepository,
        KhachHangRepository khachHangRepository,
        SanPhamRepository sanPhamRepository,
        MaHoaDonGenerator maGenerator
    ) {
        this.hoaDonRepository = hoaDonRepository;
        this.khachHangRepository = khachHangRepository;
        this.sanPhamRepository = sanPhamRepository;
        this.maGenerator = maGenerator;
    }
    
    public TaoHoaDonResponse execute(TaoHoaDonRequest request) {
        // 1. Validate input
        validateRequest(request);
        
        // 2. Lấy khách hàng
        KhachHang khachHang = khachHangRepository.findByMa(request.maKhachHang())
            .orElseThrow(() -> new KhachHangNotFoundException(request.maKhachHang()));
        
        // 3. Tạo chi tiết hóa đơn
        List<ChiTietHoaDon> chiTietList = taoChiTiet(request.chiTiet());
        
        // 4. Tạo hóa đơn
        String maHD = maGenerator.generate();
        HoaDon hoaDon = new HoaDon(maHD, LocalDate.now(), khachHang, chiTietList);
        
        // 5. Áp dụng giảm giá
        double giamGia = hoaDon.tinhTongTien() * khachHang.getTyLeGiamGia();
        hoaDon.apDungGiamGia(giamGia);
        
        // 6. Lưu hóa đơn
        hoaDonRepository.save(hoaDon);
        
        // 7. Cập nhật tồn kho
        capNhatTonKho(chiTietList);
        
        // 8. Cập nhật khách hàng
        khachHang.tangTongChiTieu(hoaDon.getTongTien());
        khachHang.nangCapLoai();
        khachHangRepository.update(khachHang);
        
        return new TaoHoaDonResponse(
            maHD,
            hoaDon.getTongTien(),
            giamGia,
            "Tạo hóa đơn thành công"
        );
    }
    
    private void validateRequest(TaoHoaDonRequest request) {
        if (request.maKhachHang() == null || request.maKhachHang().isBlank()) {
            throw new ValidationException("Mã khách hàng không được rỗng");
        }
        if (request.chiTiet() == null || request.chiTiet().isEmpty()) {
            throw new ValidationException("Phải có ít nhất 1 sản phẩm");
        }
    }
    
    private List<ChiTietHoaDon> taoChiTiet(List<ChiTietDTO> dtoList) {
        return dtoList.stream()
            .map(dto -> {
                SanPham sp = sanPhamRepository.findByMa(dto.maSanPham())
                    .orElseThrow(() -> new SanPhamNotFoundException(dto.maSanPham()));
                
                if (sp.getSoLuongTon() < dto.soLuong()) {
                    throw new KhongDuHangException("Sản phẩm " + sp.getTen() + " không đủ hàng");
                }
                
                return new ChiTietHoaDon(sp, dto.soLuong());
            })
            .toList();
    }
    
    private void capNhatTonKho(List<ChiTietHoaDon> chiTiet) {
        chiTiet.forEach(ct -> {
            SanPham sp = ct.getSanPham();
            sp.giamSoLuongTon(ct.getSoLuong());
            sanPhamRepository.update(sp);
        });
    }
}

// Repository Interfaces (ở tầng Application)
package com.example.application.ports;

public interface HoaDonRepository {
    void save(HoaDon hoaDon);
    Optional<HoaDon> findByMa(String ma);
    List<HoaDon> findByThangNam(int thang, int nam);
    boolean existsByMa(String ma);
}

public interface KhachHangRepository {
    Optional<KhachHang> findByMa(String ma);
    void update(KhachHang khachHang);
}

public interface SanPhamRepository {
    Optional<SanPham> findByMa(String ma);
    void update(SanPham sanPham);
}
```

---

### 5.3. Tầng 3: Interface Adapters

**Đặc điểm:**
- Chuyển đổi dữ liệu giữa Use Cases và tầng ngoài
- Bao gồm: Controllers, Presenters, Repository Implementations

#### Controllers (UI → Use Case)

```java
package com.example.adapters.controllers;

// Swing Controller
public class HoaDonSwingController {
    private final TaoHoaDonUseCase taoHoaDonUseCase;
    private final HoaDonView view;
    
    public HoaDonSwingController(TaoHoaDonUseCase useCase, HoaDonView view) {
        this.taoHoaDonUseCase = useCase;
        this.view = view;
        setupHandlers();
    }
    
    private void setupHandlers() {
        view.onTaoHoaDon(this::handleTaoHoaDon);
    }
    
    private void handleTaoHoaDon() {
        try {
            String maKH = view.getMaKhachHang();
            List<ChiTietDTO> chiTiet = view.getChiTiet();
            
            TaoHoaDonRequest request = new TaoHoaDonRequest(maKH, chiTiet);
            TaoHoaDonResponse response = taoHoaDonUseCase.execute(request);
            
            view.showSuccess("Tạo hóa đơn thành công!\nMã: " + response.maHoaDon());
            view.clearForm();
            
        } catch (ValidationException e) {
            view.showError("Lỗi: " + e.getMessage());
        } catch (KhachHangNotFoundException e) {
            view.showError("Không tìm thấy khách hàng");
        }
    }
}

// REST Controller
@RestController
@RequestMapping("/api/hoadon")
public class HoaDonRestController {
    private final TaoHoaDonUseCase taoHoaDonUseCase;
    
    @PostMapping
    public ResponseEntity<TaoHoaDonResponse> taoHoaDon(@RequestBody TaoHoaDonRequest request) {
        try {
            TaoHoaDonResponse response = taoHoaDonUseCase.execute(request);
            return ResponseEntity.ok(response);
        } catch (ValidationException e) {
            return ResponseEntity.badRequest().build();
        }
    }
}
```

#### Repository Implementations (Use Case → Database)

```java
package com.example.adapters.persistence;

// MySQL Implementation
public class HoaDonRepositoryMySQL implements HoaDonRepository {
    private final DataSource dataSource;
    
    public HoaDonRepositoryMySQL(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    @Override
    public void save(HoaDon hoaDon) {
        String sql = "INSERT INTO HoaDon (maHoaDon, ngayLap, tongTien, trangThai, maKH) " +
                     "VALUES (?, ?, ?, ?, ?)";
        
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            
            ps.setString(1, hoaDon.getMaHoaDon());
            ps.setDate(2, Date.valueOf(hoaDon.getNgayLap()));
            ps.setDouble(3, hoaDon.getTongTien());
            ps.setString(4, hoaDon.getTrangThai().name());
            ps.setString(5, hoaDon.getKhachHang().getMaKH());
            
            ps.executeUpdate();
            
        } catch (SQLException e) {
            throw new DatabaseException("Lỗi lưu hóa đơn", e);
        }
    }
    
    @Override
    public List<HoaDon> findByThangNam(int thang, int nam) {
        String sql = "SELECT * FROM HoaDon WHERE MONTH(ngayLap) = ? AND YEAR(ngayLap) = ?";
        
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            
            ps.setInt(1, thang);
            ps.setInt(2, nam);
            ResultSet rs = ps.executeQuery();
            
            List<HoaDon> result = new ArrayList<>();
            while (rs.next()) {
                result.add(mapToEntity(rs));
            }
            return result;
            
        } catch (SQLException e) {
            throw new DatabaseException("Lỗi truy vấn", e);
        }
    }
}

// MongoDB Implementation
public class HoaDonRepositoryMongoDB implements HoaDonRepository {
    private final MongoCollection<Document> collection;
    
    @Override
    public void save(HoaDon hoaDon) {
        Document doc = new Document()
            .append("maHoaDon", hoaDon.getMaHoaDon())
            .append("ngayLap", hoaDon.getNgayLap())
            .append("tongTien", hoaDon.getTongTien());
        
        collection.insertOne(doc);
    }
    
    @Override
    public List<HoaDon> findByThangNam(int thang, int nam) {
        // MongoDB query logic
    }
}
```

---

### 5.4. Tầng 4: Frameworks & Drivers

**Đặc điểm:**
- Tầng ngoài cùng
- Chứa các công cụ cụ thể: Database Config, UI Framework, Web Framework

```java
package com.example.infrastructure;

// Database Configuration
public class DatabaseConfig {
    public static DataSource getDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/qlbh");
        config.setUsername("root");
        config.setPassword("password");
        config.setMaximumPoolSize(10);
        return new HikariDataSource(config);
    }
}

// UI View (Swing)
public class HoaDonSwingView extends JFrame implements HoaDonView {
    private JTextField txtMaKH;
    private JTable tblChiTiet;
    private JButton btnThem;
    
    public HoaDonSwingView() {
        initComponents();
    }
    
    private void initComponents() {
        setTitle("Quản lý Hóa Đơn");
        setSize(800, 600);
        // Setup UI components...
    }
    
    @Override
    public String getMaKhachHang() {
        return txtMaKH.getText();
    }
    
    @Override
    public void showSuccess(String message) {
        JOptionPane.showMessageDialog(this, message);
    }
}

// Dependency Injection (Main)
public class Main {
    public static void main(String[] args) {
        // 1. Setup Infrastructure
        DataSource dataSource = DatabaseConfig.getDataSource();
        
        // 2. Setup Repositories
        HoaDonRepository hoaDonRepo = new HoaDonRepositoryMySQL(dataSource);
        KhachHangRepository khRepo = new KhachHangRepositoryMySQL(dataSource);
        SanPhamRepository spRepo = new SanPhamRepositoryMySQL(dataSource);
        
        // 3. Setup Use Cases
        MaHoaDonGenerator maGen = new MaHoaDonGeneratorImpl();
        TaoHoaDonUseCase useCase = new TaoHoaDonUseCase(hoaDonRepo, khRepo, spRepo, maGen);
        
        // 4. Setup UI
        HoaDonSwingView view = new HoaDonSwingView();
        HoaDonSwingController controller = new HoaDonSwingController(useCase, view);
        
        // 5. Run
        SwingUtilities.invokeLater(() -> view.setVisible(true));
    }
}
```

---

## 6. Dependency Rule (Quy tắc phụ thuộc)

### 6.1. Quy tắc vàng

> **Các dependency (phụ thuộc) chỉ được chỉ từ ngoài vào trong, KHÔNG BAO GIỜ ngược lại.**

```
Frameworks & Drivers  →  Interface Adapters  →  Use Cases  →  Entities
     (Tầng 4)                  (Tầng 3)           (Tầng 2)      (Tầng 1)
```

**Quy tắc:**
-  Tầng ngoài có thể biết về tầng trong
-  Tầng trong KHÔNG ĐƯỢC biết về tầng ngoài

### 6.2. Ví dụ SAI và ĐÚNG

####  SAI: Dependency đi từ trong ra ngoài

```java
// Domain Layer - KHÔNG ĐƯỢC biết về Database!
public class HoaDon {
    private MySQLConnection conn; //  SAI!
    
    public void save() {
        conn.execute("INSERT INTO..."); //  Entity tự lưu vào DB!
    }
}

// Use Case Layer - KHÔNG ĐƯỢC biết về UI!
public class TaoHoaDonUseCase {
    public void execute() {
        JOptionPane.showMessageDialog(null, "Success"); //  SAI!
    }
}
```

####  ĐÚNG: Dependency đi từ ngoài vào trong

```java
// Domain Layer - Hoàn toàn độc lập
public class HoaDon {
    private String ma;
    private double tong;
    // Chỉ business logic, không biết DB, UI
}

// Application Layer - Chỉ biết Interface
public class TaoHoaDonUseCase {
    private HoaDonRepository repository; // Interface!
    
    public void execute(TaoHoaDonRequest request) {
        HoaDon hd = new HoaDon(...);
        repository.save(hd); // Không biết MySQL hay MongoDB
    }
}

// Infrastructure Layer - Biết mọi thứ
public class HoaDonRepositoryMySQL implements HoaDonRepository {
    // Biết MySQL, nhưng Use Case không cần biết class này
}
```

### 6.3. Dependency Inversion trong thực tế

**Vấn đề:** Use Case cần gọi Presenter (tầng ngoài) để hiển thị kết quả

** Cách sai:**
```java
// Use Case gọi trực tiếp Presenter (tầng ngoài)
public class TaoHoaDonUseCase {
    private HoaDonSwingPresenter presenter; //  Concrete class từ tầng ngoài!
    
    public void execute() {
        presenter.display(...); //  Dependency từ trong ra ngoài!
    }
}
```

** Cách đúng: Dùng Output Port (Interface)**
```java
// 1. Define interface ở tầng Use Case
public interface HoaDonOutputPort {
    void present(HoaDonDTO dto);
}

// 2. Use Case phụ thuộc vào interface
public class TaoHoaDonUseCase {
    private HoaDonOutputPort outputPort; // Interface ở tầng này
    
    public void execute(TaoHoaDonRequest request) {
        HoaDon hd = ...;
        HoaDonDTO dto = new HoaDonDTO(hd);
        outputPort.present(dto); //  Gọi interface
    }
}

// 3. Tầng ngoài implement interface
public class HoaDonSwingPresenter implements HoaDonOutputPort {
    @Override
    public void present(HoaDonDTO dto) {
        // UI logic ở đây
        JOptionPane.showMessageDialog(null, "Mã: " + dto.getMa());
    }
}

// 4. Wiring
public class Main {
    public static void main(String[] args) {
        HoaDonOutputPort presenter = new HoaDonSwingPresenter();
        TaoHoaDonUseCase useCase = new TaoHoaDonUseCase(presenter);
    }
}
```

**Kết quả:**
-  Dependency vẫn đi từ ngoài vào trong (Presenter → Interface)
-  Use Case không biết về Swing
-  Có thể thay Presenter dễ dàng

---

## 7. Ứng dụng thực tế trong Java

### 7.1. Cấu trúc project đầy đủ

```
src/
├── main/
│   └── java/
│       └── com/
│           └── example/
│               │
│               ├── domain/                          # TẦNG 1: ENTITIES
│               │   ├── entities/
│               │   │   ├── HoaDon.java
│               │   │   ├── KhachHang.java
│               │   │   ├── SanPham.java
│               │   │   └── ChiTietHoaDon.java
│               │   │
│               │   ├── valueobjects/
│               │   │   └── Money.java
│               │   │
│               │   └── exceptions/
│               │       └── HoaDonException.java
│               │
│               ├── application/                     # TẦNG 2: USE CASES
│               │   ├── usecases/
│               │   │   ├── TaoHoaDonUseCase.java
│               │   │   └── ThongKeDoanhThuUseCase.java
│               │   │
│               │   ├── ports/                      # Interfaces
│               │   │   ├── input/
│               │   │   │   └── TaoHoaDonInputPort.java
│               │   │   │
│               │   │   └── output/
│               │   │       ├── HoaDonRepository.java
│               │   │       └── KhachHangRepository.java
│               │   │
│               │   └── dto/
│               │       ├── TaoHoaDonRequest.java
│               │       └── TaoHoaDonResponse.java
│               │
│               ├── adapters/                        # TẦNG 3: ADAPTERS
│               │   │
│               │   ├── controllers/
│               │   │   ├── swing/
│               │   │   │   └── HoaDonSwingController.java
│               │   │   │
│               │   │   └── rest/
│               │   │       └── HoaDonRestController.java
│               │   │
│               │   └── persistence/
│               │       ├── mysql/
│               │       │   └── HoaDonRepositoryMySQL.java
│               │       │
│               │       └── mongodb/
│               │           └── HoaDonRepositoryMongoDB.java
│               │
│               └── infrastructure/                  # TẦNG 4: FRAMEWORKS
│                   ├── database/
│                   │   └── DatabaseConfig.java
│                   │
│                   ├── ui/
│                   │   └── swing/
│                   │       └── HoaDonSwingView.java
│                   │
│                   └── config/
│                       └── Main.java
│
└── test/
    └── java/
        └── com/
            └── example/
                ├── domain/
                │   └── HoaDonTest.java
                │
                └── application/
                    └── TaoHoaDonUseCaseTest.java
```

### 7.2. Testing với Clean Architecture

```java
// Test Use Case (dễ dàng, nhanh, độc lập)
public class TaoHoaDonUseCaseTest {
    private TaoHoaDonUseCase useCase;
    private FakeHoaDonRepository hoaDonRepo;
    private FakeKhachHangRepository khachHangRepo;
    private FakeSanPhamRepository sanPhamRepo;
    private FakeMaGenerator maGenerator;
    
    @BeforeEach
    void setUp() {
        hoaDonRepo = new FakeHoaDonRepository();
        khachHangRepo = new FakeKhachHangRepository();
        sanPhamRepo = new FakeSanPhamRepository();
        maGenerator = new FakeMaGenerator();
        
        useCase = new TaoHoaDonUseCase(hoaDonRepo, khachHangRepo, sanPhamRepo, maGenerator);
    }
    
    @Test
    void testTaoHoaDon_ThanhCong() {
        // Arrange
        KhachHang kh = new KhachHang("KH001", "Nguyen Van A", LoaiKhachHang.VIP);
        khachHangRepo.save(kh);
        
        SanPham sp = new SanPham("SP001", "Laptop", 20000000, 10);
        sanPhamRepo.save(sp);
        
        TaoHoaDonRequest request = new TaoHoaDonRequest(
            "KH001",
            List.of(new ChiTietDTO("SP001", 2))
        );
        
        // Act
        TaoHoaDonResponse response = useCase.execute(request);
        
        // Assert
        assertNotNull(response.maHoaDon());
        assertEquals(40000000, response.tongTien(), 0.01);
        assertEquals(6000000, response.giamGia(), 0.01); // VIP 15%
    }
    
    @Test
    void testTaoHoaDon_KhachHangKhongTonTai() {
        TaoHoaDonRequest request = new TaoHoaDonRequest(
            "KH999",
            List.of(new ChiTietDTO("SP001", 1))
        );
        
        assertThrows(KhachHangNotFoundException.class, () -> {
            useCase.execute(request);
        });
    }
}

// Fake Repository cho testing (in-memory)
class FakeHoaDonRepository implements HoaDonRepository {
    private List<HoaDon> storage = new ArrayList<>();
    
    @Override
    public void save(HoaDon hd) {
        storage.add(hd);
    }
    
    @Override
    public Optional<HoaDon> findByMa(String ma) {
        return storage.stream()
            .filter(hd -> hd.getMaHoaDon().equals(ma))
            .findFirst();
    }
    
    @Override
    public List<HoaDon> findByThangNam(int thang, int nam) {
        return storage.stream()
            .filter(hd -> hd.getNgayLap().getMonthValue() == thang 
                       && hd.getNgayLap().getYear() == nam)
            .toList();
    }
}
```

---

## 8. So sánh trước và sau

### 8.1. Trước khi áp dụng Clean Architecture

```java
// Tất cả trong 1 class - Monolithic
public class QuanLyHoaDonUI extends JFrame {
    private JTextField txtMaKH, txtTongTien;
    private JTable table;
    private Connection conn;
    
    private void btnThemActionPerformed() {
        // UI logic
        String maKH = txtMaKH.getText();
        if (maKH.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Nhập mã KH!");
            return;
        }
        
        // Validation
        if (maKH.length() < 3) {
            JOptionPane.showMessageDialog(this, "Mã KH >= 3 ký tự!");
            return;
        }
        
        // Database logic
        try {
            conn = DriverManager.getConnection("jdbc:mysql://localhost/qlbh");
            
            // Business logic
            String checkSql = "SELECT COUNT(*) FROM KhachHang WHERE maKH = ?";
            PreparedStatement checkPs = conn.prepareStatement(checkSql);
            checkPs.setString(1, maKH);
            ResultSet rs = checkPs.executeQuery();
            
            if (!rs.next() || rs.getInt(1) == 0) {
                JOptionPane.showMessageDialog(this, "Khách hàng không tồn tại!");
                return;
            }
            
            // Insert
            String sql = "INSERT INTO HoaDon (maHD, maKH, ngayLap, tongTien) VALUES (?, ?, ?, ?)";
            PreparedStatement ps = conn.prepareStatement(sql);
            ps.setString(1, "HD" + System.currentTimeMillis());
            ps.setString(2, maKH);
            ps.setDate(3, new Date(System.currentTimeMillis()));
            ps.setDouble(4, Double.parseDouble(txtTongTien.getText()));
            
            ps.executeUpdate();
            
            // UI update
            JOptionPane.showMessageDialog(this, "Thêm thành công!");
            loadData();
            
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, "Lỗi: " + e.getMessage());
        }
    }
}
```

**Vấn đề:**
-  UI, Business Logic, Database trộn lẫn
-  Không test được
-  Không thể tái sử dụng
-  Thay đổi DB → Sửa toàn bộ
-  Làm Web app → Viết lại từ đầu

### 8.2. Sau khi áp dụng Clean Architecture

```java
// Domain - Độc lập
public class HoaDon {
    private String maHoaDon;
    private LocalDate ngayLap;
    private double tongTien;
    
    public HoaDon(String maHoaDon, LocalDate ngayLap, double tongTien) {
        if (maHoaDon == null || maHoaDon.isBlank()) {
            throw new ValidationException("Mã không được rỗng");
        }
        this.maHoaDon = maHoaDon;
        this.ngayLap = ngayLap;
        this.tongTien = tongTien;
    }
}

// Use Case - Business Logic rõ ràng
public class TaoHoaDonUseCase {
    private final HoaDonRepository hoaDonRepo;
    private final KhachHangRepository khachHangRepo;
    
    public TaoHoaDonResponse execute(TaoHoaDonRequest request) {
        // Validate
        KhachHang kh = khachHangRepo.findByMa(request.maKhachHang())
            .orElseThrow(() -> new KhachHangNotFoundException());
        
        // Create
        String maHD = generateMa();
        HoaDon hoaDon = new HoaDon(maHD, LocalDate.now(), request.tongTien());
        
        // Save
        hoaDonRepo.save(hoaDon);
        
        return new TaoHoaDonResponse(maHD, "Thành công");
    }
}

// Controller - Chỉ điều khiển UI
public class HoaDonSwingController {
    private final TaoHoaDonUseCase useCase;
    private final HoaDonView view;
    
    private void handleThem() {
        try {
            TaoHoaDonRequest request = new TaoHoaDonRequest(
                view.getMaKhachHang(),
                view.getTongTien()
            );
            
            TaoHoaDonResponse response = useCase.execute(request);
            view.showSuccess(response.message());
            
        } catch (ValidationException e) {
            view.showError(e.getMessage());
        }
    }
}

// Repository - Chỉ database
public class HoaDonRepositoryMySQL implements HoaDonRepository {
    public void save(HoaDon hd) {
        // MySQL specific code
    }
}
```

**Lợi ích:**
-  Mỗi class một trách nhiệm rõ ràng
-  Test dễ dàng (mock repository)
-  Tái sử dụng Use Case cho Web, Mobile, CLI
-  Đổi DB chỉ cần thay Repository
-  Code sạch, dễ đọc, dễ maintain

---

## 9. Ưu điểm và nhược điểm

### 9.1. Ưu điểm

| Ưu điểm | Giải thích | Ví dụ |
|---------|-----------|-------|
| **Độc lập Framework** | Business logic không phụ thuộc framework | Chuyển từ Spring sang Jakarta EE dễ dàng |
| **Độc lập Database** | Đổi DB không ảnh hưởng Use Case | MySQL → MongoDB chỉ đổi Repository |
| **Độc lập UI** | Một Use Case cho nhiều UI | Swing, Web, Mobile dùng chung logic |
| **Testable** | Test nhanh, không cần DB/UI | Unit test Use Case với fake repo |
| **Maintainable** | Code sạch, dễ maintain | Mỗi layer rõ ràng, dễ tìm bug |
| **Teamwork** | Nhiều người làm song song | Dev A làm UI, Dev B làm Use Case, Dev C làm DB |
| **Reusable** | Tái sử dụng business logic | API, CLI, Desktop dùng chung Use Case |

### 9.2. Nhược điểm

| Nhược điểm | Giải thích | Khi nào chấp nhận được |
|------------|-----------|------------------------|
| **Phức tạp** | Nhiều layer, nhiều file | Dự án lớn, lâu dài |
| **Boilerplate** | Nhiều interface, DTO | Có công cụ generate code |
| **Learning curve** | Khó học cho người mới | Team có experience |
| **Overkill** | Quá mức cho dự án nhỏ | Dự án < 1 tháng không nên dùng |
| **Tốn thời gian** | Setup ban đầu lâu | Dự án dài hạn (>6 tháng) |

### 9.3. Bảng so sánh tổng quan

| Tiêu chí | Không Clean Arch | Clean Architecture |
|----------|------------------|-------------------|
| **Số lượng file** | Ít (1-5 files) | Nhiều (20-50 files) |
| **Thời gian setup** | Nhanh (1 giờ) | Chậm (1-2 ngày) |
| **Complexity** | Thấp | Cao |
| **Maintainability** | Khó | Dễ |
| **Testability** | Khó/Không thể | Rất dễ |
| **Flexibility** | Thấp | Cao |
| **Learning curve** | Dễ | Khó |
| **Phù hợp dự án** | Nhỏ, ngắn hạn | Lớn, dài hạn |

---

## 10. Khi nào nên áp dụng

### 10.1. NÊN áp dụng Clean Architecture khi:

 **Dự án lớn, phức tạp**
- Nhiều tính năng
- Nhiều business rules
- Team > 3 người

 **Dự án dài hạn**
- Sống > 1 năm
- Cần maintain lâu dài
- Có kế hoạch mở rộng

 **Yêu cầu linh hoạt công nghệ**
- Có thể đổi database
- Có thể đổi UI framework
- Cần hỗ trợ nhiều platform (Web, Mobile, Desktop)

 **Cần test coverage cao**
- Yêu cầu unit test
- CI/CD pipeline
- Chất lượng code quan trọng

 **Team có kinh nghiệm**
- Hiểu SOLID principles
- Biết Design Patterns
- Có thời gian học

**Ví dụ dự án phù hợp:**
- Hệ thống ERP
- Ứng dụng ngân hàng
- E-commerce platform
- SaaS products
- Enterprise applications

### 10.2. KHÔNG NÊN áp dụng khi:

 **Dự án nhỏ, đơn giản**
- CRUD đơn giản
- Ít business logic
- 1-2 người làm

 **Thời gian gấp**
- Deadline < 2 tuần
- Prototype, POC
- MVP cần nhanh

 **Team thiếu kinh nghiệm**
- Junior developers
- Chưa biết SOLID
- Học mất thời gian

 **Dự án ngắn hạn**
- Sống < 3 tháng
- Không maintain
- Dùng 1 lần rồi bỏ

**Ví dụ dự án KHÔNG phù hợp:**
- Assignment trường học
- Internal tools đơn giản
- Scripts, automation
- Hackathon projects
- Quick prototypes

### 10.3. Quy tắc ngón tay cái (Rule of Thumb)

**Hỏi bản thân 5 câu:**

1. **Dự án sống > 6 tháng?**
   - Có → +1 điểm
   - Không → 0 điểm

2. **Team > 2 người?**
   - Có → +1 điểm
   - Không → 0 điểm

3. **Có khả năng đổi công nghệ?**
   - Có → +1 điểm
   - Không → 0 điểm

4. **Cần test tự động?**
   - Có → +1 điểm
   - Không → 0 điểm

5. **Business logic phức tạp?**
   - Có → +1 điểm
   - Không → 0 điểm

**Đánh giá:**
- **4-5 điểm:** NÊN dùng Clean Architecture
- **2-3 điểm:** CÂN NHẮC dùng
- **0-1 điểm:** KHÔNG NÊN dùng

### 10.4. Giải pháp trung gian

Nếu Clean Architecture quá phức tạp nhưng Monolithic quá đơn giản:

**Option 1: Layered Architecture (3-tier)**
```
UI Layer
Business Layer
Data Layer
```

**Option 2: MVC/MVP/MVVM**
```
Model - View - Controller
```

**Option 3: Hexagonal Architecture (nhẹ hơn Clean Arch)**
```
Core (Domain + Use Cases)
Ports (Interfaces)
Adapters (Implementation)
```

---

## Kết luận

### Tóm tắt Clean Architecture

**Clean Architecture là gì?**
- Kiến trúc phần mềm tạo ra hệ thống linh hoạt, dễ maintain, dễ test
- Dựa trên nguyên tắc SOLID và Separation of Concerns
- Chia thành 4 tầng: Entities, Use Cases, Interface Adapters, Frameworks & Drivers

**Khi nào dùng?**
-  Dự án lớn, dài hạn, nhiều người
-  Dự án nhỏ, ngắn hạn, ít người

**Lợi ích chính:**
1. Độc lập Framework, Database, UI
2. Dễ test (unit test nhanh, không cần DB)
3. Dễ maintain (code rõ ràng, tách biệt)
4. Tái sử dụng (1 Use Case → nhiều UI)
5. Làm việc nhóm hiệu quả

**Nhược điểm chính:**
1. Phức tạp (nhiều file, nhiều layer)
2. Boilerplate code (nhiều interface, DTO)
3. Learning curve cao
4. Overkill cho dự án nhỏ



## Tài liệu tham khảo

1. **Clean Architecture** - Robert C. Martin (Uncle Bob)
2. **SOLID Principles** - Robert C. Martin
3. **Hexagonal Architecture** - Alistair Cockburn
4. **Onion Architecture** - Jeffrey Palermo
5. **Domain-Driven Design** - Eric Evans

---

**Ghi chú:** Tài liệu này được tạo để học tập về Clean Architecture trong Java. Các ví dụ được đơn giản hóa để dễ hiểu. 

---



---
title: "So sánh Strategy Pattern và Factory Pattern trong thiết kế phần mềm"
date: 2026-03-16 21:38:00 +0700
categories: [Architecture, Pattern]
tags: [Design pattern, architecture]
---


# Strategy Pattern vs Factory Pattern — Hai anh em hay bị nhầm lẫn

> Nếu bạn từng nhìn vào Strategy Pattern và Factory Pattern rồi tự hỏi "hai cái này khác gì nhau?" thì bạn không đơn độc. Đây là một trong những câu hỏi phổ biến nhất khi học Design Pattern. Bài viết này sẽ giúp bạn phân biệt rõ ràng một lần và mãi mãi — với ví dụ thực tế, code Java chi tiết, và những tình huống dễ nhầm lẫn nhất.

---


## Tại sao dễ nhầm?

Cả hai pattern đều có cấu trúc bề ngoài rất giống:

- Đều có một **interface** hoặc abstract class
- Đều có **nhiều class implement** interface đó
- Đều giúp code **linh hoạt, dễ mở rộng**
- Đều tuân thủ nguyên tắc **Open/Closed Principle** — mở rộng mà không cần sửa code cũ

Nhưng nếu chỉ nhìn cấu trúc mà kết luận chúng giống nhau thì cũng giống như nói "xe đạp và xe máy giống nhau vì đều có 2 bánh". Đúng về hình thức, nhưng **mục đích sử dụng hoàn toàn khác**.

Sự khác biệt nằm ở **câu hỏi mà mỗi pattern trả lời**:

- **Strategy**: "Tôi nên **làm việc này bằng cách nào**?"
- **Factory**: "Tôi nên **tạo ra object nào**?"

Hãy cùng đi sâu vào từng pattern.

---

## Strategy Pattern là gì?

### Định nghĩa

Strategy Pattern thuộc nhóm **Behavioral Pattern** (mẫu hành vi). Nó cho phép bạn định nghĩa một nhóm các thuật toán, đóng gói từng thuật toán thành class riêng, và cho phép chúng **hoán đổi cho nhau** lúc runtime.

Nói đơn giản: **cùng một việc nhưng có nhiều cách làm khác nhau**, và bạn muốn đổi cách làm một cách linh hoạt.

### Cấu trúc

<p align="center">
  <img src="/assets/images/pattern/strategy_pattern.png" alt="Image title_1" />
</p>


Có 3 thành phần chính:

1. **Strategy (Interface)**: định nghĩa method chung mà tất cả các chiến lược phải implement
2. **Concrete Strategy**: các class cụ thể implement interface, mỗi class là một cách làm khác nhau
3. **Context**: class sử dụng strategy, giữ một tham chiếu đến interface và gọi method qua interface đó

### Ví dụ thực tế: tính giá nhập hàng

Giả sử bạn đang xây dựng hệ thống quản lý cho thuê trang phục. Khi nhập hàng từ nhà cung cấp (NCC), mỗi NCC có chính sách giá khác nhau:

- **NCC thường**: tính giá gốc, không giảm
- **NCC VIP** (hợp tác lâu năm): giảm 10%
- **NCC bulk** (nhập số lượng lớn): giảm 15%

Nếu không dùng pattern, bạn sẽ viết thế này:

```java
//  Code xấu — khó mở rộng, vi phạm Open/Closed Principle
public float tinhGia(float donGia, int soLuong, String loaiNCC) {
    if (loaiNCC.equals("thuong")) {
        return donGia * soLuong;
    } else if (loaiNCC.equals("vip")) {
        return donGia * soLuong * 0.9f;
    } else if (loaiNCC.equals("bulk")) {
        return donGia * soLuong * 0.85f;
    }
    // Thêm loại NCC mới → phải sửa method này
    // Càng nhiều loại → if-else càng dài → càng khó maintain
    return 0;
}
```

Với Strategy Pattern:

```java
//  Interface — định nghĩa hành vi chung
public interface PricingStrategy {
    float tinhGia(float donGia, int soLuong);
}

//  Concrete Strategy 1: Giá thường
public class NormalPricing implements PricingStrategy {
    @Override
    public float tinhGia(float donGia, int soLuong) {
        System.out.println("[NormalPricing] Tính giá gốc, không giảm");
        return donGia * soLuong;
    }
}

//  Concrete Strategy 2: Giá VIP — giảm 10%
public class VIPPricing implements PricingStrategy {
    @Override
    public float tinhGia(float donGia, int soLuong) {
        float giaGoc = donGia * soLuong;
        float giamGia = giaGoc * 0.1f;
        System.out.println("[VIPPricing] Giá gốc: " + giaGoc + " → Giảm 10%: -" + giamGia);
        return giaGoc - giamGia;
    }
}

//  Concrete Strategy 3: Giá bulk — giảm 15%
public class BulkPricing implements PricingStrategy {
    @Override
    public float tinhGia(float donGia, int soLuong) {
        float giaGoc = donGia * soLuong;
        float giamGia = giaGoc * 0.15f;
        System.out.println("[BulkPricing] Giá gốc: " + giaGoc + " → Giảm 15%: -" + giamGia);
        return giaGoc - giamGia;
    }
}

//  Context — sử dụng strategy
public class NhapTrangPhucContext {
    private PricingStrategy pricingStrategy;

    // Inject strategy qua constructor
    public NhapTrangPhucContext(PricingStrategy pricingStrategy) {
        this.pricingStrategy = pricingStrategy;
    }

    // Có thể đổi strategy lúc runtime
    public void setPricingStrategy(PricingStrategy pricingStrategy) {
        this.pricingStrategy = pricingStrategy;
    }

    public float tinhTongTien(float donGia, int soLuong) {
        return pricingStrategy.tinhGia(donGia, soLuong);
    }
}
```

Sử dụng:

```java
public class Demo {
    public static void main(String[] args) {
        // Nhập từ NCC thường
        NhapTrangPhucContext context = new NhapTrangPhucContext(new NormalPricing());
        float gia1 = context.tinhTongTien(500000, 10);
        System.out.println("Tổng tiền: " + gia1);
        // → [NormalPricing] Tính giá gốc, không giảm
        // → Tổng tiền: 5000000.0

        System.out.println("---");

        // Đổi sang NCC VIP — không cần sửa code, chỉ đổi strategy
        context.setPricingStrategy(new VIPPricing());
        float gia2 = context.tinhTongTien(500000, 10);
        System.out.println("Tổng tiền: " + gia2);
        // → [VIPPricing] Giá gốc: 5000000.0 → Giảm 10%: -500000.0
        // → Tổng tiền: 4500000.0

        System.out.println("---");

        // Đổi sang NCC bulk
        context.setPricingStrategy(new BulkPricing());
        float gia3 = context.tinhTongTien(500000, 10);
        System.out.println("Tổng tiền: " + gia3);
        // → [BulkPricing] Giá gốc: 5000000.0 → Giảm 15%: -750000.0
        // → Tổng tiền: 4250000.0
    }
}
```

### Điểm mấu chốt của Strategy

- Object (context) **đã tồn tại**, chỉ đổi cách xử lý
- **Đổi được lúc runtime** bằng setter
- Thêm strategy mới = thêm 1 class, **không sửa code cũ**
- Loại bỏ hoàn toàn if-else phức tạp

---

## Factory Pattern là gì?

### Định nghĩa

Factory Pattern thuộc nhóm **Creational Pattern** (mẫu khởi tạo). Nó cung cấp một interface để tạo object, nhưng để các subclass hoặc method quyết định **loại object nào sẽ được tạo**.

Nói đơn giản: bạn cần tạo ra các loại object khác nhau, nhưng muốn **che giấu logic khởi tạo** để client không phụ thuộc vào class cụ thể.

### Cấu trúc

<p align="center">
  <img src="/assets/images/pattern/factory_pattern.png" alt="Image title_1" />
</p>


Có 3 thành phần chính:

1. **Product (Interface/Abstract)**: định nghĩa interface chung cho các object sẽ được tạo
2. **Concrete Product**: các class cụ thể, mỗi class là một loại object khác nhau
3. **Factory**: class chứa method tạo object, quyết định tạo loại nào dựa trên tham số

### Ví dụ thực tế: tạo trang phục theo loại

Cửa hàng cho thuê trang phục có nhiều loại: vest, áo dài, cosplay. Mỗi loại có thuộc tính riêng, cách tính giá thuê riêng, yêu cầu bảo quản riêng.

Nếu không dùng pattern:

```java
//  Code xấu — client phụ thuộc trực tiếp vào class cụ thể
public TrangPhuc taoTrangPhuc(String loai) {
    if (loai.equals("vest")) {
        VestTrangPhuc vest = new VestTrangPhuc();
        vest.setTen("Vest công sở");
        vest.setGiaThue(500000);
        vest.setChatLieu("Wool");
        vest.setCanLaUi(true);
        return vest;
    } else if (loai.equals("aodai")) {
        AoDaiTrangPhuc aoDai = new AoDaiTrangPhuc();
        aoDai.setTen("Áo dài truyền thống");
        aoDai.setGiaThue(300000);
        aoDai.setChatLieu("Lụa");
        aoDai.setCanTheu(true);
        return aoDai;
    }
    // Thêm loại mới → phải sửa method này
    return null;
}
```

Với Factory Pattern:

```java
//  Abstract Product — định nghĩa interface chung
public abstract class TrangPhuc {
    protected String maTrangPhuc;
    protected String tenTrangPhuc;
    protected float giaThue;
    protected String loaiTrangPhuc;

    // Method chung mà mọi trang phục đều có
    public abstract void hienThiThongTin();
    public abstract float tinhGiaThue(int soNgay);
    public abstract String huongDanBaoQuan();

    // Getter/Setter
    public String getTenTrangPhuc() { return tenTrangPhuc; }
    public float getGiaThue() { return giaThue; }
}

//  Concrete Product 1: Vest
public class VestTrangPhuc extends TrangPhuc {
    private String chatLieu;
    private boolean canLaUi;

    public VestTrangPhuc() {
        this.maTrangPhuc = "VEST-" + System.currentTimeMillis();
        this.tenTrangPhuc = "Vest công sở";
        this.giaThue = 500000;
        this.loaiTrangPhuc = "Vest";
        this.chatLieu = "Wool blend";
        this.canLaUi = true;
    }

    @Override
    public void hienThiThongTin() {
        System.out.println("=== VEST ===");
        System.out.println("Tên: " + tenTrangPhuc);
        System.out.println("Giá thuê/ngày: " + giaThue + " VND");
        System.out.println("Chất liệu: " + chatLieu);
        System.out.println("Cần là ủi: " + (canLaUi ? "Có" : "Không"));
    }

    @Override
    public float tinhGiaThue(int soNgay) {
        // Vest thuê dài ngày giảm 5%/ngày từ ngày thứ 3
        if (soNgay <= 2) return giaThue * soNgay;
        return giaThue * 2 + giaThue * 0.95f * (soNgay - 2);
    }

    @Override
    public String huongDanBaoQuan() {
        return "Treo trên móc áo, tránh gấp. Là ủi trước khi trả.";
    }
}

//  Concrete Product 2: Áo dài
public class AoDaiTrangPhuc extends TrangPhuc {
    private String chatLieu;
    private boolean coTheu;

    public AoDaiTrangPhuc() {
        this.maTrangPhuc = "AD-" + System.currentTimeMillis();
        this.tenTrangPhuc = "Áo dài truyền thống";
        this.giaThue = 300000;
        this.loaiTrangPhuc = "Áo dài";
        this.chatLieu = "Lụa tơ tằm";
        this.coTheu = true;
    }

    @Override
    public void hienThiThongTin() {
        System.out.println("=== ÁO DÀI ===");
        System.out.println("Tên: " + tenTrangPhuc);
        System.out.println("Giá thuê/ngày: " + giaThue + " VND");
        System.out.println("Chất liệu: " + chatLieu);
        System.out.println("Có thêu: " + (coTheu ? "Có" : "Không"));
    }

    @Override
    public float tinhGiaThue(int soNgay) {
        // Áo dài giá cố định theo ngày
        return giaThue * soNgay;
    }

    @Override
    public String huongDanBaoQuan() {
        return "Giặt tay nhẹ nhàng, không vắt. Phơi trong bóng râm.";
    }
}

//  Concrete Product 3: Cosplay
public class CosplayTrangPhuc extends TrangPhuc {
    private String nhanVat;
    private int soMonPhuKien;

    public CosplayTrangPhuc() {
        this.maTrangPhuc = "COS-" + System.currentTimeMillis();
        this.tenTrangPhuc = "Bộ cosplay";
        this.giaThue = 800000;
        this.loaiTrangPhuc = "Cosplay";
        this.nhanVat = "Custom";
        this.soMonPhuKien = 3;
    }

    @Override
    public void hienThiThongTin() {
        System.out.println("=== COSPLAY ===");
        System.out.println("Tên: " + tenTrangPhuc);
        System.out.println("Giá thuê/ngày: " + giaThue + " VND");
        System.out.println("Nhân vật: " + nhanVat);
        System.out.println("Số phụ kiện: " + soMonPhuKien + " món");
    }

    @Override
    public float tinhGiaThue(int soNgay) {
        // Cosplay có phụ phí phụ kiện
        float phuPhi = soMonPhuKien * 50000;
        return (giaThue * soNgay) + phuPhi;
    }

    @Override
    public String huongDanBaoQuan() {
        return "Kiểm tra đầy đủ phụ kiện khi trả. Không giặt, chỉ lau nhẹ.";
    }
}

//  Factory — che giấu logic khởi tạo
public class TrangPhucFactory {
    public static TrangPhuc create(String loai) {
        switch (loai.toLowerCase()) {
            case "vest":
                return new VestTrangPhuc();
            case "aodai":
                return new AoDaiTrangPhuc();
            case "cosplay":
                return new CosplayTrangPhuc();
            default:
                throw new IllegalArgumentException(
                    "Loại trang phục không hợp lệ: " + loai
                );
        }
    }
}
```

Sử dụng:

```java
public class Demo {
    public static void main(String[] args) {
        // Client không cần biết VestTrangPhuc, AoDaiTrangPhuc...
        // Chỉ cần gọi Factory với tên loại
        TrangPhuc tp1 = TrangPhucFactory.create("vest");
        tp1.hienThiThongTin();
        System.out.println("Thuê 5 ngày: " + tp1.tinhGiaThue(5) + " VND");
        System.out.println("Bảo quản: " + tp1.huongDanBaoQuan());
        // → === VEST ===
        // → Tên: Vest công sở
        // → Giá thuê/ngày: 500000.0 VND
        // → Chất liệu: Wool blend
        // → Cần là ủi: Có
        // → Thuê 5 ngày: 2425000.0 VND
        // → Bảo quản: Treo trên móc áo, tránh gấp. Là ủi trước khi trả.

        System.out.println("---");

        TrangPhuc tp2 = TrangPhucFactory.create("cosplay");
        tp2.hienThiThongTin();
        System.out.println("Thuê 2 ngày: " + tp2.tinhGiaThue(2) + " VND");
        System.out.println("Bảo quản: " + tp2.huongDanBaoQuan());
        // → === COSPLAY ===
        // → Tên: Bộ cosplay
        // → Giá thuê/ngày: 800000.0 VND
        // → Nhân vật: Custom
        // → Số phụ kiện: 3 món
        // → Thuê 2 ngày: 1750000.0 VND
        // → Bảo quản: Kiểm tra đầy đủ phụ kiện khi trả. Không giặt, chỉ lau nhẹ.
    }
}
```

### Điểm mấu chốt của Factory

- Object **chưa tồn tại**, cần được tạo mới
- Client **không biết** class cụ thể, chỉ làm việc qua interface
- Logic khởi tạo phức tạp được **gom vào một chỗ**
- Thêm loại mới = thêm 1 class + sửa Factory (hoặc dùng Abstract Factory để không cần sửa)

---

## So sánh trực tiếp

### Bảng so sánh tổng quan

| Tiêu chí | Strategy Pattern | Factory Pattern |
|---|---|---|
| **Câu hỏi cốt lõi** | Làm thế nào? | Tạo cái gì? |
| **Nhóm** | Behavioral (hành vi) | Creational (khởi tạo) |
| **Mục đích** | Thay đổi **hành vi/thuật toán** | Thay đổi **loại object được tạo** |
| **Thời điểm** | Object đã tồn tại, đổi cách xử lý | Object chưa tồn tại, cần tạo mới |
| **Interface chứa** | Method hành vi (tinhGia, thanhToan) | Method tạo object (create, build) |
| **Đổi lúc runtime** | Có — đổi strategy bất cứ lúc nào qua setter | Thường chỉ tạo 1 lần |
| **Client biết gì** | Biết context, chọn strategy | Không biết class cụ thể, chỉ biết Factory |
| **Loại bỏ** | if-else chọn thuật toán | if-else chọn class để new |
| **Số object tạo ra** | Không tạo object nghiệp vụ mới | Tạo ra object nghiệp vụ mới |

### So sánh qua phép ẩn dụ

**Strategy giống như chọn phương tiện đi từ A đến B:**

Bạn đã có điểm đi (A) và điểm đến (B). Bạn chỉ cần chọn **cách đi**: đi bộ, xe bus, taxi, hay grab. Điểm đi và điểm đến không đổi — chỉ phương tiện thay đổi.

```
Cùng 1 hành trình → khác cách di chuyển
A ──(đi bộ)──→ B
A ──(taxi)────→ B
A ──(grab)────→ B
```

**Factory giống như đặt đồ uống tại quán:**

Bạn chưa có đồ uống. Bạn nói với nhân viên "cho tôi một ly cà phê" hoặc "cho tôi một ly trà sữa". Nhân viên (Factory) sẽ **tạo ra** đúng loại đồ uống cho bạn. Bạn không cần biết công thức pha chế (logic khởi tạo).

```
"cà phê"  → Factory → ☕ CaPheDen (object mới)
"trà sữa" → Factory → 🧋 TraSua (object mới)
"sinh tố" → Factory → 🥤 SinhTo (object mới)
```

### So sánh qua code structure

```java
// STRATEGY — đổi CÁCH LÀM
context.setStrategy(new StrategyA());  // đổi hành vi
context.execute();                      // cùng method, khác kết quả

context.setStrategy(new StrategyB());  // đổi hành vi khác
context.execute();                      // cùng method, khác kết quả

// FACTORY — đổi VẬT ĐƯỢC TẠO
Product p1 = Factory.create("typeA");  // tạo object loại A
Product p2 = Factory.create("typeB");  // tạo object loại B
// p1 và p2 là 2 object KHÁC LOẠI
```

---

## Những tình huống dễ nhầm nhất

### Tình huống 1: "Thanh toán bằng nhiều hình thức"

- Tiền mặt, chuyển khoản, ví điện tử → **Strategy**
- Lý do: cùng 1 hóa đơn, cùng 1 số tiền, chỉ khác **cách trả**

```java
//  Strategy — đúng
context.setPaymentStrategy(new CashPayment());
context.thanhToan(5000000);

context.setPaymentStrategy(new BankTransfer());
context.thanhToan(5000000);
```

### Tình huống 2: "Tạo nhiều loại tài khoản ngân hàng"

- Tài khoản tiết kiệm, tài khoản thanh toán, tài khoản VIP → **Factory**
- Lý do: mỗi loại là một **object khác nhau** với thuộc tính riêng

```java
//  Factory — đúng
TaiKhoan tk1 = TaiKhoanFactory.create("tietkiem");   // → TKTietKiem
TaiKhoan tk2 = TaiKhoanFactory.create("thanhtoan");  // → TKThanhToan
```

### Tình huống 3: "Sắp xếp danh sách theo nhiều tiêu chí"

- Sắp xếp theo tên, theo giá, theo ngày → **Strategy**
- Lý do: cùng 1 danh sách, chỉ đổi **cách sắp xếp**

```java
//  Strategy — đúng
list.sort(new SortByName());   // sắp theo tên
list.sort(new SortByPrice());  // sắp theo giá
```

### Tình huống 4: "Đọc file từ nhiều định dạng khác nhau"

- Đọc CSV, Excel, JSON → **Factory** (tạo ra đúng loại reader)
- Lý do: mỗi định dạng cần **object reader khác nhau**

```java
//  Factory — đúng
FileReader reader = FileReaderFactory.create("csv");    // → CSVReader
FileReader reader2 = FileReaderFactory.create("excel"); // → ExcelReader
```

### Tình huống 5: "Gửi thông báo qua nhiều kênh"

- Gửi qua email, SMS, push notification → **Strategy**
- Lý do: cùng 1 nội dung thông báo, chỉ đổi **kênh gửi**

```java
//  Strategy — đúng
notifier.setStrategy(new EmailNotification());
notifier.send("Đơn hàng đã được xác nhận");

notifier.setStrategy(new SMSNotification());
notifier.send("Đơn hàng đã được xác nhận");
```

### Quy tắc phân biệt nhanh

Khi phân vân, hãy tự hỏi:

1. **"Tôi đang cần tạo object mới hay đổi hành vi?"**
   - Tạo object mới → Factory
   - Đổi hành vi → Strategy

2. **"Sau khi dùng pattern, tôi có thêm object nghiệp vụ mới không?"**
   - Có (thêm trang phục, thêm tài khoản, thêm sản phẩm) → Factory
   - Không (chỉ đổi cách tính, cách gửi, cách sắp xếp) → Strategy

3. **"Client có cần biết loại cụ thể không?"**
   - Không cần biết → Factory (che giấu class cụ thể)
   - Client tự chọn → Strategy (client biết và chọn strategy)

---

## Khi nào dùng cái nào?

### Dùng Strategy khi

- Bạn có **một thao tác** nhưng cần **nhiều cách thực hiện** khác nhau
- Muốn **đổi thuật toán lúc runtime** mà không sửa code
- Thấy code có nhiều `if-else` hoặc `switch` để **chọn cách xử lý**
- Các thuật toán có **cùng input/output** nhưng khác logic bên trong
- Ví dụ: tính giá, thanh toán, sắp xếp, validate, nén file, mã hóa...

### Dùng Factory khi

- Bạn cần **tạo object** nhưng không muốn client phụ thuộc vào class cụ thể
- Loại object được tạo **phụ thuộc vào điều kiện** (input, config, database...)
- Logic khởi tạo **phức tạp** (set nhiều thuộc tính, gọi nhiều method)
- Thấy code có nhiều `new ConcreteClass()` **rải rác** khắp nơi
- Ví dụ: tạo trang phục, tạo tài khoản, tạo document, tạo connection...

---

## Kết hợp cả hai

Trong thực tế, hai pattern này rất hay được dùng cùng nhau. Factory tạo ra đúng Strategy, sau đó Strategy thực thi hành vi.

```java
// Factory tạo ra đúng PricingStrategy dựa trên loại NCC
public class PricingStrategyFactory {
    public static PricingStrategy create(String loaiNCC) {
        switch (loaiNCC.toLowerCase()) {
            case "vip":
                return new VIPPricing();
            case "bulk":
                return new BulkPricing();
            case "normal":
            default:
                return new NormalPricing();
        }
    }
}

// Factory tạo ra đúng PaymentStrategy dựa trên hình thức thanh toán
public class PaymentStrategyFactory {
    public static PaymentStrategy create(String hinhThuc) {
        switch (hinhThuc.toLowerCase()) {
            case "cash":
                return new CashPayment();
            case "bank":
                return new BankTransfer();
            default:
                throw new IllegalArgumentException(
                    "Hình thức thanh toán không hợp lệ: " + hinhThuc
                );
        }
    }
}

// Sử dụng kết hợp — rất sạch và linh hoạt
public class Demo {
    public static void main(String[] args) {
        // Nhập từ NCC VIP, thanh toán chuyển khoản
        String loaiNCC = "vip";           // có thể đọc từ database
        String hinhThucTT = "bank";       // có thể đọc từ UI

        // Factory tạo đúng strategy
        PricingStrategy pricing = PricingStrategyFactory.create(loaiNCC);
        PaymentStrategy payment = PaymentStrategyFactory.create(hinhThucTT);

        // Context dùng strategy để xử lý
        NhapTrangPhucContext context = new NhapTrangPhucContext(pricing, payment);
        float tongTien = context.tinhTongTien(500000, 10);
        context.thanhToan(tongTien);
    }
}
```

Ở đây mỗi pattern giữ đúng trách nhiệm:

- **Factory** lo việc **tạo ra đúng strategy** → trả lời "tạo cái gì?"
- **Strategy** lo việc **thực thi hành vi** → trả lời "làm thế nào?"

Kết hợp này rất phổ biến trong các dự án thực tế và là dấu hiệu của code chất lượng cao.

---

## Kết luận

Strategy và Factory giống nhau về cấu trúc bên ngoài — đều có interface, đều có nhiều class implement — nhưng **khác nhau hoàn toàn về bản chất**:

- **Strategy** thay đổi **cách làm** → "Cùng một việc, nhiều cách thực hiện"
- **Factory** thay đổi **vật được tạo** → "Cùng một lệnh tạo, nhiều loại object"

Lần sau khi phân vân nên dùng pattern nào, hãy tự hỏi: **"Mình đang muốn đổi hành vi hay tạo object mới?"** Câu trả lời sẽ dẫn bạn đến đúng pattern.

Và nhớ rằng, hai pattern này không loại trừ nhau — chúng có thể **kết hợp** để tạo ra code vừa linh hoạt vừa sạch sẽ.

---


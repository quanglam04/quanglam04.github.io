---
title: "SOLID Principles trong phát triển phần mềm"
date: 2025-08-11 01:17:00  +0700
categories: [Programming, BestPractices]
tags: [clean code, coding style, best practice]
---

---


_Nguồn: baeldung.com_

## 1. Overview - Tổng quan

Trong bài nafym chúng ta sẽ thảo luận về **Các nguyên lý SOLID trong thiế kế hướng đối tượng**

Đầu tiên, chúng ta sẽ bắt đầu bằng cách tìm hiểu lý do chúng ra đời và tại sao nên xem xét chúng khi thiết kế phần mềm. Sau đó, chúng ta sẽ trình bày từng nguyên lý cùng một số ví dụ mã nguồn

## 2. The reason for SOLID Principles - Lý do ra đời của các Nguyên lý SOLID

SOLID là một tập các nguyên lý thiết kế giúp tạo ra phần mềm dễ bảo trì hơn, dễ hiểu hơn và linh hoạt hơn. Do đó, khi các ứng dụng của chúng ta phát triển về quy mô, chúng ta có thể giảm độ phức tạp của chúng và tự cứu mình ra khởi rất nhiều rắc rối về sau!

Năm khái niệm sau đây tạo nên các nguyên lý SOLID:

1. **S**ingle Responsibility - Trách nhiệm đơn nhất
2. **O**pen/Closed - Mở/đóng
3. **L**iskov Substitution - Thay thế Liskov
4. **I**nterface Segragation - Phân tách giao diện
5. **D**ependency Inversion - Đảo ngược phụ thuộc

## 3. Single Responsibility - Nguyên lý Đơn trách nhiệm

Hãy bắt đầu với nguyên lý trách nhiệm đơn nhất. Đúng như tên gọi, nguyên lý này nêu rõ rằng **một lớp chỉ nên có một trách nhiệm duy nhất**. Hơn nữa, nó chỉ nên có **một lý do duy nhất để thay đổi.**

**Nguyên lý này giúp chúng ta xây dựng phần mềm tốt hơn như thế nào?**Hãy cùng xem một vài lợi ích của nó:

- Kiểm thử: Một lớp với một trách nhiệm duy nhất sẽ có số lượng trường hợp kiểm thử ít hơn nhiều
- Giảm sự gắn kết(coupling): Ít chức năng hơn trong một lớp udy nhất sẽ dẫn đến ít sự phụ thuộc hơn
- Sắp xếp: Các lớp nhỏ hơn, được tổ chức tốt sẽ dễ tìm kiếm hơn so với các lớp nguyên khối

Ví dụ, hãy xem xét một lớp để đại diện cho một cuốn sách:

```
public class Book {

    private String name;
    private String author;
    private String text;

    //constructor, getters and setters
}
```

Trong đoạn mã này, chúng ta lưu trữ tên, tác giả và nội dung liên quan dến một thể hiện của lớp Book. Bây giờ hãy thêm một vài phương thức để truy vấn nội dung:

```
public class Book {

    private String name;
    private String author;
    private String text;

    //constructor, getters and setters

    // methods that directly relate to the book properties
    public String replaceWordInText(String word, String replacementWord){
        return text.replaceAll(word, replacementWord);
    }

    public boolean isWordInText(String word){
        return text.contains(word);
    }
}
```

Hiện tại lớp `Book` của chúng ta hoạt động tốt, và chúng ta có thể lưu trữ bao nhiêu cuốn sách tùy thích trong ứng dụng. Nhưng việc lưu trữ thông tin thì có ích gì nếu chúng ta không thể trích xuất nội dung ra console để đọc?

Hãy "ném sự cẩn trọng vào gió" (tức là làm mà không suy nghĩ) và thêm một phương thức in:

```
public class BadBook {
    //...

    void printTextToConsole(){
        // our code for formatting and printing the text
    }
}
```

Tuy nhiên, đoạn mã này đã vi phạm nguyên lý trách nhiệm đơn nhất mà chúng đã nói đến trước đó.
Để khắc phục được vấn đề này, chúng ta nên triển khai một lớp riêng biệt chỉ để xử lý việc `in nội dung văn bản`:

```
public class BookPrinter {

    // methods for outputting text
    void printTextToConsole(String text){
        //our code for formatting and printing the text
    }

    void printTextToAnotherMedium(String text){
        // code for writing to any other location..
    }
}
```

Tuyệt vời. Chúng ta đã phát triển một không chỉ giúp `Book` thoát khỏi nhiệm vụ in ấn, mà còn có thể tận dụng lớp `BookPrinter` của mình để gửi văn bản tới các phương tiện khác
Dù là gửi qua email, ghi nhật ký(logging), hay bất cứ thứ gì khác, chúng ta đều có một lớp riêng biệt chuyên trách cho mối quan hệ này

## 4. Open for Extension, Closed for Modification - Nguyên lý Mở rộng, Đóng với sửa đổi

Đã đến lúc tìm hiểu chữ O trong SOLID, được gọi là **nguyên lý mở/đóng**. Đơn giản mà nói, **các lớp nên mở để mở rộng nhưng đóng để sửa đổi**. Bằng cách này, chúng ta ngăn chặn việc tự ý sửa đổi mã hiện có và gây ra các lỗi tiềm ẩn mới trong một ứng dụng đang hoạt động tốt.

Tất nhiên, **một ngoại lệ duy nhất của quy tắc này là khi sửa lỗi trong mã hiện có.**

Hãy cùng khám phá khái niệm này với một ví dụ mã nhanh. Như một phần của dự án mới, hãy tưởng tượng chúng ta đã triển khai một lớp `Guitar`.

Nó đã được phát triển hoàn chỉnh và thậm chí còn có núm chỉnh âm lượng:

```
public class Guitar {

    private String make;
    private String model;
    private int volume;

    //Constructors, getters & setters
}
```

Chúng ta ra mắt ứng dụng, và mọi người đều yêu thích nó. Nhưng sau vài tháng, chúng ta quyết định chiếc Guitar hơi nhàm chán và cần `thêm` một mẫu họa tiết ngọn lửa ngầu lòi để trông "rock and roll" hơn.

Vào thời điểm này, có thể chúng ta sẽ muốn mở lớp Guitar và thêm ngay họa tiết ngọn lửa vào — nhưng ai mà biết được điều đó có thể gây ra những lỗi gì trong ứng dụng của chúng ta.

Thay vào đó, hãy tuân thủ **nguyên tắc mở/đóng và đơn giản là mở rộng lớp Guitar của chúng ta**:

```
public class SuperCoolGuitarWithFlames extends Guitar {

    private String flameColor;

    //constructor, getters + setters
}
```

Bằng cách mở rộng lớp `Guitar`, chúng ta có thể chắc chắn rằng ứng dụng hiện có của chúng ta sẽ không bị ảnh hưởng.

## 5. Liskov Substitution - Nguyên lý thay thế Liskov

Tiếp theo trong danh sách của chúng ta là Nguyên lý Thay thế Liskov, có lẽ là nguyên lý phức tạp nhất trong năm nguyên lý. **Đơn giản mà nói, nếu lớp A là kiểu con của lớp B, chúng ta có thể thay thế B bằng A mà không làm thay đổi hành vi của chương trình.**

Hãy cùng đi thẳng vào mã nguồn để giúp chúng ta hiểu khái niệm này:

```
public interface Car {

    void turnOnEngine();
    void accelerate();
}
```

Ở trên, chúng ta định nghĩa một giao diện `Car` đơn giản với một vài phương thức mà tất cả các loại xe ô tô nên có khả năng thực hiện: khởi động động cơ và tăng tốc tiến về phía trước.

Hãy cùng triển khai giao diện này và cung cấp mã cho các phương thức:

```
public class MotorCar implements Car {

    private Engine engine;

    //Constructors, getters + setters

    public void turnOnEngine() {
        //turn on the engine!
        engine.on();
    }

    public void accelerate() {
        //move forward!
        engine.powerOn(1000);
    }
}
```

Như đoạn mã của chúng ta mô tả, chúng ta có một động cơ mà chúng ta có thể bật lên và tăng công suất.

Nhưng khoan đã — chúng ta đang sống trong thời đại của **xe điện**:

```
public class ElectricCar implements Car {

    public void turnOnEngine() {
        throw new AssertionError("I don't have an engine!");
    }

    public void accelerate() {
        //this acceleration is crazy!
    }
}
```

Bằng cách đưa một chiếc ô tô không có động cơ vào, chúng ta đang thay đổi hành vi vốn có của chương trình. Đây là một **sự vi phạm trắng trợn nguyên lý thay thế Liskov và khó khắc phục hơn so với hai nguyên lý trước**.

Một giải pháp khả thi là tái cấu trúc mô hình của chúng ta thành các giao diện có tính đến trạng thái "không có động cơ" của chiếc ô tô.

## 6. Interface Segragation - Nguyên lý phân tách Giao diện

Tiếp theo là chữ `I` trong `SOLID`, viết tắt của **nguyên lý phân tách giao diện (interface segregation)**. Đơn giản mà nói, nó có nghĩa là **các giao diện lớn hơn nên được chia thành các giao diện nhỏ hơn**. Bằng cách này, chúng ta có thể đảm bảo rằng các lớp triển khai chỉ cần quan tâm đến các phương thức mà chúng cần.

Để minh họa cho ví dụ này, chúng ta sẽ thử sức với vai trò của những người trông coi sở thú. Cụ thể hơn, chúng ta sẽ làm việc trong chuồng gấu.

Hãy bắt đầu với một giao diện phác thảo vai trò của chúng ta với tư cách là người trông coi gấu:

```
public interface BearKeeper {
    void washTheBear();
    void feedTheBear();
    void petTheBear();
}
```

Là những người chăm sóc sở thú đầy nhiệt huyết, chúng ta rất sẵn lòng tắm rửa và cho những chú gấu đáng yêu của mình ăn. Nhưng chúng ta cũng quá ý thức được sự nguy hiểm khi vuốt ve chúng. Thật không may, giao diện của chúng ta khá lớn, và chúng ta không có lựa chọn nào khác ngoài việc triển khai mã để vuốt ve gấu.

Hãy khắc phục điều này bằng cách **chia giao diện lớn của chúng ta thành ba giao diện riêng biệt**:

```
public interface BearCleaner {
    void washTheBear();
}

public interface BearFeeder {
    void feedTheBear();
}

public interface BearPetter {
    void petTheBear();
}
```

Giờ đây, nhờ có **nguyên lý phân tách giao diện**, chúng ta có thể thoải mái chỉ triển khai những phương thức thực sự quan trọng đối với mình:

```
public class BearCarer implements BearCleaner, BearFeeder {

    public void washTheBear() {
        //I think we missed a spot...
    }

    public void feedTheBear() {
        //Tuna Tuesdays...
    }
}
```

Và cuối cùng, chúng ta có thể để những việc nguy hiểm lại cho những người liều lĩnh:

```
public class CrazyPerson implements BearPetter {

    public void petTheBear() {
        //Good luck with that!
    }
}
```

Tiếp tục, chúng ta thậm chí có thể chia lớp `BookPrinter` từ ví dụ trước đó của chúng ta để sử dụng nguyên lý phân tách giao diện theo cùng một cách. Bằng cách triển khai một giao diện Printer với một phương thức print duy nhất, chúng ta có thể tạo các thể hiện riêng biệt cho các lớp _ConsoleBookPrinter_ và _OtherMediaBookPrinter_.

## 7. Dependency Inversion - Nguyên lý đảo ngược phụ thuộc

**Nguyên lý đảo ngược phụ thuộc (Dependency Inversion Principle) đề cập đến việc tách rời các mô-đun phần mềm. Bằng cách này, thay vì các mô-đun cấp cao phụ thuộc vào các mô-đun cấp thấp, cả hai sẽ phụ thuộc vào các lớp trừu tượng (abstractions).**

Để minh họa điều này, hãy quay ngược về "thời xa xưa" và đưa một chiếc máy tính Windows 98 vào đời thực bằng mã:

```
public class Windows98Machine {}
```

Nhưng một chiếc máy tính thì có ích gì nếu không có màn hình và bàn phím? Hãy thêm một cái vào hàm tạo của chúng ta để mỗi _Windows98Computer_ chúng ta khởi tạo đều đi kèm với một _Monitor_ và một _StandardKeyboard_

```
public class Windows98Machine {

    private final StandardKeyboard keyboard;
    private final Monitor monitor;

    public Windows98Machine() {
        monitor = new Monitor();
        keyboard = new StandardKeyboard();
    }

}
```

Đoạn mã này sẽ hoạt động, và chúng ta có thể tự do sử dụng `StandardKeyboard` và `Monitor` bên trong lớp `Windows98Computer` của mình.

Vấn đề đã được giải quyết ư? Chưa hẳn. Bằng cách khai báo `StandardKeyboard` và `Monitor` bằng từ khóa `new`, **chúng ta đã kết nối chặt chẽ ba lớp này lại với nhau.**

Điều này không chỉ làm cho `Windows98Computer` của chúng ta khó kiểm thử, mà chúng ta còn mất khả năng thay thế lớp `StandardKeyboard` bằng một loại khác nếu cần. Và chúng ta cũng bị "mắc kẹt" với lớp `Monitor` của mình.

Hãy tách rời máy tính của chúng ta khỏi `StandardKeyboard` bằng cách thêm một giao diện `Keyboard` tổng quát hơn và sử dụng nó trong lớp của chúng ta:

```
public interface Keyboard { }
```

```
public class Windows98Machine{

    private final Keyboard keyboard;
    private final Monitor monitor;

    public Windows98Machine(Keyboard keyboard, Monitor monitor) {
        this.keyboard = keyboard;
        this.monitor = monitor;
    }
}
```

Ở đây, chúng ta đang sử dụng **mẫu thiết kế dependency injection (tiêm phụ thuộc)** để tạo điều kiện thuận lợi cho việc thêm phụ thuộc `Keyboard` vào lớp `Windows98Machine`.

Hãy cùng sửa đổi lớp `StandardKeyboard` của chúng ta để nó triển khai giao diện `Keyboard`, giúp nó phù hợp để được tiêm vào lớp `Windows98Machine`:

```
public class StandardKeyboard implements Keyboard { }
```

Bây giờ, các lớp của chúng ta đã được tách rời và giao tiếp thông qua trừu tượng hóa `Keyboard`. Nếu muốn, chúng ta có thể dễ dàng thay đổi loại bàn phím trong máy của mình bằng một triển khai khác của giao diện. Chúng ta có thể áp dụng nguyên tắc tương tự cho lớp `Monitor`.

Tuyệt vời! Chúng ta đã tách rời các phụ thuộc và có thể tự do kiểm thử `Windows98Machine` của mình với bất kỳ framework kiểm thử nào chúng ta chọn.

## 8. Conclusion - Kết luận

Trong bài viết này, chúng ta đã đi sâu vào **các nguyên lý SOLID trong thiết kế hướng đối tượng.**

Chúng ta đã bắt đầu với một chút lịch sử của SOLID và những lý do mà các nguyên lý này tồn tại.

Từng chữ một, chúng ta đã phân tích ý nghĩa của mỗi nguyên lý với một ví dụ mã nhanh minh họa việc vi phạm. Sau đó, chúng ta đã thấy cách sửa chữa mã và làm cho nó tuân thủ các nguyên lý SOLID.

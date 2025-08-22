---
title: "Hiểu cơ chế Re-render của React để tối ưu cho đúng"
date: 2025-08-06 21:38:00  +0700
categories: [Học tập]
tags: [Học tập]
---

---

# Hiểu cơ chế Re-render của React để tối ưu cho đúng

## Trong bài viết này, chúng ta sẽ cùng xem xét 1 Bug hiệu năng phổ biến, hiểu về cơ chế re-render của Recat và khám phá một kỹ thuật tối ưu chỉ bằng cấu trúc lại component

Hãy xem xét một component App lớn, chứa nhiều thành phần con rất "chậm" và phức tạp. Nhiệm vụ bây giờ cần thêm một nút bấm và khi bấm vào sẽ mở dialog

```
const App = () => {
  // rất nhiều code ở đây
  return (
    <div className="layout">
      {/* Nút bấm mở dialog sẽ nằm đâu đó ở đây */}
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

Đây là một cách làm rất đơn giản:

```
const App = () => {
  // Thêm state để quản lý dialog
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="layout">
      {/* Thêm nút bấm */}
      <Button onClick={() => setIsOpen(true)}>
        Open dialog
      </Button>

      {/* Thêm dialog */}
      {isOpen && <ModalDialog onClose={() => setIsOpen(false)} />}

      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};
```

Và đây là lúc vấn đè xuất hiện. Khi nhấn nút "Open dialog", ứng dụng bị khựng lại một lúc, tại sao lại như vậy?

<p align="center">
  <img src="/assets/images/re-render/1.png" alt="Image title_1" />
</p>

## Re-render lan truyền như thế nào

Để hiểu được vấn đề, chúng ta cần nắm vững quy tắc cỏa bản của re-render trong React

> Khi một component cha re-render, tất cả các component con bên trong nó cũng sẽ bị re-render theo, bất kể props của chúng có thay đổi hay không

<p align="center">
  <img src="/assets/images/re-render/2.png" alt="Image title_1" />
</p>

Trong ví dụ trên, khi chúng ta gọi `setIsOpen(true)`, state của component App thay đổi. Điều này khiến App phải re-render để cập nhật giao diện. Và theo quy tắc trên, nó kéo theo việc re-render toàn bộ các component con của nó: `Button`, `ModalDialog` (nếu có), và quan trọng nhất là cả `VerySlowComponent`, `BunchOfStuff`, `OtherStuffAlsoComplicated`.

Chính việc re-render những component nặng nề không liên quan này đã gây ra tình trạng giật lag. React không bao giờ re-render "ngược lên" cây component, nó chỉ lan truyền "xuống dưới".

## Lầm tưởng phổ biến: "Component re-render khi props thay đổi"

Đây là một trong những hiểu lầm phổ biến nhất trong cộng đồng React. Nhiều người tin rằng việc một component có re-render hay không phụ thuộc vào props của nó. **Điều này không đúng với các component thông thường.**

Nếu không có state update nào được kích hoạt, việc bạn thay đổi props một cách "thủ công" (ví dụ qua một biến let thông thường) sẽ không gây ra bất kỳ hiệu ứng nào. React sẽ không "để ý" đến sự thay đổi đó.

Việc so sánh props chỉ thực sự có ý nghĩa trong một trường hợp duy nhất: khi component con được bọc trong React.memo. Chỉ khi đó, React mới dừng lại, so sánh props cũ và mới. Nếu không có gì thay đổi, nó sẽ bỏ qua việc re-render component đó và các con của nó.

Nhưng trong trường hợp của chúng ta, việc dùng React.memo là không cần thiết và phức tạp hóa vấn đề.

## Giải pháp đơn giản

Thay vì cố gắng ngăn chặn sự lan truyền của re-render bằng memoization, tại sao chúng ta không giới hạn phạm vi ảnh hưởng của nó ngay từ đầu?

Nhìn lại code, chúng ta thấy rằng state isOpen chỉ thực sự được dùng bởi Button và ModalDialog. Các component "chậm" kia hoàn toàn không quan tâm đến nó.

Vậy giải pháp rất đơn giản: **Tách state và các component phụ thuộc vào state đó ra một component nhỏ hơn.**

```
// 1. Tạo một component mới chứa state và logic
const ButtonWithModalDialog = () => {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <Button onClick={() => setIsOpen(true)}>
        Open dialog
      </Button>
      {isOpen && <ModalDialog onClose={() => setIsOpen(false)} />}
    </>
  );
};

// 2. Sử dụng component mới này trong App
const App = () => {
  return (
    <div className="layout">
      <ButtonWithModalDialog />
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
};

```

Bây giờ, khi nhấn nút, state update chỉ xảy ra bên trong `ButtonWithModalDialog`. Chỉ có component này và các con của nó (là `Button` và `ModalDialog`) re-render. Component App không còn state, nó không re-render, và do đó, các component "chậm" được an toàn.

Kết quả? Dialog xuất hiện tức thì. Chúng ta vừa giải quyết một bug hiệu năng nghiêm trọng chỉ bằng một kỹ thuật composition đơn giản!

## Kết luận

Tối ưu hiệu năng trong React không phải lúc nào cũng là cuộc chiến với `useMemo` và `useCallback`. Thường thì, vũ khí mạnh mẽ nhất của chúng ta lại chính là những nguyên tắc cơ bản về **composition**.

Trước khi vội vã bọc mọi thứ trong memoization, hãy tự hỏi: "_State này có đang được đặt ở đúng chỗ không?_".

**Những điểm cần nhớ**

1. **Nguồn gốc của mọi re-render là state update** ( Hoặc context update, props từ parent đã re-render)
2. Khi 1 component re-render, **tất cả các con của nó cũng re-render theo**
3. Lầm tưởng "component re-render khi props thay đổi" là **sai**. Nó chỉ đúng với React.Memo
4. Kỹ thuật tạo 1 component mới để bao bọc state là cách cực kỳ hiệu quả để ngăn chặn re-render không cần thiết

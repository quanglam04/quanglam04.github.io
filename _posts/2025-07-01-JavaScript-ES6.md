---
title: "Một số tính năng JavaScript ES6 mà bạn cần biết"
date: 2025-07-01 21:38:00  +0700
categories: [Programming, JavaScript]
tags: [javascript, es6, tips]
---

---

## Một số tính năng JavaScript ES6 bạn cần biết

### 1. Arrow Function

Arrow function là cú pháp viết ngắn gọn cho function trong JavaScript

```
// Cách viết thường
const add = function (a,b) {
    return a+b;
};

// Arrow function
const add = (a,b) => a+b;
console.log(add(2,3)) //5
```

### 2. Template Literals

Template literals cho phép chèn biến và biểu thức trực tiếp vào chuỗi bằng cú pháp ${}

```
const name = "Lam";
const address = "Viet Nam"
console.log(`Tôi tên là ${name}, tôi ở ${address}`) // Output: Tôi tên là Lam, tôi ở Viet Nam
```

### 3. Destructuring Assignment

Cú pháp này cho phép trích xuất giá trị từ object hoặc array và gán vào biến một cách ngắn gọn

```
const person = { name: "Lam", age: 22 };
const { name, age } = person;

console.log(name); // Lam
console.log(age);  // 22
```

> Node: khi destructuring object, tên biến phải đúng với key

### 4. Spead Operator (...)

Dùng để sao chép, gộp array hoặc object một cách dễ dàng

```
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4];

console.log(arr2); // [1, 2, 3, 4]
```

### 5. Rest Parameter

Thu thập nhiều đối số thành một mảng, rất cần thiết khi hàm nhận số lượng tham số linh hoạt

```
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

console.log(sum(1, 2, 3)); // 6
```

### 6. Async/Await

Giúp xử lý bất đồng bộ dễ đọc hơn thay vì dùng .then() trong Promise

```
async function fetchData() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  console.log(data);
}
```

### 7. Map & Set

- **Map**: Cấu trúc dữ liệu lưu key-value, hỗ trợ key dạng object
- **Set**: Lưu danh sách giá trị không trùng lặp

```
// Map
const map = new Map();
map.set('name', 'Lam');
console.log(map.get('name')); // Lam

// Set
const set = new Set([1, 2, 2, 3]);
console.log([...set]); // [1, 2, 3]
```

### 8. Default Parameters

Cho phép khai báo giá trị mặc định cho tham số hàm

```
function greet(name = "Bạn") {
  console.log(`Xin chào, ${name}!`);
}

greet();        // Xin chào, Bạn!
greet("Lam");   // Xin chào, Lam!
```

### 9. Modules (import/export)

Giúp chia nhỏ code thành các module có thể tái sử dụng

```
// greet.js
export function greet(name) {
  console.log(`Hello, ${name}!`);
}

// main.js
import { greet } from './greet.js';
greet("Lam");
```

### 10. map() Method

Dùng để tạo ra mảng mới từ mảng gốc thông qua một hàm chuyển đổi

```
const numbers = [1, 2, 3];
const squared = numbers.map(n => n * n);

console.log(squared); // [1, 4, 9]
```

### 11. filter() Method

Dùng để lọc các phần tử trong mảng theo điều kiện

```
const numbers = [1, 2, 3, 4];
const evens = numbers.filter(n => n % 2 === 0);

console.log(evens); // [2, 4]
```

---
title: "10 nguyên tắc Clean Code phổ biến"
date: 2025-08-25 01:17:00  +0700
categories: [Programming, BestPractices]
tags: [clean code, coding style, best practice]
---

---


## 1. Return sớm nhất có thể hay đơn giản là tránh việc để quá nhiều điều kiện lồng nhau

```
❌ Không nên
function isEligibleForDiscount(user) {
  if (user) {
    if (user.age >= 18) {
      if (user.isMember) {
        return true;
      }
    }
  }
  return false;
}

✅ Clean

function isEligibleForDiscount(user) {
  if (!user) return false;
  if (user.age < 18) return false;
  if (!user.isMember) return false;

  return true;
}
```

## 2. Để tiền tố Is, Has, Can đối với những function hoặc biến boolean để cho người dùng để hiểu nhất và chỉ nên trả về boolean

```
// Kiểm tra xem người dùng có đủ điều kiện để tham gia một chương trình khuyến mãi không
function isUserEligible(user) {
  // Giả sử người dùng đủ điều kiện nếu họ trên 18 tuổi và có tài khoản hoạt động
  return user.age >= 18 && user.isActive;
}

// Kiểm tra xem người dùng có quyền thực hiện một hành động nào đó hay không
function hasPermission(user, action) {
  // Giả sử nếu vai trò của người dùng là "admin", họ có tất cả các quyền
  if (user.role === "admin") return true;

  // Nếu không phải admin, kiểm tra quyền cụ thể của người dùng
  return user.permissions.includes(action);
}

// Kiểm tra xem người dùng có thể truy cập một tài nguyên nào đó hay không
function canAccessResource(user, resource) {
  // Giả sử tài nguyên chỉ có thể truy cập nếu nó không yêu cầu quyền đặc biệt
  // hoặc người dùng có quyền "access" đối với tài nguyên đó
  return !resource.requiresPermission || hasPermission(user, "access");
}

// Tên biến boolean rõ ràng -> Không cần comment vẫn hiểu được
const isActive = true;
const hasDiscount = false;
const canEdit = true;
```

## 3. Nên để tên hàm nó nói lên tác dụng của nó mà không cần phải comment

```
// Thay vì đặt tên không rõ ràng như handleClick hoặc processData
// thì ta đặt tên mô tả rõ ràng chức năng của hàm.

// Kiểm tra xem email có hợp lệ không
function isValidEmail(email) {
  const emailRegex = /^[^s@]+@[^s@]+.[^s@]+$/;
  return emailRegex.test(email);
}

// Tính tổng số tiền thanh toán đã bao gồm thuế
function calculateTotalWithTax(amount, taxRate) {
  return amount + amount * taxRate;
}

// Chuyển chữ cái đầu của mỗi từ trong chuỗi thành chữ hoa
function capitalizeWords(sentence) {
  return sentence
    .split(" ")
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
    .join(" ");
}

// Kiểm tra xem một số có phải là số nguyên tố hay không
function isPrime(number) {
  if (number <= 1) return false;
  for (let i = 2; i <= Math.sqrt(number); i++) {
    if (number % i === 0) return false;
  }
  return true;
}
```

## 4. Dùng ESLint + Prettier cho dự án

```
Vừa sử dụng để config (ví dụ để check xem đã bỏ hết console chưa trước khi build để deploy) + prettier để format code
```

## 5. Mỗi function chỉ nên thực thi đúng nhiệm vụ của nó (và nên tách ra nhiều function)

```
❌ Không nên
function processOrder(order) {
  // Kiểm tra tính hợp lệ của đơn hàng
  if (!order || !order.items || order.items.length === 0) {
    console.log("Đơn hàng không hợp lệ");
    return;
  }

  // Tính tổng giá trị đơn hàng
  let total = 0;
  order.items.forEach((item) => {
    total += item.price * item.quantity;
  });

  // Kiểm tra nếu đủ hàng trong kho
  if (!checkInventory(order.items)) {
    console.log("Không đủ hàng trong kho");
    return;
  }

  // Tiến hành thanh toán
  processPayment(total);
  console.log("Đơn hàng đã được xử lý thành công!");
}

✅ Clean
function validateOrder(order) {
  return order && order.items && order.items.length > 0;
}

function calculateTotal(order) {
  return order.items.reduce(
    (total, item) => total + item.price * item.quantity,
    0
  );
}

function checkInventory(items) {
  // Logic kiểm tra hàng trong kho
  return items.every((item) => item.stock >= item.quantity);
}

function processPayment(total) {
  // Logic thanh toán
  console.log(`Đã thanh toán thành công số tiền ${total}`);
}

function processOrder(order) {
  if (!validateOrder(order)) {
    console.log("Đơn hàng không hợp lệ");
    return;
  }
}
```

## 6. Hạn chế dùng magic value hay còn gọi là giá trị mà không có mô tả. Hãy tạo constant

```
❌ Không nên
function calculateDiscount(price) {
  return price * 0.15; // 0.15 là magic value
}

function getShippingFee(weight) {
  return weight > 5 ? 50 : 30; // 5, 50, 30 là magic value
}

✅ Clean
const DISCOUNT_RATE = 0.15;
const MAX_FREE_SHIPPING_WEIGHT = 5; // cân nặng tối đa được miễn phí vận chuyển
const HIGH_WEIGHT_FEE = 50; // phí vận chuyển cho hàng nặng
const LOW_WEIGHT_FEE = 30; // phí vận chuyển cho hàng nhẹ

function calculateDiscount(price) {
  return price * DISCOUNT_RATE;
}

function getShippingFee(weight) {
  return weight > MAX_FREE_SHIPPING_WEIGHT ? HIGH_WEIGHT_FEE : LOW_WEIGHT_FEE;
}
```

## 7. Viết code dễ hiểu 

```
❌ Không nên
const userStatus = isAdmin && isVerified ? "Admin" : isGuest ? "Guest" : "User";

✅ Clean
let userStatus;
if (isAdmin && isVerified) {
  userStatus = "Admin";
} else if (isGuest) {
  userStatus = "Guest";
} else {
  userStatus = "User";
}
```

## 8. Tuần theo nguyên tắc SOLID

## 9. Giữ sự nhất quán về kiểu mã hóa

```
❌ Không nhất quán: Trộn giữa camelCase và snake_case
let user_name = "John";
function GetUserAge(user_age) {
  return user_age;
}

✅ Nhất quán: Dùng camelCase
let userName = "John";
function getUserAge(userAge) {
  return userAge;
}
```

## 10. Đặt tên biến và hàm rõ ràng

```
❌ Không nên
let x = 500;
let y = 200;
let z = x + y;

✅ Clean
let itemPrice = 500;
let shippingFee = 200;
let totalCost = itemPrice + shippingFee;
```


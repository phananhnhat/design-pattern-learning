# Strategy Pattern

Là mẫu thiết kế hành vi (behavioral design pattern)

Ý tưởng:
- Định nghĩa nhiều thuật toán/chiến lược khác nhau cho một tác vụ.
- Cho phép hoán đổi (switch) chiến lược đó lúc runtime, mà không cần thay đổi code của client.

👉 Nói dễ hiểu: thay vì viết nhiều if/else để chọn cách làm, bạn bọc mỗi cách làm thành một strategy riêng, rồi inject vào lúc cần.

Ví dụ: Hook trong React:

```javascript
const shippingStrategies = {
    standard: (order) => order.weight * 1000,
    express: (order) => order.weight * 2000 + 5000,
    free: () => 0,
};

function calculateShipping(order, strategy) {
    return strategy(order);
}

const order = { weight: 3 };

console.log(calculateShipping(order, shippingStrategies.standard)); // 3000
console.log(calculateShipping(order, shippingStrategies.express));  // 11000
console.log(calculateShipping(order, shippingStrategies.free));     // 0
```

👉 Ưu điểm: bạn có thể thêm strategy mới mà không cần sửa code calculateShipping.



```javascript
function preOrderPrice(originalPrice) { return originalPrice * 0.2; }

function promotionPrice(originalPrice) { return originalPrice <= 200 ? originalPrice * 0.1 : originalPrice - 30; }

function blackFridayPrice(originalPrice) { return originalPrice <= 200 ? originalPrice * 0.2 : originalPrice - 50; }

function defaultPrice(originalPrice) { return originalPrice; }

const getPriceStrategies = {
    preOrder: preOrderPrice,
    promotion: promotionPrice,
    blackFriday: blackFridayPrice,
    default: defaultPrice,
} 

function getPrice(originalPrice, typePromotion) {
    return getPriceStrategies[typePromotion](originalPrice); 
}
```
✅ Lợi ích (giống textbook Strategy pattern)
Muốn thêm chiến lược mới (vd: loyalCustomerPrice) → chỉ cần viết thêm 1 function và đăng ký trong getPriceStrategies.
Không phải sửa getPrice → tuân thủ Open/Closed Principle (OCP). Dễ test từng strategy độc lập.

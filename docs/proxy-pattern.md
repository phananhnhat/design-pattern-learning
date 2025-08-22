# Proxy Pattern

Proxy Pattern là một structural design pattern (mẫu thiết kế cấu trúc), trong đó một đối tượng proxy (đại diện) đứng giữa client và đối tượng thực sự (real subject).

Proxy giúp:
- Kiểm soát quyền truy cập đối tượng thật.
- Thêm logic bổ sung (caching, logging, lazy loading, security…).
- Không làm thay đổi code của đối tượng thật.

👉 Nói đơn giản: Proxy là "người trung gian".

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★★☆)

```javascript
// Proxy cơ bản (ES6 Proxy)

const user = {
    name: "Alice",
    age: 25,
};

const proxyUser = new Proxy(user, {
    get(target, prop) {
        console.log(`Truy cập thuộc tính: ${prop}`);
        return target[prop];
    },
    set(target, prop, value) {
        console.log(`Gán giá trị ${value} cho ${prop}`);
        target[prop] = value;
        return true;
    },
});

// Sử dụng
console.log(proxyUser.name);  // log: Truy cập thuộc tính: name
proxyUser.age = 30;           // log: Gán giá trị 30 cho age
console.log(proxyUser.age);
```

```javascript
// Proxy Pattern thủ công (function wrapper)

function realFetch(url) {
    console.log("Fetching data from:", url);
    return `Dữ liệu từ ${url}`;
}

function fetchProxy(url) {
    console.log("Proxy kiểm tra cache...");
    // Ví dụ: không cho fetch ngoài domain
    if (!url.startsWith("https://myapi.com")) {
        throw new Error("Không được phép truy cập URL ngoài hệ thống!");
    }
    return realFetch(url);
}

// Client
try {
    console.log(fetchProxy("https://myapi.com/users"));
    console.log(fetchProxy("https://google.com")); // sẽ bị chặn
} catch (e) {
    console.error(e.message);
}
```
### So sánh Proxy Pattern vs Decorator Pattern
| Tiêu chí                | Proxy Pattern                                      | Decorator Pattern                                |
|--------------------------|---------------------------------------------------|-------------------------------------------------|
| **Mục đích chính**       | Kiểm soát quyền truy cập, lazy loading, caching…  | Bổ sung hành vi mới cho đối tượng               |
| **Cách triển khai**      | Thay mặt đối tượng thật, có thể chặn hoặc forward | Gói đối tượng và thêm logic trước/sau           |
| **Quan hệ với đối tượng**| Có thể từ chối hoặc trì hoãn lời gọi               | Luôn forward lời gọi đến đối tượng gốc          |
| **Tính minh bạch**       | Client thường **không biết** đang dùng Proxy      | Client có thể kết hợp nhiều Decorator linh hoạt |
| **Ví dụ thực tế**        | Security proxy, Virtual proxy, Caching proxy      | Logging, Mã hóa dữ liệu, Thêm topping cho đồ ăn |

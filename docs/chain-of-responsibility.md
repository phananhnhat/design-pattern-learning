# Chain of Responsibility

Là mẫu thiết kế hành vi (behavioral design pattern)
Mục đích: tránh gắn cứng người gửi request với người xử lý request → nhiều “handler” được xếp thành chuỗi, request sẽ đi qua chuỗi cho đến khi có handler xử lý nó

Ý tưởng: Có nhiều đối tượng có khả năng xử lý request. Thay vì gọi trực tiếp 1 đối tượng, request đi qua chain cho đến khi 1 đối tượng nhận trách nhiệm.

👉 Hình dung: bạn nộp đơn xin nghỉ → gửi cho team lead, nếu lead không duyệt thì chuyển manager, nếu manager không duyệt thì chuyển HR.

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★★★)

Rất hay gặp trong framework, logging, event handling, middleware

```javascript
function createHandler(check, next) {
    return (req) => {
        if (check(req)) {
            console.log("✅ Handled by", check.name);
        } else if (next) {
            next(req);
        } else {
            console.log("❌ No handler could process:", req);
        }
    };
}

// Các handler cụ thể
function isNumber(req) { return typeof req === "number"; }
function isString(req) { return typeof req === "string"; }
function isArray(req) { return Array.isArray(req); }

// Tạo chain
const handler = createHandler(isNumber, createHandler(isString, createHandler(isArray, null)));

handler(42);        // ✅ Handled by isNumber
handler("hello");   // ✅ Handled by isString
handler([1, 2, 3]); // ✅ Handled by isArray
handler({});        // ❌ No handler could process
```
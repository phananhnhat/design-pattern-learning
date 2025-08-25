# Iterator Pattern

Là một behavioral pattern (mẫu hành vi).
Mục đích: Định nghĩa một ngôn ngữ nhỏ (DSL) với cú pháp và ngữ pháp riêng, và xây dựng bộ thông dịch (interpreter) để thực thi các biểu thức trong ngôn ngữ đó.

Ý tưởng:
- Xác định ngữ pháp (grammar) của ngôn ngữ.
- Biểu diễn mỗi quy tắc ngữ pháp bằng một class hoặc hàm.
- Dùng cây biểu thức (AST) để phân tích và đánh giá.

Mức độ khó (★★★★☆) | Mức độ phổ biến (★★☆☆☆)

👉 Hình dung: giống như bạn viết một “mini-language” cho máy tính hiểu. Ví dụ: ngôn ngữ Regex, SQL, hoặc một DSL để filter dữ liệu.

Ứng dụng:
- Regex engine
- SQL parser
- Rule engine (ví dụ: "age > 18 AND country = 'VN'")
- Mini scripting trong ứng dụng

```javascript
// Context: giá trị đầu vào
const context = {
    true: true,
    false: false
};

// Interpreter functions
function interpret(expr) {
    const tokens = expr.split(" ");
    let result = context[tokens[0]];

    for (let i = 1; i < tokens.length; i += 2) {
        const operator = tokens[i];
        const value = context[tokens[i + 1]];
        if (operator === "AND") {
            result = result && value;
        } else if (operator === "OR") {
            result = result || value;
        }
    }
    return result;
}

// Demo
console.log(interpret("true AND false")); // false
console.log(interpret("true AND false OR true")); // true
// 👉 Đây là interpreter rất đơn giản cho logic expression.
```


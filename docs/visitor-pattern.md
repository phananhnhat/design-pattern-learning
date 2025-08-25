# Visitor Pattern

Là mẫu thiết kế hành vi (behavioral design pattern)
Mục đích: tách riêng thuật toán xử lý khỏi cấu trúc đối tượng.

Ý tưởng:
- Có một cấu trúc phức tạp (ví dụ: File, Folder, Tree, ...).
- Thay vì nhét mọi logic vào class/object/module đó, ta viết các “visitor” bên ngoài.
- Object chỉ cần “chấp nhận” (accept) visitor, rồi visitor sẽ quyết định xử lý.

👉 Hình dung: giống như bạn có nhiều loại phần tử (hoa, lá, quả), và bạn mời người thăm quan (visitor) đến → mỗi visitor sẽ “làm gì đó” khác nhau với hoa/lá/quả.

Mức độ khó (★★★★☆) | Mức độ phổ biến (★★★☆☆)

Dùng nhiều trong framework, khi cần định nghĩa luồng chuẩn (vd: hook của React, lifecycle của Servlet, JUnit test).

```javascript
function file(name, size) {
    return {
        type: "file",
        name,
        size,
        accept: (visitor) => visitor.file(name, size)
    };
}

function folder(name, children) {
    return {
        type: "folder",
        name,
        children,
        accept: (visitor) => visitor.folder(name, children)
    };
}

// Visitors
const sizeVisitor = {
    file: (name, size) => size,
    folder: (name, children) =>
        children.reduce((sum, c) => sum + c.accept(sizeVisitor), 0)
};

const printVisitor = {
    file: (name) => console.log("📄 " + name),
    folder: (name, children) => {
        console.log("📁 " + name);
        children.forEach(c => c.accept(printVisitor));
    }
};

// --- Demo ---
const tree = folder("root", [
    file("a.txt", 10),
    folder("sub", [ file("b.txt", 20), file("c.txt", 30) ])
]);

console.log("Total size:", tree.accept(sizeVisitor)); // 60
tree.accept(printVisitor);
```
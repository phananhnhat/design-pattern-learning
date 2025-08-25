# Memento Pattern

Là mẫu thiết kế hành vi (behavioral design pattern).
Để lưu trữ và khôi phục trạng thái của một object mà không để lộ chi tiết nội tại của nó

Ý tưởng: Có 3 vai trò:
- Originator: đối tượng có trạng thái cần lưu.
- Memento: snapshot của trạng thái.
- Caretaker: nơi giữ memento để khôi phục khi cần.

👉 Hình dung: giống như Undo/Redo trong editor → mỗi lần thay đổi là một memento.

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★★☆)

Phổ biến editor, game, transaction rollback

```javascript
function createEditor() {
    let content = "";

    return {
        type: (text) => { content += text; },
        getContent: () => content,
        save: () => content,             // tạo memento
        restore: (memento) => { content = memento; } // khôi phục
    };
}

const editor = createEditor();
const history = [];

editor.type("Hello ");
history.push(editor.save()); // lưu state

editor.type("World!");
console.log(editor.getContent()); // Hello World!

editor.restore(history[0]); // undo
console.log(editor.getContent()); // Hello
// 👉 Rất đơn giản: save() trả về memento, restore() phục hồi.
```

Hạn chế: Mỗi lần save(), toàn bộ nội dung của editor (content) được lưu nguyên xi vào history => tốn bộ nhớ.

Giải pháp thường dùng trong thực tế:
- Lưu diff thay vì snapshot đầy đủ
- Giới hạn số lượng memento
- Nén dữ liệu

Trong thực tế:
- Editor, IDE, Photoshop thường dùng Command Pattern cho Undo/Redo.
- Game checkpoint hoặc transaction rollback lại thiên về Memento (lưu nguyên state).
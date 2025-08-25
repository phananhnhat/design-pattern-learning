# Command Pattern

Là mẫu thiết kế hành vi (behavioral design pattern).
Để đóng gói một hành động (operation) thành một object để có thể truyền, hoàn tác, lưu trữ, xếp hàng,...

Ý tưởng: Có 3 vai trò:
- Command: interface chung (có execute(), undo()).
- ConcreteCommand: cài đặt cụ thể hành động.
- Invoker: gọi lệnh (không biết chi tiết).
- Receiver: đối tượng thực sự thực hiện hành động.

👉 Hình dung: bạn bấm nút remote TV → nút đó là Command, TV là Receiver, còn bạn là Invoker.

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★★★)

Cực kỳ phổ biến (Undo/Redo, menu action, macro, event handler)

```javascript
// Receiver
function createLight() {
    let isOn = false;
    return {
        on: () => { isOn = true; console.log("💡 Light ON"); },
        off: () => { isOn = false; console.log("💡 Light OFF"); }
    };
}

// Command factory
function createCommand(execute, undo) {
    return { execute, undo };
}

// Invoker
function createRemote() {
    let history = [];
    return {
        press: (command) => {
            command.execute();
            history.push(command);
        },
        undo: () => {
            const command = history.pop();
            if (command) command.undo();
        }
    };
}

// --- Demo ---
const light = createLight();
const lightOn = createCommand(light.on, light.off);
const lightOff = createCommand(light.off, light.on);

const remote = createRemote();
remote.press(lightOn);  // 💡 Light ON
remote.press(lightOff); // 💡 Light OFF
remote.undo();          // 💡 Light ON (undo off)
```

```javascript
// Receiver (Editor)
function createEditor() {
  let content = "";

  return {
    insert: (text) => { content += text; },
    delete: (count) => {
      const removed = content.slice(-count);
      content = content.slice(0, -count);
      return removed;
    },
    getContent: () => content
  };
}

// Command factory
function InsertCommand(editor, text) {
  return {
    execute: () => editor.insert(text),
    undo: () => editor.delete(text.length)
  };
}

function DeleteCommand(editor, count) {
  let removedText = "";
  return {
    execute: () => { removedText = editor.delete(count); },
    undo: () => editor.insert(removedText)
  };
}

// Invoker (Command Manager)
function createCommandManager() {
  const history = [];
  const redoStack = [];

  return {
    execute: (command) => {
      command.execute();
      history.push(command);
      redoStack.length = 0; // clear redo
    },
    undo: () => {
      const cmd = history.pop();
      if (cmd) {
        cmd.undo();
        redoStack.push(cmd);
      }
    },
    redo: () => {
      const cmd = redoStack.pop();
      if (cmd) {
        cmd.execute();
        history.push(cmd);
      }
    }
  };
}

const editor = createEditor();
const manager = createCommandManager();

manager.execute(InsertCommand(editor, "Hello "));
manager.execute(InsertCommand(editor, "World!"));
console.log(editor.getContent()); // Hello World!

manager.undo();
console.log(editor.getContent()); // Hello 

manager.redo();
console.log(editor.getContent()); // Hello World!

manager.execute(DeleteCommand(editor, 6));
console.log(editor.getContent()); // Hello

manager.undo();
console.log(editor.getContent()); // Hello World!
```
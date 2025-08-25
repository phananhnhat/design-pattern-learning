# Mediator Pattern

Là mẫu thiết kế hành vi (behavioral design pattern).
Giảm sự phụ thuộc trực tiếp (coupling) giữa các object bằng cách để chúng giao tiếp thông qua một "trung gian" (mediator).

Ý tưởng:
- Thay vì các component gọi lẫn nhau, chúng chỉ gọi mediator.
- Mediator quyết định chuyển tiếp hoặc điều phối logic.

👉 Hình dung: giống như hệ thống chat: thay vì user A gửi tin nhắn trực tiếp đến user B, A gửi cho server (mediator), rồi server chuyển đến B.

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★★☆)

Phổ biến trong GUI (button, textbox, dialog) và hệ thống messaging

```javascript
// Mediator
function createChatRoom() {
    const users = {};

    return {
        register: (user) => { users[user.name] = user; user.chatroom = this; },
        send: (message, from, to) => {
            if (to) {
                console.log(`[Private] ${from} → ${to}: ${message}`);
            } else {
                console.log(`[Public] ${from}: ${message}`);
            }
        }
    };
}

// User
function createUser(name, chatroom) {
    return {
        name,
        chatroom,
        send: function(message, to) {
            this.chatroom.send(message, this.name, to);
        }
    };
}

// --- Demo ---
const chatroom = createChatRoom();
const alice = createUser("Alice", chatroom);
const bob = createUser("Bob", chatroom);

chatroom.register(alice);
chatroom.register(bob);

alice.send("Hello everyone!");
bob.send("Hi Alice!", "Alice");
// 👉 Các user không nói chuyện trực tiếp, mà thông qua chatroom
```

```javascript
// Giả sử có 3 xe (module, objcet), thay vì tự quyết định thì chúng sẽ hỏi đèn giao thông (mediator).

function trafficLight() {
    let color = "red";
    return {
        setColor: (c) => { color = c; },
        canGo: () => color === "green"
    };
}

const light = trafficLight();

function car(name) {
    return {
        move: () => {
            if (light.canGo()) console.log(name, "🚗 đi");
            else console.log(name, "⛔ dừng lại");
        }
    };
}

const carA = car("Xe A");
const carB = car("Xe B");

carA.move(); // Xe A ⛔ dừng lại
carB.move(); // Xe B ⛔ dừng lại

light.setColor("green");
carA.move(); // Xe A 🚗 đi
carB.move(); // Xe B 🚗 đi

// Các car không nói chuyện trực tiếp với nhau.
// Tất cả chỉ hỏi mediator (trafficLight) để quyết định hành vi.
// Mediator (đèn giao thông) giữ logic điều phối.
```

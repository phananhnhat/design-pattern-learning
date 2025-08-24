# State Pattern

Là một behavioral pattern (mẫu hành vi).

Ý tưởng: cho phép một đối tượng thay đổi hành vi của nó khi trạng thái bên trong thay đổi. Đối tượng sẽ có vẻ như đã thay đổi lớp của nó.

👉 Thay vì dùng `if/else` hoặc `switch` khổng lồ để kiểm tra trạng thái, ta tạo ra các đối tượng "trạng thái" riêng biệt và object chính sẽ ủy quyền hành vi cho trạng thái hiện tại.

👉 Giống như máy bán hàng:
- Khi có tiền → cho phép mua hàng.
- Khi hết tiền → yêu cầu nạp tiền.
- Khi hết hàng → thông báo hết hàng.
=> Hành vi khác nhau tùy thuộc trạng thái.

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★☆☆)

### Khi nào dùng?
- Khi hành vi của một đối tượng phụ thuộc vào trạng thái và phải thay đổi lúc runtime.
- Khi có nhiều câu lệnh điều kiện lớn (`if/else`, `switch`) để xử lý các trạng thái khác nhau.
- Khi muốn thêm trạng thái mới mà không cần thay đổi code hiện có.

```javascript
function createAudioPlayer() {
  // Các state dưới dạng object literal
  const states = {
    playing: {
      play: () => console.log("Already playing!"),
      pause: (player) => {
        console.log("Pausing...");
        player.state = states.paused;
      },
      stop: (player) => {
        console.log("Stopping...");
        player.state = states.stopped;
      }
    },
    paused: {
      play: (player) => {
        console.log("Resuming play...");
        player.state = states.playing;
      },
      pause: () => console.log("Already paused!"),
      stop: (player) => {
        console.log("Stopping from pause...");
        player.state = states.stopped;
      }
    },
    stopped: {
      play: (player) => {
        console.log("Starting play...");
        player.state = states.playing;
      },
      pause: () => console.log("Can't pause, already stopped."),
      stop: () => console.log("Already stopped!")
    }
  };

  // Context
  const player = {
    state: states.stopped,
    play() { this.state.play(this); },
    pause() { this.state.pause(this); },
    stop() { this.state.stop(this); }
  };

  return player;
}

// --- Sử dụng
const player = createAudioPlayer();
player.play();   // Starting play...
player.pause();  // Pausing...
player.play();   // Resuming play...
player.stop();   // Stopping...
player.pause();  // Can't pause, already stopped.
```

Nếu ko dùng State Pattern thì sẽ như này.
```javascript
function createAudioPlayer_NoStatePattern() {
  let state = "stopped";

  return {
    play() {
      if (state === "stopped") {
        console.log("Starting play...");
        state = "playing";
      } else if (state === "paused") {
        console.log("Resuming play...");
        state = "playing";
      } else if (state === "playing") {
        console.log("Already playing!");
      }
    },
    pause() {
      if (state === "playing") {
        console.log("Pausing...");
        state = "paused";
      } else if (state === "paused") {
        console.log("Already paused!");
      } else if (state === "stopped") {
        console.log("Can't pause, already stopped.");
      }
    },
    stop() {
      if (state === "playing" || state === "paused") {
        console.log("Stopping...");
        state = "stopped";
      } else {
        console.log("Already stopped!");
      }
    }
  };
}
```

**Nhược điểm cách cũ:**
- Code phức tạp, khó bảo trì khi có nhiều state
- Vi phạm Open/Closed Principle: phải sửa code cũ khi thêm state mới
- Logic scattered (rải rác) khắp nơi

### Lợi ích của State Pattern
- **Code sạch hơn**: Loại bỏ các khối `if/else` phức tạp
- **Dễ mở rộng**: Thêm state mới mà không cần sửa code cũ
- **Single Responsibility**: Mỗi state class chỉ lo logic của trạng thái đó
- **Polymorphism**: Cùng interface nhưng hành vi khác nhau tùy state


```javascript
// OOP in javascript
class PlayerState {
    play() {}
    pause() {}
    stop() {}
}

class PlayingState extends PlayerState {
    constructor(player) {
        super();
        this.player = player;
    }
    
    play() {
        console.log("Already playing");
    }
    
    pause() {
        console.log("Pausing playback");
        this.player.setState(this.player.pausedState);
    }
    
    stop() {
        console.log("Stopping playback");
        this.player.setState(this.player.stoppedState);
    }
}

class PausedState extends PlayerState {
    constructor(player) {
        super();
        this.player = player;
    }
    
    play() {
        console.log("Resuming playback");
        this.player.setState(this.player.playingState);
    }
    
    pause() {
        console.log("Already paused");
    }
    
    stop() {
        console.log("Stopping playback");
        this.player.setState(this.player.stoppedState);
    }
}

class StoppedState extends PlayerState {
    constructor(player) {
        super();
        this.player = player;
    }
    
    play() {
        console.log("Starting playback");
        this.player.setState(this.player.playingState);
    }
    
    pause() {
        console.log("Can't pause when stopped");
    }
    
    stop() {
        console.log("Already stopped");
    }
}

// --- Context ---
class MusicPlayer {
    constructor() {
        this.playingState = new PlayingState(this);
        this.pausedState = new PausedState(this);
        this.stoppedState = new StoppedState(this);
        
        // Trạng thái ban đầu
        this.currentState = this.stoppedState;
    }
    
    setState(state) { this.currentState = state; }
    
    // Ủy quyền các hành động cho state hiện tại
    play() { this.currentState.play(); }
    pause() { this.currentState.pause(); }
    stop() { this.currentState.stop(); }
}

// --- Sử dụng ---
const player = new MusicPlayer();

player.play();  // Starting playback
player.pause(); // Pausing playback
player.play();  // Resuming playback
player.stop();  // Stopping playback
player.pause(); // Can't pause when stopped
```

### Ví dụ: Java

```java
// --- State Interface ---
interface OrderState {
    void processOrder(Order order);
    void shipOrder(Order order);
    void deliverOrder(Order order);
    void cancelOrder(Order order);
}

// --- Concrete States ---
class NewOrderState implements OrderState {
    @Override
    public void processOrder(Order order) {
        System.out.println("Processing new order...");
        order.setState(new ProcessingState());
    }
    
    @Override
    public void shipOrder(Order order) {
        System.out.println("Cannot ship unprocessed order");
    }
    
    @Override
    public void deliverOrder(Order order) {
        System.out.println("Cannot deliver unprocessed order");
    }
    
    @Override
    public void cancelOrder(Order order) {
        System.out.println("Cancelling new order");
        order.setState(new CancelledState());
    }
}

class ProcessingState implements OrderState {
    @Override
    public void processOrder(Order order) {
        System.out.println("Order is already being processed");
    }
    
    @Override
    public void shipOrder(Order order) {
        System.out.println("Shipping processed order...");
        order.setState(new ShippedState());
    }
    
    @Override
    public void deliverOrder(Order order) {
        System.out.println("Cannot deliver order that hasn't shipped");
    }
    
    @Override
    public void cancelOrder(Order order) {
        System.out.println("Cancelling processed order");
        order.setState(new CancelledState());
    }
}

class ShippedState implements OrderState {
    @Override
    public void processOrder(Order order) {
        System.out.println("Order already processed and shipped");
    }
    
    @Override
    public void shipOrder(Order order) {
        System.out.println("Order already shipped");
    }
    
    @Override
    public void deliverOrder(Order order) {
        System.out.println("Delivering shipped order...");
        order.setState(new DeliveredState());
    }
    
    @Override
    public void cancelOrder(Order order) {
        System.out.println("Cannot cancel shipped order");
    }
}

class DeliveredState implements OrderState {
    @Override
    public void processOrder(Order order) {
        System.out.println("Order already delivered");
    }
    
    @Override
    public void shipOrder(Order order) {
        System.out.println("Order already delivered");
    }
    
    @Override
    public void deliverOrder(Order order) {
        System.out.println("Order already delivered");
    }
    
    @Override
    public void cancelOrder(Order order) {
        System.out.println("Cannot cancel delivered order");
    }
}

class CancelledState implements OrderState {
    // Tất cả methods đều không làm gì hoặc báo lỗi
    @Override
    public void processOrder(Order order) {
        System.out.println("Cannot process cancelled order");
    }
    // ... tương tự cho các method khác
}

// --- Context ---
class Order {
    private OrderState currentState;
    
    public Order() {
        this.currentState = new NewOrderState();
    }
    
    public void setState(OrderState state) {
        this.currentState = state;
    }
    
    // Ủy quyền hành động cho state hiện tại
    public void process() { currentState.processOrder(this); }
    public void ship() { currentState.shipOrder(this); }
    public void deliver() { currentState.deliverOrder(this); }
    public void cancel() { currentState.cancelOrder(this); }
}

// --- Sử dụng ---
public class StateDemo {
    public static void main(String[] args) {
        Order order = new Order();
        
        order.process(); // Processing new order...
        order.ship();    // Shipping processed order...
        order.deliver(); // Delivering shipped order...
        order.cancel();  // Cannot cancel delivered order
    }
}
```
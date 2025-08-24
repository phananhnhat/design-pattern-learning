# Bridge Pattern

Là một structural pattern (mẫu cấu trúc).

Ý tưởng chính:
- Tách abstraction (trừu tượng) ra khỏi implementation (hiện thực) để hai phần có thể phát triển độc lập.
- Tránh việc “class explosion”(quá nhiều class) khi có nhiều biến thể (ví dụ: nhiều hình dạng × nhiều cách vẽ).

👉 **Tóm lại**: "Cầu nối" giữa giao diện (cách dùng) và phần thực thi (cách làm). Thay vì kế thừa, ta dùng composition (kết hợp đối tượng).

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★☆☆)

### Khi nào dùng?
- Khi bạn muốn chia một lớp lớn có nhiều biến thể chức năng (ví dụ: một lớp có thể hoạt động với nhiều loại cơ sở dữ liệu khác nhau).
- Khi bạn cần mở rộng một lớp theo nhiều chiều độc lập. Ví dụ: có nhiều loại `Shape` (Circle, Square) và nhiều cách vẽ `DrawingAPI` (Canvas, SVG). Thay vì tạo các lớp `CanvasCircle`, `SVGCircle`, `CanvasSquare`,... ta tách chúng ra.
- Khi bạn muốn có thể thay đổi implementation lúc runtime.

### Ví dụ: JavaScript

```javascript
function createTV() {
  let on = false;
  return {
    isEnabled: () => on,
    enable: () => { on = true; console.log("TV is ON"); },
    disable: () => { on = false; console.log("TV is OFF"); }
  };
}

function createRadio() {
  let on = false;
  return {
    isEnabled: () => on,
    enable: () => { on = true; console.log("Radio is ON"); },
    disable: () => { on = false; console.log("Radio is OFF"); }
  };
}

// createRemote → điều khiển thiết bị thông qua “bridge” (device).
function createRemote(device) {
  return {
    togglePower: () => {
      if (device.isEnabled()) {
        device.disable();
      } else {
        device.enable();
      }
    }
  };
}

// Refined abstraction: advanced remote
function createAdvancedRemote(device) {
    const remote = createRemote(device);
    return {
        ...remote,
        mute: () => console.log("Device muted")
    };
}

const tv = createTV();
const radio = createRadio();
const tvRemote = createAdvancedRemote(tv);
tvRemote.togglePower(); // TV is ON
tvRemote.mute();        // Device muted

const radioRemote = createRemote(radio);
radioRemote.togglePower(); // Radio is ON
// => Implementation: createTV() và createRadio()
// => Abstraction: createRemote(device) và createAdvancedRemote(device)
// => Đây là cầu nối (bridge) giúp người dùng điều khiển được thiết bị mà không cần biết đó là TV hay Radio.
// => tách Remote (Abstraction) ra khỏi Device (Implementation).
```

```javascript
function createEmailSender() {
    return {
        send: (message) => console.log(`📧 Gửi email: ${message}`)
    };
}
function createSmsSender() {
    return {
        send: (message) => console.log(`📱 Gửi SMS: ${message}`)
    };
}
function createNotificationSender() {
    return {
        send: (message) => console.log(`📱 Gửi thông báo: ${message}`)
    };
}

function createAlert(sender) {
    return {
        notify: () => sender.send("⚠️ Đây là ALERT khẩn cấp!")
    };
}
function createReminder(sender) {
    return {
        notify: () => sender.send("⏰ Đây là REMINDER nhắc nhở!")
    };
}
function createInfo(sender) {
    return {
        notify: () => sender.send("⚠️ Đây là thông tin cập nhật!")
    };
}

const alertEmail = createAlert(createEmailSender());
alertEmail.notify();
const alertSms = createAlert(createSmsSender());
alertSms.notify();
const reminderEmail = createReminder(createEmailSender());
reminderEmail.notify();


// Sender (Email/SMS) = Implementation
// Alert/Reminder = Abstraction.
// createAlert(sender) hay createReminder(sender) bridge giữa abstraction và implementation.

// Ưu điểm:
//     Thêm loại thông báo mới (ví dụ: Promotion) chỉ cần thêm createPromotion(sender).
//     Thêm kênh gửi mới (ví dụ: PushNotification) chỉ cần thêm createPushSender().

// => đây thực ra là chi nhỏ phần message gồm 2 phần: cách gửi và nội dung
// Cách gửi (Implementation) → Email, SMS, Push...
// Nội dung (Abstraction) → Alert, Reminder, Promotion...
// * Nếu viết bình thường thì sẽ cần có rất nhiều hàm: (theo cấp nó nhân) => Class explosion
// AlertEmail, AlertSMS, AlertPush
// ReminderEmail, ReminderSMS, ReminderPush
// PromotionEmail, PromotionSMS, PromotionPush
```

### Ví dụ: Java

Ví dụ về các loại điều khiển từ xa (Remote) cho các thiết bị (Device) khác nhau.

```java
// --- Implementation (Implementor) ---
// Interface cho các thiết bị
interface Device {
    boolean isEnabled();
    void enable();
    void disable();
    int getVolume();
    void setVolume(int percent);
    int getChannel();
    void setChannel(int channel);
}

// Concrete Implementation 1: TV
class Tv implements Device {
    private boolean on = false;
    private int volume = 30;
    private int channel = 1;

    @Override public boolean isEnabled() { return on; }
    @Override public void enable() { on = true; System.out.println("TV enabled"); }
    @Override public void disable() { on = false; System.out.println("TV disabled"); }
    @Override public int getVolume() { return volume; }
    @Override public void setVolume(int volume) { this.volume = volume; }
    @Override public int getChannel() { return channel; }
    @Override public void setChannel(int channel) { this.channel = channel; }
}

// Concrete Implementation 2: Radio
class Radio implements Device {
    // ... triển khai tương tự TV
}

// --- Abstraction ---
// Lớp trừu tượng cho Remote, chứa tham chiếu đến Device
class RemoteControl {
    protected Device device; // Bridge

    public RemoteControl(Device device) {
        this.device = device;
    }

    public void togglePower() {
        if (device.isEnabled()) {
            device.disable();
        } else {
            device.enable();
        }
    }

    public void volumeUp() {
        device.setVolume(device.getVolume() + 10);
    }

    public void volumeDown() {
        device.setVolume(device.getVolume() - 10);
    }
}

// --- Refined Abstraction ---
// Một loại remote "xịn" hơn
class AdvancedRemoteControl extends RemoteControl {
    public AdvancedRemoteControl(Device device) {
        super(device);
    }

    public void mute() {
        device.setVolume(0);
        System.out.println("Device muted");
    }
}

// --- Sử dụng ---
public class BridgeDemo {
    public static void main(String[] args) {
        Tv tv = new Tv();
        RemoteControl remote = new RemoteControl(tv);
        remote.togglePower(); // TV enabled

        Radio radio = new Radio();
        // Remote "xịn" có thể dùng với radio
        AdvancedRemoteControl advancedRemote = new AdvancedRemoteControl(radio);
        advancedRemote.togglePower();
        advancedRemote.mute(); // Device muted
    }
}
```

### Lợi ích
- **Phát triển độc lập**: Bạn có thể tạo và sửa đổi các lớp abstraction và implementation một cách độc lập.
- **Linh hoạt và dễ mở rộng**: Dễ dàng thêm các abstraction và implementation mới mà không ảnh hưởng đến nhau. Tuân thủ nguyên tắc Open/Closed.
- **Ẩn chi tiết triển khai**: Client chỉ làm việc với abstraction, không cần biết chi tiết về nền tảng (platform-specific) bên dưới.


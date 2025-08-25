# Abstract Factory Pattern

Là một creational pattern (mẫu tạo).

Mục đích: cung cấp một interface chung để tạo ra các họ (family) object liên quan mà không cần chỉ rõ class cụ thể.

Ý tưởng:

- Khi bạn có nhiều loại product (vd: Button, Checkbox)
- Và có nhiều family (vd: MacOSButton, WindowsButton, MacOSCheckbox, WindowsCheckbox)
- Bạn dùng Abstract Factory để đảm bảo các object tạo ra luôn cùng một “họ” → tránh việc mix nhầm MacOSButton với
  WindowsCheckbox.

👉 Hình dung: bạn có xưởng sản xuất UI cho từng hệ điều hành. Bạn không gọi new MacOSButton(), mà gọi
MacOSFactory.createButton().

Mức độ khó (★★★★☆) | Mức độ phổ biến (★★★★☆)

Khá phổ biến trong framework, UI toolkit, khi hỗ trợ nhiều môi trường khác nhau (cross-platform).

### Ví dụ: JavaScript

Trong JavaScript, prototype là một khái niệm cốt lõi. Chúng ta có thể dùng `Object.create()` hoặc một phương thức
`clone` đơn giản.

```javascript
function createWindowsButton() {
    return {render: () => console.log("Render Windows Button 🪟")};
}

function createMacButton() {
    return {render: () => console.log("Render MacOS Button 🍎")};
}

function createWindowsCheckbox() {
    return {render: () => console.log("Render Windows Checkbox 🪟☑️")};
}

function createMacCheckbox() {
    return {render: () => console.log("Render MacOS Checkbox 🍎☑️")};
}

// Abstract Factory
function createUIFactory(os) {
    if (os === "windows") {
        return {
            createButton: createWindowsButton,
            createCheckbox: createWindowsCheckbox
        };
    }
    if (os === "mac") {
        return {
            createButton: createMacButton,
            createCheckbox: createMacCheckbox
        };
    }
    throw new Error("Unknown OS");
}

// --- Demo ---
const factory = createUIFactory("windows");
const btn = factory.createButton();
const chk = factory.createCheckbox();

btn.render(); // Render Windows Button 🪟
chk.render(); // Render Windows Checkbox 🪟☑️
// 👉 Ở đây factory đóng vai trò Abstract Factory, còn WindowsFactory/MacFactory là Concrete Factory.
```

```java
// Abstract Products
interface Button {
    void render();
}
interface Checkbox {
    void render();
}

// Concrete Products
class WindowsButton implements Button {
    public void render() { System.out.println("Render Windows Button 🪟"); }
}
class MacButton implements Button {
    public void render() { System.out.println("Render MacOS Button 🍎"); }
}

class WindowsCheckbox implements Checkbox {
    public void render() { System.out.println("Render Windows Checkbox 🪟☑️"); }
}
class MacCheckbox implements Checkbox {
    public void render() { System.out.println("Render MacOS Checkbox 🍎☑️"); }
}

// Abstract Factory
interface UIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete Factories
class WindowsFactory implements UIFactory {
    public Button createButton() { return new WindowsButton(); }
    public Checkbox createCheckbox() { return new WindowsCheckbox(); }
}

class MacFactory implements UIFactory {
    public Button createButton() { return new MacButton(); }
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}

// Demo
public class AbstractFactoryDemo {
    public static void main(String[] args) {
        UIFactory factory = new WindowsFactory();
        Button btn = factory.createButton();
        Checkbox chk = factory.createCheckbox();

        btn.render();
        chk.render();
    }
}
```

So sánh với Abstract Factory, Factory Method, Simple Factory

| Pattern          | Mục đích chính                                                   | Số lượng Product Type | Số lượng Family | Cách tạo object                         |
|------------------|------------------------------------------------------------------|-----------------------|-----------------|-----------------------------------------|
| Simple Factory   | Tạo object mà không cần lộ class/ hàm cụ thể                     | 1                     | 1               | Một hàm tĩnh (static method)            |
| Factory Method   | Cho phép sub-class/sub-function quyết định class nào sẽ được tạo | 1                     | Nhiều           | Subclass override phương thức tạo       |
| Abstract Factory | Tạo họ (family) object liên quan                                 | Nhiều                 | Nhiều           | Interface chung, nhiều Concrete Factory |

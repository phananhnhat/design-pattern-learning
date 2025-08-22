# Singleton Pattern

Là một creational pattern (mẫu tạo).

Ý tưởng:
- Đảm bảo rằng một class chỉ có duy nhất một instance trong suốt vòng đời của ứng dụng, và cung cấp một điểm truy cập toàn cục (global access) tới instance đó.

Dùng khi bạn chỉ cần một đối tượng duy nhất: ví dụ như cấu hình hệ thống, logger, connection pool, cache...

Mức độ khó (★☆☆☆☆) | Mức độ phổ biến (★★★★☆)

```javascript
// Trong JS không có class-based strict OOP như Java, nhưng vẫn có thể tạo Singleton qua module hoặc closure.
const Logger = (function () {
    let instance; // giữ instance duy nhất

    function createInstance() {
        return {
            log: (msg) => console.log("[LOG]", msg)
        };
    }

    return {
        getInstance: function () {
            if (!instance) {
                instance = createInstance();
            }
            return instance;
        }
    };
})();

const logger1 = Logger.getInstance();
const logger2 = Logger.getInstance();
logger1.log("Hello World!"); // [LOG] Hello World!
console.log(logger1 === logger2); // true
```

```javascript
    // Với ESModule: logger.js
    const createInstance = () => {
        return {
            log: (msg) => console.log("[LOG]", msg)
        };
    }
    
    export const getInstance = () => {
        if (!instance) {
            instance = createInstance();
        }
        return instance;
    }
    
    // Hoặc: logger1.js

    export const logger1 = {
        log: (msg) => console.log("[LOG]", msg)
    };
    // => Đây cũng đc coi là Singleton vì chỉ có một instance duy nhất.
```

```java
public class Logger {
    // 1. Tạo instance duy nhất, private static
    private static Logger instance;

    // 2. Constructor private để chặn tạo từ bên ngoài
    private Logger() {}

    // 3. Cung cấp phương thức public để lấy instance
    public static Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }

    public void log(String message) {
        System.out.println("[LOG] " + message);
    }
}

// ---- Sử dụng ----
public class Main {
    public static void main(String[] args) {
        Logger logger1 = Logger.getInstance();
        Logger logger2 = Logger.getInstance();

        logger1.log("Hello World!");

        // Kiểm tra 2 biến có trỏ cùng một object không
        System.out.println(logger1 == logger2); // true
    }
}
```
Logger.getInstance() luôn trả về cùng một instance.

logger1 == logger2 trả về true.
# Flyweight Pattern

Là một structural pattern (mẫu cấu trúc).

Mục đích: giảm bộ nhớ sử dụng bằng cách chia sẻ các object giống nhau thay vì tạo mới mỗi lần.

Ý tưởng:
- Một object được chia thành intrinsic state (trạng thái nội tại, cố định, có thể dùng chung) và extrinsic state (trạng thái bên ngoài, thay đổi theo ngữ cảnh).
- Flyweight Factory quản lý và tái sử dụng các object có intrinsic state giống nhau.

Dùng khi có rất nhiều object giống nhau để tiết kiệm bộ nhớ.

👉 Hình dung: game có hàng triệu “cây” hoặc “viên đạn” → thay vì new object cho từng cái, ta tái sử dụng 1 số ít model chung (intrinsic), chỉ thay đổi vị trí, màu, size (extrinsic).

Mức độ khó (★★★★☆) | Mức độ phổ biến (★★★☆☆)

Gặp nhiều trong game dev, UI, caching, nhưng không phổ biến trong ứng dụng business thông thường.

```javascript
// Ví dụ: quản lý icon user trong UI (nhiều user nhưng cùng loại icon).
const iconFactory = (() => {
    const icons = {};
    return {
        getIcon: (type) => {
            if (!icons[type]) {
                console.log("Creating new icon:", type);
                // intrinsic state: hình ảnh
                icons[type] = { type, data: `<svg>${type}</svg>` };
            }
            return icons[type];
        }
    };
})();

const user1Icon = iconFactory.getIcon("user");
const user2Icon = iconFactory.getIcon("user");
const adminIcon = iconFactory.getIcon("admin");

console.log(user1Icon === user2Icon); // true (dùng chung)
console.log(user1Icon === adminIcon); // false
// 👉 Ở đây "user" icon chỉ tạo 1 lần, tái sử dụng cho nhiều người.
```

```java
import java.util.*;

// Flyweight
class Icon {
    private String type;   // intrinsic state
    private String data;

    public Icon(String type) {
        this.type = type;
        this.data = "<svg>" + type + "</svg>";
    }

    public void draw(int x, int y) { // extrinsic state: vị trí
        System.out.println("Draw " + type + " at (" + x + ", " + y + ")");
    }
}

// Flyweight Factory
class IconFactory {
    private Map<String, Icon> icons = new HashMap<>();

    public Icon getIcon(String type) {
        if (!icons.containsKey(type)) {
            System.out.println("Creating new icon: " + type);
            icons.put(type, new Icon(type));
        }
        return icons.get(type);
    }
}

// Demo
public class FlyweightDemo {
    public static void main(String[] args) {
        IconFactory factory = new IconFactory();

        Icon user1Icon = factory.getIcon("user");
        Icon user2Icon = factory.getIcon("user");
        Icon adminIcon = factory.getIcon("admin");

        user1Icon.draw(10, 20);
        user2Icon.draw(50, 80);
        adminIcon.draw(100, 200);

        System.out.println(user1Icon == user2Icon); // true
    }
}
```

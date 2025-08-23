# Prototype Pattern

Là một creational pattern (mẫu tạo).

Ý tưởng chính của mẫu này là tạo ra các đối tượng mới bằng cách sao chép (cloning) một đối tượng đã tồn tại, được gọi là "prototype". Thay vì tạo một đối tượng từ đầu (tốn kém chi phí khởi tạo), bạn chỉ cần nhân bản một đối tượng mẫu.

👉 **Tóm lại**: Clone đối tượng đã có để tạo đối tượng mới.

Mức độ khó (★★☆☆☆) | Mức độ phổ biến (★★★☆☆)

### Khi nào dùng?
- Khi việc tạo một đối tượng rất tốn kém (ví dụ: cần gọi API, truy vấn database) và việc clone sẽ hiệu quả hơn.
- Khi bạn muốn có nhiều đối tượng giống hệt nhau hoặc chỉ khác nhau một vài thuộc tính.
- Hệ thống muốn ẩn chi tiết về việc khởi tạo đối tượng.
- Prototype Pattern định nghĩa một giao diện (interface) có phương thức clone().

### Ví dụ: JavaScript

Trong JavaScript, prototype là một khái niệm cốt lõi. Chúng ta có thể dùng `Object.create()` hoặc một phương thức `clone` đơn giản.

```javascript
// Prototype object
const carPrototype = {
  init(model) {
    this.model = model;
  },
  getModel() {
    return `Car model is ${this.model}`;
  },
  clone() {
    // Object.create tạo một object mới với prototype được chỉ định
    const clone = Object.create(this); 
    // Hoặc bạn có thể dùng spread syntax cho shallow copy đơn giản
    // const clone = {...this};
    return clone;
  }
};

// Sử dụng prototype để tạo các đối tượng mới
const car1 = carPrototype.clone();
car1.init("Toyota Camry");
console.log(car1.getModel()); // Car model is Toyota Camry

const car2 = carPrototype.clone();
car2.init("Honda Civic");
console.log(car2.getModel()); // Car model is Honda Civic
```

```javascript
// Hàm khởi tạo tài liệu (giả lập tốn kém)
function createDocument(content) {
    console.log("Đang tải dữ liệu lớn...");
    return {
        content: content.split("").reverse().join(""), // xử lý giả lập nặng
        clone: () => { return {...this} }
    };
}

// Hàm clone nhanh (không chạy lại xử lý nặng)
function cloneDocument(doc) {
    return { ...doc }; // shallow copy
}

// Khởi tạo ban đầu (tốn kém)
const doc1 = createDocument("Nội dung gốc");

// Clone (rẻ hơn nhiều, không chạy lại xử lý nặng)
const doc2 = cloneDocument(doc1);
const doc3 = doc1.clone();
```

### Ví dụ: Java

Trong Java, chúng ta có thể sử dụng interface `Cloneable` và override phương thức `clone()`.

```java
// Interface cho các đối tượng có thể clone
interface Prototype extends Cloneable {
    Prototype clone() throws CloneNotSupportedException;
}

// Lớp cụ thể
class Sheep implements Prototype {
    private String name;
    private String category;

    public Sheep(String name, String category) {
        this.name = name;
        this.category = category;
        // Giả sử có một quá trình khởi tạo tốn kém ở đây
        System.out.println("Sheep instance created.");
    }

    // Getters and Setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getCategory() { return category; }
    public void setCategory(String category) { this.category = category; }

    @Override
    public Prototype clone() throws CloneNotSupportedException {
        System.out.println("Cloning Sheep...");
        return (Sheep) super.clone();
    }

    @Override
    public String toString() {
        return "Sheep [name=" + name + ", category=" + category + "]";
    }
}

// --- Sử dụng ---
public class PrototypeDemo {
    public static void main(String[] args) throws CloneNotSupportedException {
        // Tạo đối tượng gốc (tốn kém)
        Sheep original = new Sheep("Jolly", "Mountain Sheep");
        System.out.println("Original: " + original);

        // Clone từ đối tượng gốc (nhanh hơn)
        Sheep cloned = (Sheep) original.clone();
        cloned.setName("Dolly");
        System.out.println("Cloned: " + cloned);
        
        // Original không bị thay đổi
        System.out.println("Original after clone: " + original);
    }
}
```

### Shallow vs. Deep Copy

- **Shallow Copy (Sao chép nông)**: Chỉ sao chép các trường giá trị (primitive types). Các trường tham chiếu (đối tượng, mảng) sẽ chỉ được sao chép địa chỉ. Nếu sửa đổi đối tượng tham chiếu trong bản sao, bản gốc cũng bị thay đổi. `Object.clone()` mặc định là shallow copy.
- **Deep Copy (Sao chép sâu)**: Sao chép tất cả, kể cả các đối tượng lồng nhau. Bản sao và bản gốc hoàn toàn độc lập. Để deep copy, bạn cần tự triển khai logic sao chép cho các đối tượng bên trong.

Prototype pattern yêu cầu bạn phải chú ý đến việc này để tránh các lỗi không mong muốn.


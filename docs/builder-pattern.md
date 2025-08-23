# Builder Pattern

Là một creational pattern (mẫu tạo).

Ý tưởng chính:
- Tách biệt quá trình xây dựng một đối tượng phức tạp khỏi cách mà nó được biểu diễn. Điều này cho phép bạn sử dụng cùng một quy trình xây dựng để tạo ra các phiên bản khác nhau của đối tượng.
- Người dùng không cần biết chi tiết object được tạo ra như thế nào, chỉ cần gọi builder để ghép từng phần.
- Dùng một đối tượng Builder để từng bước xây dựng một object phức tạp.
- 
👉 **Tóm lại**: Xây dựng một đối tượng phức tạp từng bước một (step-by-step).

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★★☆)

### Khi nào dùng?
- Khi việc khởi tạo một đối tượng cần nhiều tham số, một vài trong số đó là tùy chọn (optional). Thay vì tạo ra hàng loạt constructor(hàm khởi tạo), Builder Pattern là lựa chọn sạch sẽ hơn.
- Khi bạn muốn tạo một đối tượng phức tạp và quá trình tạo cần diễn ra theo từng giai đoạn.
- Khi bạn muốn đảm bảo đối tượng được tạo ra là immutable (bất biến) sau khi đã xây dựng xong.

### Ví dụ: JavaScript

Trong JavaScript, chúng ta có thể implement Builder bằng cách sử dụng một class hoặc object có các phương thức nối chuỗi (chainable methods).

```javascript
class User {
    constructor(name, age, phone, address) {
        this.name = name;
        this.age = age;
        this.phone = phone;
        this.address = address;
    }

    toString() {
        return `User: ${this.name}, Age: ${this.age}, Phone: ${this.phone}, Address: ${this.address}`;
    }
}

class UserBuilder {
    constructor(name) {
        this.name = name;
    }

    setAge(age) {
        this.age = age;
        return this; // Trả về this để nối chuỗi
    }

    setPhone(phone) {
        this.phone = phone;
        return this;
    }

    setAddress(address) {
        this.address = address;
        return this;
    }

    build() {
        return new User(this.name, this.age, this.phone, this.address);
    }
}

// --- Sử dụng ---
const user = new UserBuilder("John Doe")
    .setAge(30)
    .setPhone("123-456-7890")
    .setAddress("123 Main St")
    .build();

const user2 = new UserBuilder("Jane Doe")
    .setAge(25)
    .build(); // Tạo user chỉ với age
```




```javascript
// Bạn cũng có thể triển khai Builder Pattern bằng cách sử dụng các đối tượng và hàm thuần túy, giúp code linh hoạt hơn và tránh dùng `this` trong ngữ cảnh class.
const userBuilder = {
  withName(name) {
    return { ...this, name };     // Dùng spread syntax để tạo object mới, đảm bảo tính bất biến
  },
  withAge(age) {
    return { ...this, age };
  },
  withPhone(phone) {
    return { ...this, phone };
  },
  withAddress(address) {
    return { ...this, address };
  },
  build() {
    return {
      name: this.name,
      age: this.age,
      phone: this.phone,
      address: this.address,
      toString() {
        return `User: ${this.name}, Age: ${this.age}, Phone: ${this.phone}, Address: ${this.address}`;
      }
    };
  }
};
const user3 = Object.assign({}, userBuilder)
    .withName("Peter Pan")
    .withAge(100)
    .build();
const user4 = Object.assign({}, userBuilder)
    .withName("Wendy Darling")
    .withAge(15)
    .withAddress("London")
    .build();
```

### Ví dụ: Java - Trong Java, Builder Pattern cực kỳ phổ biến, đặc biệt với các thư viện như Lombok (`@Builder`).

```java
// Product
class House {
    private final String foundation;
    private final String structure;
    private final String roof;
    private final String interior;

    // Constructor là private, chỉ Builder mới được gọi
    private House(HouseBuilder builder) {
        this.foundation = builder.foundation;
        this.structure = builder.structure;
        this.roof = builder.roof;
        this.interior = builder.interior;
    }

    @Override
    public String toString() {
        return "House [foundation=" + foundation + ", structure=" + structure + ", roof=" + roof + ", interior=" + interior + "]";
    }

    // Static nested Builder class
    public static class HouseBuilder {
        private String foundation;
        private String structure;
        private String roof;
        private String interior;

        // Các bước xây dựng
        public HouseBuilder buildFoundation(String foundation) {
            this.foundation = foundation;
            return this;
        }

        public HouseBuilder buildStructure(String structure) {
            this.structure = structure;
            return this;
        }

        public HouseBuilder buildRoof(String roof) {
            this.roof = roof;
            return this;
        }

        public HouseBuilder buildInterior(String interior) {
            this.interior = interior;
            return this;
        }

        // Phương thức build trả về đối tượng House hoàn chỉnh
        public House build() {
            return new House(this);
        }
    }
}

// --- Sử dụng ---
public class BuilderDemo {
    public static void main(String[] args) {
        // Tạo một ngôi nhà đơn giản
        House simpleHouse = new House.HouseBuilder()
            .buildFoundation("Concrete")
            .buildStructure("Wood")
            .buildRoof("Shingles")
            .build();
        System.out.println("Simple House: " + simpleHouse);

        // Tạo một ngôi nhà đầy đủ nội thất
        House fancyHouse = new House.HouseBuilder()
            .buildFoundation("Concrete, Steel, and Glass")
            .buildStructure("Steel Frame")
            .buildRoof("Solar Panels")
            .buildInterior("Luxury Modern")
            .build();
        System.out.println("Fancy House: " + fancyHouse);
    }
}
```

### Lợi ích
- **Dễ đọc hơn**: Code khởi tạo đối tượng trở nên rõ ràng, dễ hiểu hơn so với một constructor có hàng chục tham số.
- **Linh hoạt**: Cho phép tạo các đối tượng khác nhau với cùng một quy trình xây dựng.
- **Giảm số lượng constructor**: Tránh được "telescoping constructor anti-pattern" (khi bạn phải tạo nhiều constructor với các tham số khác nhau).
- **Tạo đối tượng bất biến (Immutable)**: Builder có thể tạo ra các đối tượng mà trạng thái của chúng không thể thay đổi sau khi tạo.

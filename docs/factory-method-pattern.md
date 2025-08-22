# Factory Method Pattern

Là một creational pattern (mẫu tạo).

Khác với Simple Factory (một hàm duy nhất tạo ra mọi thứ), Factory Method cho phép mỗi subclass (hoặc từng “loại”) tự định nghĩa cách tạo ra đối tượng của nó.

Ý tưởng:
- Có một interface/abstract để định nghĩa method createObject(), còn mỗi subclass sẽ triển khai chi tiết tạo ra sản phẩm cụ thể.


Mức độ khó (★★☆☆☆) | Mức độ phổ biến (★★★★★)

```javascript
// Product
function circle() {
    return {
        draw: () => console.log("Drawing a Circle")
    };
}

function square() {
    return {
        draw: () => console.log("Drawing a Square")
    };
}

const CircleFactory = {
    create: () => circle()
};

const SquareFactory = {
    create: () => square()
};

// ---- Sử dụng ----
const c = CircleFactory.create();
c.draw(); // Drawing a Circle

const s = SquareFactory.create();
s.draw(); // Drawing a Square
```

```java
// Product interface
interface Shape {
    void draw();
}

class Circle implements Shape {
    public void draw() {
        System.out.println("Drawing a Circle");
    }
}

class Square implements Shape {
    public void draw() {
        System.out.println("Drawing a Square");
    }
}

abstract class ShapeFactory {
    abstract Shape createShape();
}

class CircleFactory extends ShapeFactory {
    public Shape createShape() {
        return new Circle();
    }
}

class SquareFactory extends ShapeFactory {
    public Shape createShape() {
        return new Square();
    }
}

// ---- Sử dụng ----
public class Main {
    public static void main(String[] args) {
        ShapeFactory factory = new CircleFactory();
        Shape circle = factory.createShape();
        circle.draw(); // Drawing a Circle

        factory = new SquareFactory();
        Shape square = factory.createShape();
        square.draw(); // Drawing a Square
    }
}
```
👉 Ở đây ShapeFactory là factory method: mỗi subclass của nó (
CircleFactory, SquareFactory) quyết định sản xuất ra loại Shape nào.

🔑 Khác biệt chính so với Simple Factory

Simple Factory: chỉ có một function (hoặc class) duy nhất để tạo ra nhiều loại sản phẩm → nếu thêm sản phẩm mới phải sửa lại factory.

Factory Method: tách riêng ra từng factory (mỗi factory biết cách tạo sản phẩm của nó), dễ mở rộng mà không cần sửa code cũ → tuân thủ Open/Closed Principle (O trong SOLID).
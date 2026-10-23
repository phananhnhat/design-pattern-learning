Nếu class B kế thừa class A, ta phải có thể thay thế A bằng B mà không làm sai tính đúng đắn của chương trình.

```java
class Bird {
  fly(): void {
    console.log("Bird is flying");
  }
}

class Penguin extends Bird {
  fly(): void {
    throw new Error("Penguins cannot fly!");
  }
}

function makeBirdFly(bird: Bird) {
  bird.fly();
}

const bird = new Bird();
makeBirdFly(bird); // OK

const penguin = new Penguin();
makeBirdFly(penguin); // Lỗi runtime!
```

Tại sao vi phạm LSP?

Hàm makeBirdFly() kỳ vọng mọi đối tượng Bird đều có thể bay. Nhưng Penguin không đáp ứng được kỳ vọng đó.

Việc kế thừa khiến Penguin không thể thay thế Bird trong ngữ cảnh này.

Lưu ý: Vấn đề không nằm ở việc chim cánh cụt không biết bay, mà nằm ở việc thiết kế class cha đã giả định rằng mọi loài chim đều bay được.

1. Khác kết quả chưa chắc vi phạm LSP


```java
class A {
  x(): number {
    return 10;
  }
}

class B extends A {
  x(): number {
    return 20;
  }
}
```

B.x() trả về 20, khác với A.x() trả về 10.

Điều này chưa đủ để kết luận vi phạm LSP. Ta cần biết hợp đồng của A.x() là gì:
- Nếu hợp đồng chỉ yêu cầu trả về number, B trả về 20 vẫn có thể đúng.
- Nếu hợp đồng yêu cầu luôn trả về chính xác 10, B trả về 20 là vi phạm.

2. Chương trình không lỗi vẫn có thể vi phạm LSP

```java
function test(obj: A) {
  if (obj.x() === 10) {
    console.log("OK");
  } else {
    console.log("FAIL");
  }
}

test(new A()); // OK
test(new B()); // FAIL
```

Nếu hợp đồng của A.x() yêu cầu trả về 10, thì B không thể thay thế A đúng như kỳ vọng của chương trình.

Việc chương trình không throw exception không có nghĩa là tuân thủ LSP.

4. Ví dụ thêm về việc vi phạm LSP để rõ hơn về kế thừa (lấy ở tài liệu)

```java
class Rectangle {
    protected double height;
    protected double width;

    public void setHeight(double h) {
        height = h;
    }

    public void setWidth(double w) {
        width = w;
    }
}

class Square extends Rectangle {
    @Override
    public void setWidth(double w) {
        width = w;
        height = w;
    }

    @Override
    public void setHeight(double h) {
        height = h;
        width = h;
    }
}
```

Square kế thừa Rectangle nhưng thay đổi hành vi của setWidth() và setHeight(), khiến code sử dụng Rectangle có thể không hoạt động như kỳ vọng.
Có 2 cách fix phổ biến, tùy vào thiết kế bạn muốn.

Cách 1: Không cho Square kế thừa Rectangle. Tách chúng thành hai class độc lập, cùng triển khai một interface Shape.

```java
interface Shape {
    double getArea();
}

class Rectangle implements Shape {
    protected double height;
    protected double width;

    public void setHeight(double h) {
        height = h;
    }

    public void setWidth(double w) {
        width = w;
    }

    @Override
    public double getArea() {
        return height * width;
    }
}

class Square implements Shape {
    private double side;

    public void setSide(double side) {
        this.side = side;
    }

    @Override
    public double getArea() {
        return side * side;
    }
}
```

```java
Shape shape = new Rectangle();
Shape square = new Square();
```
Cả hai đều cung cấp getArea(), nhưng không giả vờ rằng chúng có cùng hành vi chỉnh sửa kích thước.

Cách 2: Dùng composition thay cho inheritance

Nếu muốn tái sử dụng logic tính diện tích hình chữ nhật, bạn có thể tách phần tính toán ra thành một class riêng và để Square sử dụng nó.

```java

class AreaCalculator {
    public double rectangleArea(double width, double height) {
        return width * height;
    }

    public double squareArea(double side) {
        return side * side;
    }
}

class Rectangle {
    private double width;
    private double height;
    private final AreaCalculator calculator;

    public Rectangle(AreaCalculator calculator) {
        this.calculator = calculator;
    }

    public void setWidth(double width) {
        this.width = width;
    }

    public void setHeight(double height) {
        this.height = height;
    }

    public double getArea() {
        return calculator.rectangleArea(width, height);
    }
}

class Square {
    private double side;
    private final AreaCalculator calculator;

    public Square(AreaCalculator calculator) {
        this.calculator = calculator;
    }

    public void setSide(double side) {
        this.side = side;
    }

    public double getArea() {
        return calculator.squareArea(side);
        //  hoặc
        return calculator.rectangleArea(side, side);
    }
}
```
Composition nằm ở đâu?
Hai class này đều chứa một đối tượng AreaCalculator:
Rectangle quản lý width và height độc lập.

Square chỉ quản lý side.
Cả hai tái sử dụng logic tính toán thông qua AreaCalculator.
Square không cần kế thừa Rectangle, nên không gặp vấn đề thay đổi hành vi setter.
Nhưng có một điểm quan trọng
Ví dụ trên minh họa composition, nhưng AreaCalculator khá đơn giản và không nhất thiết phải tách thành class riêng trong dự án thực tế.
Ta có thể tính diện tích trực tiếp trong từng class mà vẫn tuân thủ LSP, vì hai class không còn quan hệ kế thừa.

---
Tuy nhiên, với ví dụ này, cách 1 thường đơn giản và dễ hiểu hơn.
Tại sao không giữ Square extends Rectangle rồi sửa các method?
Vì bản chất hai class có những hợp đồng khác nhau:
- Rectangle: đổi width không làm thay đổi height.
- Square: đổi một cạnh bắt buộc phải làm thay đổi cạnh còn lại.

Nếu giữ quan hệ kế thừa, Square không thể đáp ứng hợp đồng của Rectangle trong những ngữ cảnh yêu cầu hai chiều độc lập.
Một lưu ý: nếu Rectangle chỉ cam kết có hai kích thước và không cam kết chúng độc lập, thì phải xem xét hợp đồng cụ thể trước khi kết luận vi phạm LSP.
Nhưng trong thiết kế thông thường, ví dụ này được dùng để minh họa sự vi phạm đó.
Chốt lại: Đừng chọn kế thừa chỉ vì trong toán học hình vuông là một hình chữ nhật đặc biệt.
Trong OOP, quan hệ kế thừa phải bảo đảm khả năng thay thế về mặt hành vi, không chỉ đúng về mặt phân loại.

5. Làm sao biết có vi phạm LSP?

Cần xác định hợp đồng của class cha thông qua:
- Đặc tả nghiệp vụ: Hành vi method phải đáp ứng.
- Tài liệu: Những gì method cam kết.
- Điều kiện đầu vào và đầu ra: Method được phép nhận gì và phải đảm bảo điều gì sau khi thực hiện.
- Test: Các hành vi mà class con cần giữ nguyên khi thay thế class cha.

Kết luận quan trọng nhất: LSP không yêu cầu class cha và class con phải có cách triển khai hoặc kết quả giống hệt nhau.
Nó yêu cầu class con giữ đúng hợp đồng hành vi của class cha để có thể thay thế class cha mà không phá vỡ tính đúng đắn của chương trình.
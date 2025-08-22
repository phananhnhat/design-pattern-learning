# Simple Factory Pattern

Là mẫu thiết kế khởi tạo (creational design pattern) nhưng không phải là Design Pattern chính thức trong GoF, nhưng được dùng rất phổ biến.

Ý tưởng:
- Tạo ra một Factory (nhà máy) có nhiệm vụ sinh ra các đối tượng cụ thể dựa trên tham số đầu vào.
- Thay vì new trực tiếp object, ta gọi Factory để quyết định tạo cái gì.

👉 Mục đích:
- Giấu logic khởi tạo phức tạp.
- Tách biệt code khởi tạo với code sử dụng.

Mức độ khó (★★☆☆☆) | Mức độ phổ biến (★★★☆☆)

```javascript
class Car {
  drive() {
    console.log("Driving a car 🚗");
  }
}

class Bike {
  drive() {
    console.log("Riding a bike 🚲");
  }
}

// Simple Factory
function VehicleFactory(type) {
  if (type === "car") {
    return new Car();
  } else if (type === "bike") {
    return new Bike();
  } else {
    throw new Error("Unknown vehicle type");
  }
}

// Client code
const vehicle1 = VehicleFactory("car");
vehicle1.drive(); // Driving a car 🚗

const vehicle2 = VehicleFactory("bike");
vehicle2.drive(); // Riding a bike 🚲

// Không new Car() trực tiếp trong client.
// Client chỉ cần gọi VehicleFactory.createVehicle("car").
```

```javascript

function circle(radius) {
  return { type: "circle", radius, area: () => Math.PI * radius * radius };
}

function square(side) {
    return { type: "square", side, area: () => side * side };
}

function rectangle(width, height) {
    return { type: "rectangle", width, height, area: () => width * height };
}

function shapeFactory(type, ...params) {
    switch (type) {
        case "circle":
            return circle(...params);
        case "square":
            return square(...params);
        case "rectangle":
            return rectangle(...params);
        default:
            throw new Error("Unknown shape type");
    }
}

// ---- Sử dụng ----
const c = shapeFactory("circle", 5);
console.log(c.type, c.area()); // circle 78.53981633974483

const s = shapeFactory("square", 4);
console.log(s.type, s.area()); // square 16

const r = shapeFactory("rectangle", 3, 6);
console.log(r.type, r.area()); // rectangle 18
```

```java
// Các "sản phẩm"
interface Vehicle {
    void drive();
}

class Car implements Vehicle {
    public void drive() {
        System.out.println("Driving a car 🚗");
    }
}

class Bike implements Vehicle {
    public void drive() {
        System.out.println("Riding a bike 🚲");
    }
}

// Simple Factory
class VehicleFactory {
    public static Vehicle createVehicle(String type) {
        if ("car".equalsIgnoreCase(type)) {
            return new Car();
        } else if ("bike".equalsIgnoreCase(type)) {
            return new Bike();
        } else {
            throw new IllegalArgumentException("Unknown vehicle type");
        }
    }
}

// Client code
public class Main {
    public static void main(String[] args) {
        Vehicle v1 = VehicleFactory.createVehicle("car");
        v1.drive(); // Driving a car 🚗

        Vehicle v2 = VehicleFactory.createVehicle("bike");
        v2.drive(); // Riding a bike 🚲
    }
}
```
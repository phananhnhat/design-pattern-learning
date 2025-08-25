# Template Method Pattern

Là mẫu thiết kế hành vi (behavioral design pattern)
Mục đích: định nghĩa bộ khung (skeleton) của một thuật toán trong một method, và cho phép subclass (hoặc callback) thay thế một số bước cụ thể mà không thay đổi cấu trúc chung.

Ý tưởng:
- Một thuật toán được chia thành nhiều bước.
- Template method chứa thứ tự các bước.
- Subclass (hoặc function) sẽ override / inject các bước chi tiết.

👉 Hình dung: giống như công thức nấu ăn:
- Template: “Chuẩn bị nguyên liệu → Nấu → Ăn”.
- Chi tiết: món súp hay món chiên thì mỗi bước “Nấu” khác nhau.

Mức độ khó (★★★☆☆) | Mức độ phổ biến (★★★★☆)

Dùng nhiều trong framework, khi cần định nghĩa luồng chuẩn (vd: hook của React, lifecycle của Servlet, JUnit test).

```javascript
function dataProcessor(template) {
    return {
        process: () => {
            console.log("Step 1: Load data");
            template.parse();
            template.save();
        }
    };
}

const csvProcessor = dataProcessor({
    parse: () => console.log("Parsing CSV file..."),
    save: () => console.log("Saving CSV to DB...")
});

const jsonProcessor = dataProcessor({
    parse: () => console.log("Parsing JSON file..."),
    save: () => console.log("Saving JSON to DB...")
});

csvProcessor.process();
jsonProcessor.process();
```

```javascript
function makeDrink(template) {
  console.log("1. Đun nước...");
  template.brew();       // bước đặc thù
  console.log("3. Rót ra cốc...");
  if (template.addCondiments) {
    template.addCondiments(); // bước tùy chọn
  }
  console.log("✅ Hoàn thành!\n");
}

const makeTea = () => makeDrink({
    brew: () => console.log("2. Ngâm trà 🍵"),
    addCondiments: () => console.log("4. Cho thêm chanh 🍋")
});

const makeCoffee = () => makeDrink({
    brew: () => console.log("2. Pha cà phê ☕"),
    addCondiments: () => console.log("4. Cho thêm sữa 🥛")
});

makeTea();
makeCoffee();
```


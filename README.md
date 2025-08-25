# design-pattern-learning

GoF Design Patterns (23)

* **Creational Patterns (5) – Nhóm khởi tạo**
    * [Factory Method](./docs/factory-method-pattern.md)
    * [Abstract Factory](./docs/abstract-factory-pattern.md)
    * [Builder](./docs/builder-pattern.md)
    * [Prototype](./docs/prototype-pattern.md)
    * [Singleton](./docs/singleton-pattern.md)
* **Structural Patterns (7) – Nhóm cấu trúc**
    * [Adapter](docs/adapter-pattern.md)
    * [Bridge](./docs/bridge-pattern.md)
    * [Composite](./docs/composite-pattern.md)
    * [Decorator](./docs/decorator-pattern.md)
    * [Facade](docs/facade-pattern.md)
    * [Flyweight](docs/flyweight-pattern.md)
    * [Proxy](./docs/proxy-pattern.md)
* **Behavioral Patterns (11) – Nhóm hành vi**
    * [Chain of Responsibility](./docs/chain-of-responsibility.md)
    * [Command](docs/command-pattern.md)
    * [Interpreter](docs/interpreter-pattern.md)
    * [Iterator](docs/iterator-pattern.md)
    * [Mediator](docs/mediator-pattern.md)
    * [Memento](docs/memento-pattern.md)
    * [Observer](./docs/observer-pattern.md)
    * [State](./docs/state-pattern.md)
    * [Strategy](./docs/strategy-pattern.md)
    * [Template Method](./docs/template-method-pattern.md)
    * [Visitor](./docs/template-method-pattern.md)

1. **Factory Method**
    - Cung cấp một interface để tạo object, nhưng để subclass quyết định sẽ tạo object nào.
    - → Giảm phụ thuộc, dễ mở rộng.

2. **Abstract Factory**
    - Cung cấp một interface để tạo ra *một nhóm* object liên quan hoặc phụ thuộc, mà không cần chỉ định class cụ thể.
    - → Dễ dàng tạo nhiều sản phẩm “đồng bộ” với nhau.

3. **Builder**
    - Tách rời quá trình *xây dựng* một object phức tạp khỏi *cách biểu diễn* của nó.
    - → Cho phép tạo ra các biểu diễn khác nhau của cùng một đối tượng.

4. **Prototype**
    - Tạo object mới bằng cách *sao chép (clone)* một object mẫu có sẵn.
    - → Hữu ích khi việc tạo object từ đầu tốn kém.

5. **Singleton**
    - Đảm bảo một class chỉ có duy nhất một instance, và cung cấp một điểm truy cập toàn cục đến nó.
    - → Kiểm soát tài nguyên dùng chung.

6. **Adapter**
    - Chuyển đổi interface của một class thành một interface khác mà client mong muốn.
    - → Cho phép các thành phần không tương thích làm việc với nhau.

7. **Bridge**
    - Tách rời *abstraction* (trừu tượng) khỏi *implementation* (cài đặt) để cả hai có thể thay đổi độc lập.
    - → Tránh tình trạng class explosion khi có nhiều chiều mở rộng.

8. **Composite**
    - Tổ chức object thành cấu trúc cây để biểu diễn quan hệ *phần – toàn thể*.
    - → Cho phép xử lý object riêng lẻ và nhóm object theo cùng một cách.

9. **Decorator**
    - Gắn thêm hành vi/trách nhiệm mới cho object một cách linh hoạt mà không cần thay đổi class gốc.
    - → “Bọc” object thay vì kế thừa.

10. **Facade**
    - Cung cấp một interface đơn giản cho một hệ thống phức tạp.
    - → Giảm phụ thuộc giữa client và hệ thống con.

11. **Flyweight**
    - Tái sử dụng object chung thay vì tạo mới, nhằm tiết kiệm bộ nhớ.
    - → Hữu ích khi có nhiều object giống nhau, chỉ khác nhau một vài trạng thái nhỏ.

12. **Proxy**
    - Cung cấp một lớp thay thế/đại diện cho object khác để kiểm soát truy cập.
    - → Có thể dùng cho lazy load, security, logging, caching…

13. **Chain of Responsibility**
    - Truyền request qua một chuỗi các handler cho đến khi có handler xử lý nó.
    - → Giảm sự phụ thuộc giữa sender và receiver.

14. **Command**
    - Đóng gói một request thành object, cho phép log, undo/redo, queue.
    - → Tách biệt *người gửi* và *người thực hiện*.

15. **Interpreter**
    - Xác định ngữ pháp cho một ngôn ngữ đơn giản và cung cấp bộ thông dịch cho nó.
    - → Áp dụng khi cần xử lý ngôn ngữ/domain-specific language.

16. **Iterator**
    - Cung cấp một cách tuần tự để duyệt qua các phần tử trong collection mà không tiết lộ cấu trúc bên trong.

17. **Mediator**
    - Định nghĩa một đối tượng trung gian quản lý giao tiếp giữa nhiều object.
    - → Giảm phụ thuộc chéo, tránh mối quan hệ “mạng nhện”.

18. **Memento**
    - Lưu trữ trạng thái nội bộ của object để có thể khôi phục sau này, mà không vi phạm encapsulation.

19. **Observer**
    - Thiết lập quan hệ *một – nhiều*: khi một object thay đổi trạng thái, các object phụ thuộc được thông báo tự động.

20. **State**
    - Cho phép object thay đổi hành vi khi trạng thái nội bộ của nó thay đổi.
    - → Như thể class của object thay đổi tại runtime.

21. **Strategy**
    - Định nghĩa một họ thuật toán, đóng gói và làm cho chúng có thể thay thế lẫn nhau.
    - → Cho phép chọn thuật toán phù hợp tại runtime.

22. **Template Method**
    - Xác định khung (skeleton) của một thuật toán trong phương thức, để subclass định nghĩa chi tiết các bước.

23. **Visitor**
    - Cho phép định nghĩa thêm các thao tác trên một cấu trúc object mà không thay đổi class của chúng.
    - → Tách riêng “hành vi” ra khỏi “cấu trúc dữ liệu”.




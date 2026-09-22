# ACID và Isolation Levels trong Database

## ACID là gì?

**ACID** là tập hợp 4 tính chất đảm bảo độ tin cậy của các giao dịch (transaction) trong cơ sở dữ liệu, đặc biệt quan trọng với hệ quản trị CSDL quan hệ (RDBMS) như MySQL, PostgreSQL, Oracle...

### 1. Atomicity (Tính nguyên tử)

Một giao dịch được xem là **một đơn vị không thể chia nhỏ**: hoặc thực hiện thành công toàn bộ, hoặc không thực hiện gì cả (rollback hoàn toàn).

*Ví dụ:* Chuyển tiền từ tài khoản A sang B gồm 2 bước: trừ tiền A, cộng tiền B. Nếu bước 2 lỗi, bước 1 cũng phải được hủy — không thể để tiền "biến mất".

### 2. Consistency (Tính nhất quán)

Giao dịch phải đưa dữ liệu từ **trạng thái hợp lệ này sang trạng thái hợp lệ khác**, tuân thủ mọi ràng buộc (constraints), quy tắc, khóa ngoại đã định nghĩa.

*Ví dụ:* Nếu quy định số dư tài khoản không được âm, thì không giao dịch nào được phép khiến số dư âm.

### 3. Isolation (Tính cô lập)

Nhiều giao dịch chạy đồng thời **không được ảnh hưởng lẫn nhau** — kết quả cuối cùng phải giống như khi chúng chạy tuần tự.

*Ví dụ:* Hai người cùng đặt vé máy bay cho chuyến còn 1 ghế trống — hệ thống phải đảm bảo chỉ một người đặt thành công, không xảy ra tình trạng cả hai đều thấy "còn ghế" và đặt trùng.

### 4. Durability (Tính bền vững)

Khi giao dịch đã **commit** (hoàn tất) thành công, dữ liệu phải được lưu trữ vĩnh viễn, kể cả khi hệ thống bị mất điện hay crash ngay sau đó.

### Tóm tắt nhanh

| Chữ   | Ý nghĩa     | Đảm bảo điều gì                  |
|-------|-------------|----------------------------------|
| **A** | Atomicity   | Tất cả hoặc không gì cả          |
| **C** | Consistency | Dữ liệu luôn hợp lệ              |
| **I** | Isolation   | Giao dịch không "giẫm chân" nhau |
| **D** | Durability  | Đã lưu là không mất              |

> ACID thường được so sánh với mô hình **BASE** (dùng trong nhiều hệ CSDL NoSQL), vốn đánh đổi tính nhất quán chặt chẽ để lấy khả năng mở rộng và hiệu năng cao hơn.

---

## Isolation Levels (Các cấp độ cô lập)

Có **4 cấp độ chuẩn** theo SQL standard. Ở cấp thấp nhất, transaction *có thể* đọc được dữ liệu chưa commit của transaction khác (gọi là **dirty read**).

### Các hiện tượng cần tránh (Concurrency Phenomena)

| Hiện tượng              | Mô tả                                                                                                                    |
|-------------------------|--------------------------------------------------------------------------------------------------------------------------|
| **Dirty Read**          | Đọc dữ liệu mà transaction khác *chưa commit* — nếu họ rollback, dữ liệu bạn đọc là "rác"                                |
| **Non-repeatable Read** | Đọc cùng 1 dòng 2 lần trong 1 transaction nhưng kết quả khác nhau (vì transaction khác đã UPDATE và commit ở giữa)       |
| **Phantom Read**        | Chạy cùng 1 câu query 2 lần nhưng số lượng dòng trả về khác nhau (vì transaction khác đã INSERT/DELETE và commit ở giữa) |

### 1. Read Uncommitted (thấp nhất)

- Cho phép đọc dữ liệu **chưa commit** của transaction khác → có **dirty read**
- Nhanh nhất nhưng rủi ro cao nhất

*Ví dụ:* A đang chuyển tiền (chưa commit), B đọc thấy số dư đã tăng, nhưng sau đó A rollback → B đã dùng dữ liệu sai.

### 2. Read Committed

- Chỉ đọc dữ liệu **đã commit** → tránh được dirty read
- Nhưng vẫn có thể gặp **non-repeatable read** và **phantom read**
- Đây là mức **mặc định** của PostgreSQL, Oracle, SQL Server.

### 3. Repeatable Read

- Đảm bảo nếu đọc lại cùng 1 dòng trong transaction, giá trị không đổi → tránh **non-repeatable read**
- Vẫn có thể gặp **phantom read** (theo chuẩn SQL) — dù MySQL InnoDB thực tế đã khắc phục phần lớn phantom read ở mức này nhờ cơ chế gap lock
- Đây là mức **mặc định** của MySQL (InnoDB).

### 4. Serializable (cao nhất)

- Các transaction chạy như thể chúng được thực hiện **tuần tự**, không có gì chồng chéo
- Tránh được cả 3 hiện tượng trên
- An toàn nhất nhưng **hiệu năng thấp nhất** do phải khóa nhiều hoặc dùng cơ chế kiểm tra xung đột phức tạp (như MVCC + validation).

### Bảng tổng hợp

| Isolation Level  | Dirty Read | Non-repeatable Read | Phantom Read |
|------------------|:----------:|:-------------------:|:------------:|
| Read Uncommitted |  ✅ Có thể  |      ✅ Có thể       |   ✅ Có thể   |
| Read Committed   |  ❌ Không   |      ✅ Có thể       |   ✅ Có thể   |
| Repeatable Read  |  ❌ Không   |       ❌ Không       |  ✅ Có thể*   |
| Serializable     |  ❌ Không   |       ❌ Không       |   ❌ Không    |

*\*Theo chuẩn SQL. Thực tế triển khai có thể khác (MySQL InnoDB xử lý tốt hơn ở mức này).*

> Càng lên cấp cao, dữ liệu càng an toàn nhưng **hiệu năng và khả năng đồng thời (concurrency) càng giảm** — đây là sự đánh đổi (trade-off) mà người thiết kế hệ thống phải cân nhắc tùy theo yêu cầu nghiệp vụ.

---

## Mức Isolation mặc định của từng hệ CSDL

| Hệ CSDL            | Mức mặc định                               |
|--------------------|--------------------------------------------|
| **MySQL (InnoDB)** | **Repeatable Read**                        |
| **PostgreSQL**     | **Read Committed**                         |
| **Oracle**         | **Read Committed**                         |
| **SQL Server**     | **Read Committed**                         |
| **SQLite**         | **Serializable** (do khóa cả file khi ghi) |

### Cách thay đổi isolation level

### Cách kiểm tra mức hiện tại

## Cơ chế bên dưới: MVCC (Multi-Version Concurrency Control)

Đây là cơ chế giúp Repeatable Read đảm bảo đọc 2 lần ra cùng kết quả, dù có transaction khác commit ở giữa.

### Ý tưởng cốt lõi

Thay vì `UPDATE` ghi đè trực tiếp, DB **giữ lại nhiều phiên bản (version)** của cùng 1 dòng dữ liệu, mỗi phiên bản gắn với thông tin transaction nào tạo ra nó:

### Cơ chế Snapshot

Khi transaction bắt đầu (ở Repeatable Read), DB gán cho nó 1 **snapshot** — trả lời câu hỏi: "tại thời điểm này, những transaction nào đã commit?". Mỗi lần `SELECT`, DB chỉ hiển thị phiên bản dữ liệu nằm trong snapshot đó — dù trên đĩa đã có version mới hơn.

### Dọn dẹp version cũ

- **PostgreSQL**: tiến trình `VACUUM` chạy nền để dọn các version cũ (dead tuple) không còn transaction nào cần đọc.
- **MySQL InnoDB**: cơ chế **undo log + purge thread**.

### So sánh Read Committed vs Repeatable Read (dưới góc độ MVCC)

|                       | Read Committed                                 | Repeatable Read                                 |
|-----------------------|------------------------------------------------|-------------------------------------------------|
| Khi nào chụp snapshot | **Mỗi câu SELECT** chụp 1 snapshot mới         | **Chỉ chụp 1 lần** khi transaction bắt đầu      |
| Kết quả               | Luôn thấy commit mới nhất tại thời điểm SELECT | Luôn thấy đúng 1 bức ảnh xuyên suốt transaction |

### Còn Serializable thì sao?

MVCC snapshot chỉ giải quyết vấn đề **đọc**. Với Serializable, PostgreSQL dùng thêm **SSI (Serializable Snapshot Isolation)** — theo dõi các **write dependency** giữa các transaction để phát hiện xung đột, và sẽ **abort** một trong hai nếu phát hiện chu trình xung đột (tương tự deadlock detection nhưng cho logic, không chỉ cho lock).

---

*Tài liệu này tổng hợp kiến thức về ACID, Isolation Levels và cơ chế MVCC trong các hệ quản trị CSDL quan hệ phổ biến (PostgreSQL, MySQL, Oracle, SQL Server).*
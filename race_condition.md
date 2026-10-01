# Race Condition trong Backend

## 1. Race condition là gì

**Race condition** xảy ra khi kết quả cuối cùng của hệ thống phụ thuộc vào **thứ tự/thời điểm** thực thi của nhiều luồng xử lý (request, thread, process, instance) cùng truy cập và thay đổi một **tài nguyên dùng chung** (row DB, key cache, file, biến bộ nhớ...). Khi các thao tác đó **không atomic**, các luồng có thể chen ngang nhau, làm dữ liệu sai lệch.

**Ba điều kiện cần để xảy ra race condition:**
1. Có **tài nguyên dùng chung** (shared state).
2. Có **truy cập đồng thời** (nhiều request/instance chạy song song).
3. Có ít nhất 1 luồng **ghi**, và thao tác gồm **nhiều bước không atomic** (đọc → tính toán → ghi).

Loại bỏ được 1 trong 3 điều kiện trên là loại bỏ được race condition — mọi kỹ thuật bên dưới đều xoay quanh việc **biến chuỗi thao tác nhiều bước thành 1 bước atomic**, hoặc **serialize** (xếp hàng) các luồng truy cập.

> Lưu ý: lock trong code (mutex, `synchronized`, `sync.Mutex`) chỉ có tác dụng trong **1 process**. Backend thực tế chạy nhiều instance/pod nên lock trong bộ nhớ gần như vô dụng — phải đẩy việc đảm bảo atomic xuống tầng dùng chung: **DB** hoặc **distributed lock** (Redis, Zookeeper, etcd).

---

## 2. Các dạng race condition phổ biến

| Dạng | Mô tả | Hệ quả |
|---|---|---|
| **Check-then-act** | Kiểm tra điều kiện (SELECT) rồi mới hành động (INSERT/UPDATE) dựa trên kết quả kiểm tra. Giữa 2 bước, luồng khác có thể đã thay đổi dữ liệu. | Dữ liệu trùng lặp, vi phạm ràng buộc duy nhất |
| **Read-modify-write (Lost update)** | Đọc giá trị lên app, tính giá trị mới ở app, ghi đè xuống DB. 2 luồng cùng đọc giá trị cũ → luồng ghi sau đè mất kết quả của luồng ghi trước. | Số liệu sai (tồn kho, số dư, counter) |
| **Duplicate request** *(vấn đề idempotency, chỉ trở thành race khi các bản trùng đến đồng thời — xem 2.3)* | Cùng 1 request logic bị gửi nhiều lần (double-click, client retry, webhook at-least-once, message queue deliver lại). | Xử lý trùng: trừ tiền 2 lần, tạo 2 order |
| **Write skew** | 2 transaction cùng đọc 1 tập dữ liệu, mỗi bên ghi vào **row khác nhau**, mỗi bên đều hợp lệ riêng lẻ nhưng gộp lại vi phạm ràng buộc nghiệp vụ. Row lock không bắt được vì không ghi cùng row. | Vi phạm invariant liên row |
| **Phantom** | Transaction đọc theo điều kiện (range), transaction khác insert row mới thỏa điều kiện đó → lần đọc sau thấy "row ma". | Kiểm tra dựa trên tập hợp bị sai |
| **Partial failure / khoảng hở giữa các bước** | Các bước liên quan nằm ở nhiều transaction/service khác nhau; bước đầu đã commit, bước sau lỗi, bước bù trừ chạy chậm. | Trạng thái nửa vời, luồng khác đọc phải dữ liệu sai tạm thời |

### Ví dụ minh họa từng dạng

#### 2.1. Check-then-act — Đăng ký tài khoản trùng email

**Kịch bản:** API đăng ký kiểm tra email đã tồn tại chưa, nếu chưa thì tạo tài khoản. User bấm "Đăng ký" 2 lần, hoặc 2 người cùng đăng ký 1 email.

```sql
SELECT id FROM users WHERE email = 'a@x.com';   -- không có
INSERT INTO users (email, ...) VALUES ('a@x.com', ...);
```

| Thời điểm | Request A | Request B |
|---|---|---|
| T1 | `SELECT` → không có `a@x.com` | |
| T2 | | `SELECT` → không có `a@x.com` |
| T3 | `INSERT` thành công | |
| T4 | | `INSERT` thành công |

**Kết quả:** có 2 tài khoản cùng email `a@x.com`. Cả 2 request đều kiểm tra đúng tại thời điểm chúng đọc, nhưng kết quả kiểm tra đã **lỗi thời** khi tới bước ghi.

**Xử lý:** `UNIQUE(email)`, INSERT thẳng và bắt lỗi duplicate key (mục 4.1).

---

#### 2.2. Read-modify-write (Lost update) — Trừ tồn kho

**Kịch bản:** sản phẩm còn `stock = 1`. Code đọc tồn kho lên app, kiểm tra còn hàng, tính giá trị mới rồi ghi đè.

```sql
SELECT stock FROM products WHERE id = 1;          -- đọc ra 1
-- app: if stock >= 1 → newStock = stock - 1 = 0
UPDATE products SET stock = 0 WHERE id = 1;
```

| Thời điểm | Request A | Request B | `stock` trong DB |
|---|---|---|---|
| T1 | đọc `stock = 1` | | 1 |
| T2 | | đọc `stock = 1` | 1 |
| T3 | `UPDATE stock = 0`, tạo order | | 0 |
| T4 | | `UPDATE stock = 0`, tạo order | 0 |

**Kết quả:** bán được **2 sản phẩm** trong khi kho chỉ có 1, mà `stock` vẫn hiển thị 0 nên khó phát hiện. Phép trừ của A bị B **ghi đè mất**.

**Biến thể khác:** số dư ví = 100, A nạp +50, B trừ −30 cùng lúc. A đọc 100 và ghi 150, B đọc 100 và ghi 70. Ai ghi sau thì thắng, số dư cuối là 70 hoặc 150 thay vì 120.

**Xử lý:** `UPDATE ... SET stock = stock - 1 WHERE id = ? AND stock >= 1` (mục 4.2), hoặc `SELECT ... FOR UPDATE` (mục 4.4), hoặc cột `version` (mục 4.5).

---

#### 2.3. Duplicate request — Double-click chuyển khoản

**Kịch bản:** user bấm "Chuyển 1.000.000đ" và mạng chậm nên bấm thêm lần nữa. Hoặc client gửi request, server xử lý xong nhưng response bị mất do timeout, nên client tự retry.

| Thời điểm | Request 1 | Request 2 (trùng) |
|---|---|---|
| T1 | trừ A 1.000.000đ, cộng B 1.000.000đ, commit | |
| T2 | response bị timeout, client không nhận được | |
| T3 | | client retry, server coi là giao dịch mới: trừ A thêm 1.000.000đ |

**Kết quả:** A bị trừ **2.000.000đ**. Mỗi request đơn lẻ đều hợp lệ (đủ số dư, đúng logic), nên lock hay atomic update **không chặn được**: chúng bảo vệ dữ liệu khỏi chen ngang, nhưng không biết 2 request này thực chất là **cùng 1 ý định**.

**Trường hợp tương tự:** payment gateway gửi webhook "thanh toán thành công" 2 lần, consumer Kafka nhận lại message sau khi rebalance.

**Đây có phải race condition không?** Theo định nghĩa chặt thì **không hẳn**. Bản chất là bài toán **idempotency** (xử lý trùng lặp), và có 2 trường hợp:
- **Bản trùng đến tuần tự**, như ví dụ trên: request 1 đã commit xong thì retry mới tới. Không có 2 luồng nào chạy song song, nên **không có race**. Lỗi nằm ở chỗ hệ thống không nhận ra đây là cùng 1 thao tác. Kể cả server chỉ chạy 1 luồng thì lỗi vẫn xảy ra.
- **Bản trùng đến đồng thời**: double-click khiến 2 request tới gần như cùng lúc, hoặc gateway gửi 2 webhook sát nhau. Khi đó **có race**, nằm ở chính bước chống trùng. Nếu code kiểm tra `SELECT ... WHERE idempotency_key = ?` rồi mới xử lý, cả 2 request cùng thấy key chưa tồn tại và cùng chạy, đây chính là **check-then-act** (2.1).

Vì vậy giải pháp phải xử lý được cả 2 trường hợp:
1. Có **idempotency key** để nhận diện "cùng 1 ý định". Cách này giải quyết trường hợp tuần tự.
2. Việc ghi nhận key phải **atomic**, tức là INSERT kèm `UNIQUE`, không SELECT trước. Cách này giải quyết trường hợp đồng thời.

Dạng này được xếp vào đây vì trong thực tế nó luôn đi cùng race condition và dùng chung bộ công cụ xử lý (unique constraint, transaction).

**Xử lý:** idempotency key kết hợp unique constraint (mục 4.7).

---

#### 2.4. Write skew — Lịch trực bác sĩ

**Kịch bản:** quy định mỗi ca phải có **ít nhất 1 bác sĩ** trực. Ca hiện có 2 bác sĩ là Alice và Bob. Khi xin nghỉ, hệ thống đếm số người đang trực, nếu ≥ 2 thì cho nghỉ.

```sql
SELECT COUNT(*) FROM shifts WHERE shift_id = 10 AND on_call = true;   -- 2
UPDATE shifts SET on_call = false WHERE shift_id = 10 AND doctor = ?;
```

| Thời điểm | Alice xin nghỉ | Bob xin nghỉ |
|---|---|---|
| T1 | `COUNT` → 2 (≥ 2, được nghỉ) | |
| T2 | | `COUNT` → 2 (≥ 2, được nghỉ) |
| T3 | `UPDATE` row **của Alice** → `on_call = false` | |
| T4 | | `UPDATE` row **của Bob** → `on_call = false` |

**Kết quả:** ca trực còn **0 bác sĩ**. Điểm khác với lost update là 2 transaction ghi vào **2 row khác nhau**, nên row lock không xung đột và ngay cả `REPEATABLE READ` cũng không phát hiện được. Từng transaction riêng lẻ đều đúng, gộp lại thì vi phạm ràng buộc.

**Trường hợp tương tự:** user có 2 tài khoản và ràng buộc là tổng số dư 2 tài khoản ≥ 0. Rút tiền đồng thời từ 2 tài khoản, mỗi bên kiểm tra tổng thấy vẫn đủ, nên cả 2 cùng rút.

**Xử lý:** `SELECT ... FOR UPDATE` tất cả các row liên quan đến ràng buộc (mục 4.4), dùng `SERIALIZABLE`, hoặc thiết kế lại để ràng buộc nằm trên 1 row (ví dụ cột `on_call_count` trên bảng ca trực, rồi atomic update `WHERE on_call_count >= 2`).

---

#### 2.5. Phantom — Đặt phòng họp trùng khung giờ

**Kịch bản:** đặt phòng họp, quy định không được có 2 booking chồng giờ nhau. Hệ thống kiểm tra khung giờ còn trống rồi mới insert.

```sql
SELECT COUNT(*) FROM bookings
WHERE room_id = 5 AND start_time < '10:00' AND end_time > '09:00';    -- 0
INSERT INTO bookings (room_id, start_time, end_time) VALUES (5, '09:00', '10:00');
```

| Thời điểm | Request A (9h–10h) | Request B (9h30–10h30) |
|---|---|---|
| T1 | kiểm tra khoảng 9h–10h → 0 booking | |
| T2 | | kiểm tra khoảng 9h30–10h30 → 0 booking |
| T3 | `INSERT` booking 9h–10h | |
| T4 | | `INSERT` booking 9h30–10h30 |

**Kết quả:** phòng bị đặt chồng 30 phút. Booking của A là một **"row ma"** (phantom): nó chưa tồn tại lúc B kiểm tra, nên B không thể lock nó. `SELECT ... FOR UPDATE` thông thường chỉ khóa **các row đang tồn tại**, không khóa được row chưa sinh ra.

**Trường hợp tương tự:** mỗi user tối đa 3 đơn đang chờ xử lý. `COUNT` ra 2, cả 2 request cùng insert, kết quả user có 4 đơn.

**Xử lý:**
- Lock một row "cha" đại diện cho cả tập (`SELECT ... FROM rooms WHERE id = 5 FOR UPDATE`), để mọi request đặt cùng phòng phải xếp hàng.
- Dùng range/gap lock (InnoDB `REPEATABLE READ` kèm `FOR UPDATE` trên index phù hợp) hoặc `SERIALIZABLE`.
- Dùng ràng buộc DB: PostgreSQL `EXCLUDE USING gist (room_id WITH =, tsrange(start_time, end_time) WITH &&)`. Nếu chia slot cố định thì dùng `UNIQUE(room_id, slot)`.

---

#### 2.6. Partial failure — Trừ kho và tạo order ở 2 service

**Kịch bản:** hệ thống microservices. Inventory service trừ kho (commit ngay), sau đó Order service tạo order. Nếu tạo order lỗi thì gọi API hoàn kho.

| Thời điểm | User 1 | User 2 | `stock` |
|---|---|---|---|
| T1 | Inventory trừ kho, commit | | 1 → 0 |
| T2 | Order service lỗi, tạo order thất bại | | 0 |
| T3 | gọi API hoàn kho, nhưng chậm hoặc lỗi mạng nên phải retry | | 0 |
| T4 | | mua sản phẩm → thấy `stock = 0` → **báo hết hàng** | 0 |
| T5 | hoàn kho thành công | | 0 → 1 |

**Kết quả:** User 2 bị từ chối dù thực tế vẫn còn hàng, vì đọc phải trạng thái trung gian sai trong **khoảng hở** giữa bước lỗi và bước bù trừ. Nếu bước hoàn kho thất bại hẳn mà không có retry, tồn kho bị **lệch vĩnh viễn**. Nếu hoàn kho bị gọi 2 lần do retry mà không idempotent, kho bị **cộng dư**.

**Xử lý:** nếu cùng DB thì gộp vào 1 transaction (mục 4.3). Nếu khác service thì dùng Saga với compensating action idempotent, retry qua queue và outbox pattern (mục 4.12). Ngoài ra, có thể dùng trạng thái giữ hàng (`RESERVED`) thay vì trừ thẳng, để phân biệt hàng đang chờ chốt đơn với hàng đã bán.

---

**Tóm tắt điểm khác nhau giữa các dạng:**
- **Check-then-act** và **Phantom:** bị chen vào giữa bước **kiểm tra sự tồn tại** và bước **ghi**. Phantom khó hơn vì đối tượng cần khóa chưa tồn tại.
- **Lost update:** 2 luồng ghi **cùng 1 row**, luồng sau đè mất kết quả của luồng trước.
- **Write skew:** 2 luồng ghi **các row khác nhau**, nhưng cùng vi phạm 1 ràng buộc chung.
- **Duplicate request:** về bản chất là bài toán **idempotency**, không phải race thuần. **Cùng 1 ý định** bị thực hiện nhiều lần, và chỉ thành race khi các bản trùng đến đồng thời.
- **Partial failure:** không phải chen ngang ở mức câu lệnh, mà là **khoảng hở về thời gian** giữa các bước không cùng transaction.

---

## 3. Nguyên tắc xuyên suốt

1. **Không bao giờ tin vào check ở tầng application** (check-then-act). Mọi kiểm tra quan trọng phải được gộp **chung với thao tác ghi** trong 1 bước atomic ở tầng lưu trữ.
2. **DB là nguồn sự thật cuối cùng.** Các lớp cache/Redis/lock phân tán chỉ là **tối ưu hiệu năng** (chặn sớm, giảm tải), không được thay thế ràng buộc ở DB.
3. **Để DB tự serialize** thay vì tự viết lock ở app khi có thể — DB đã có row lock, unique index, transaction được kiểm chứng kỹ.
4. **Gom các thay đổi liên quan vào 1 transaction thật** (cùng connection, `BEGIN ... COMMIT/ROLLBACK`), để chúng cùng thành công hoặc cùng thất bại.
5. **Chọn mức độ chặt theo nghiệp vụ**: ràng buộc cứng (tiền, tồn kho, ghế) → strong consistency; số liệu thống kê chấp nhận sai lệch nhỏ → eventual consistency để đổi lấy throughput.
6. **Mọi thao tác có thể bị gọi lại phải idempotent.**

---

## 4. Các kỹ thuật xử lý

### 4.1. Unique Constraint / Unique Index

**Cơ chế:** DB kiểm tra tính duy nhất và ghi trong **cùng 1 operation ở tầng storage engine**, nên atomic tuyệt đối. Unique index hoạt động như một "lock tự nhiên" trên giá trị khóa.

**Cách dùng:**
- Khai báo `UNIQUE` trên cột hoặc **tổ hợp cột** thể hiện ràng buộc nghiệp vụ "chỉ được tồn tại 1".
- Application **INSERT thẳng**, không SELECT trước. Bắt lỗi *duplicate key / unique violation* và chuyển thành lỗi nghiệp vụ.
- Với ràng buộc chỉ áp dụng cho bản ghi còn hiệu lực: dùng **partial unique index** (PostgreSQL: `UNIQUE ... WHERE status = 'active'`) hoặc cột phụ hỗ trợ.

**Giải quyết:** check-then-act, duplicate request (khi kết hợp idempotency key), giới hạn "mỗi X chỉ được 1 lần với Y".

**Ưu điểm:** đơn giản, không cần lock tay, đúng tuyệt đối, không phụ thuộc isolation level.
**Hạn chế:** chỉ biểu diễn được ràng buộc dạng "duy nhất", không biểu diễn được ràng buộc số lượng (≤ N, ≥ 0).

---

### 4.2. Atomic Update có điều kiện (Conditional Update)

**Cơ chế:** đưa điều kiện kiểm tra vào chính câu `UPDATE`, để DB gộp "đọc giá trị mới nhất + kiểm tra + ghi" thành 1 bước dưới row lock:

```sql
UPDATE <table> SET <col> = <col> - :n WHERE id = :id AND <col> >= :n;
```

Sau đó kiểm tra `affected_rows`: `1` → thành công, `0` → điều kiện không thỏa (hết hàng, không đủ số dư, đã bị đặt...).

**Vì sao đúng kể cả khi transaction chưa commit — "first updater wins":**
- `UPDATE` luôn xin **row-level exclusive lock**, bất kể isolation level.
- Luồng thứ 2 chạm vào cùng row sẽ **bị block** đến khi luồng 1 commit/rollback.
- Khi được đánh thức, DB **re-evaluate điều kiện `WHERE` trên dữ liệu mới nhất đã commit**, không dựa vào snapshot cũ.
  - Luồng 1 commit → điều kiện của luồng 2 có thể không còn đúng → `affected_rows = 0`.
  - Luồng 1 rollback → dữ liệu trở về gốc → luồng 2 update bình thường.

**Khác biệt giữa các DB:**
- **MySQL/InnoDB:** chờ lock rồi re-check theo latest committed data (semi-consistent read), lặng lẽ trả `affected_rows = 0`.
- **PostgreSQL:** ở `READ COMMITTED` re-evaluate tương tự (EvalPlanQual). Ở `REPEATABLE READ`/`SERIALIZABLE` sẽ **ném lỗi serialization failure** (`could not serialize access due to concurrent update`), app phải bắt lỗi và retry cả transaction.

**Giải quyết:** lost update, ràng buộc số lượng (không âm, không vượt giới hạn), chuyển trạng thái (`available → booked`).

**Ưu điểm:** chỉ 1 round-trip, không retry ở app, phù hợp nhất cho tranh chấp cao.
**Hạn chế:** chỉ dùng được khi logic kiểm tra biểu diễn được bằng điều kiện SQL trên chính row đó.

**Anti-pattern cần tránh:** `SELECT` giá trị lên app → tính toán → `UPDATE SET col = <giá trị app tự tính>` không kèm điều kiện. `SELECT` thường không lấy lock, với MVCC mỗi transaction đọc snapshot riêng → lost update.

---

### 4.3. Transaction & Isolation Level

**Transaction** đảm bảo **Atomicity** cho nhóm thao tác: tất cả cùng commit hoặc cùng rollback, không để trạng thái nửa vời. Bản thân transaction **không tự chống race condition** — nó cần đi kèm lock (ngầm qua UPDATE, hoặc tường minh qua `FOR UPDATE`) hoặc isolation level đủ cao.

**Nguyên tắc:**
- Các bước liên quan (trừ tài nguyên + tạo bản ghi + ghi idempotency key...) phải nằm trong **cùng 1 transaction thật trên cùng 1 connection**. Nếu tách ra nhiều transaction và "rollback" bằng thao tác bù trừ thủ công → xuất hiện **khoảng hở** mà luồng khác đọc phải dữ liệu sai.
- Khi rollback, DB hoàn tác toàn bộ và **giải phóng lock ngay lập tức** trong 1 bước.
- Giữ transaction **ngắn**: không gọi API ngoài, không chờ I/O chậm trong transaction → giảm thời gian giữ lock.
- Đặt thao tác **dễ fail nhất / fail nhanh nhất** lên trước để rollback sớm.

**Isolation level và các hiện tượng:**

| Isolation level | Dirty read | Non-repeatable read | Phantom read | Lost update / Write skew |
|---|---|---|---|---|
| READ UNCOMMITTED | Có | Có | Có | Có |
| READ COMMITTED | Không | Có | Có | Có (nếu dùng read-modify-write) |
| REPEATABLE READ | Không | Không | Tùy DB (InnoDB chặn phần lớn bằng gap lock) | PostgreSQL phát hiện lost update và báo lỗi; write skew vẫn có thể xảy ra |
| SERIALIZABLE | Không | Không | Không | Không (DB báo lỗi serialization, app phải retry) |

**Lưu ý:**
- Atomic update (`UPDATE ... WHERE`) và unique constraint **an toàn ngay ở READ COMMITTED**, không cần nâng isolation level.
- Nâng lên `SERIALIZABLE` giải quyết được write skew/phantom nhưng làm giảm throughput và bắt buộc app có **retry logic** cho serialization failure.

---

### 4.4. Pessimistic Lock (Khóa bi quan)

**Cơ chế:** giả định xung đột **sẽ xảy ra**, nên chủ động khóa dữ liệu ngay từ lúc đọc:

```sql
SELECT ... FROM <table> WHERE id = :id FOR UPDATE;
```

Transaction khác muốn ghi (hoặc `FOR UPDATE`) cùng row phải **chờ** đến khi transaction hiện tại commit/rollback.

**Các biến thể:**
- `FOR UPDATE` — khóa ghi (exclusive).
- `FOR SHARE` / `LOCK IN SHARE MODE` — khóa đọc chia sẻ, chặn người khác ghi nhưng cho phép cùng đọc.
- `FOR UPDATE NOWAIT` — không chờ, lỗi ngay nếu row đang bị khóa (fail fast).
- `FOR UPDATE SKIP LOCKED` — bỏ qua các row đang bị khóa, rất hữu ích cho **job queue trên DB** (nhiều worker lấy việc không trùng nhau).
- Lock theo khoảng (gap/next-key lock ở InnoDB) khi `FOR UPDATE` theo điều kiện range → chống phantom.

**Khi nào dùng:** cần đọc dữ liệu lên app để xử lý logic phức tạp (nhiều điều kiện, nhiều bảng) mà không gói gọn được trong 1 câu `UPDATE ... WHERE`. Cũng dùng để chống **write skew** bằng cách khóa tường minh các row liên quan.

**Ưu điểm:** đúng tuyệt đối, không cần retry ở app, phù hợp tranh chấp cao.
**Hạn chế:**
- Tốn thêm round-trip (SELECT rồi mới UPDATE), giữ lock lâu hơn atomic update.
- Dễ **deadlock** khi khóa nhiều row theo thứ tự không nhất quán.
- Dễ nghẽn connection pool nếu transaction giữ lock lâu.
- Chỉ có tác dụng **trong transaction** (ngoài transaction / autocommit thì lock nhả ngay).

---

### 4.5. Optimistic Lock (Khóa lạc quan)

**Cơ chế:** giả định xung đột **hiếm khi xảy ra**, không khóa khi đọc. Mỗi row có cột `version` (hoặc `updated_at`). Khi ghi, kiểm tra version không đổi:

```sql
UPDATE <table> SET ..., version = version + 1 WHERE id = :id AND version = :old_version;
```

`affected_rows = 0` → đã có luồng khác ghi trước → app phát hiện **conflict**, phải đọc lại và retry hoặc báo lỗi cho user.

**Khi nào dùng:**
- Tranh chấp **thấp**: user sửa dữ liệu của riêng mình, form chỉnh sửa có thể bị 2 người cùng mở.
- Luồng xử lý **dài, có tương tác người dùng** (đọc → user chỉnh sửa vài phút → lưu) — không thể giữ DB lock suốt thời gian đó.
- Hệ thống không hỗ trợ lock (NoSQL, REST API với `ETag` / `If-Match`).

**Không nên dùng khi tranh chấp cao (hotspot):**
- N request cùng nhắm 1 row → mỗi vòng chỉ 1 request thắng, N−1 request fail và retry → **retry storm**.
- Tổng số query xuống DB tăng vọt so với số request gốc, độ trễ không đều giữa các user, cần thêm backoff/jitter → phức tạp.
- Khi đó atomic update hoặc pessimistic lock tốt hơn vì DB tự xếp hàng, không lãng phí công sức.

---

### 4.6. So sánh Atomic Update – Pessimistic – Optimistic

| Tiêu chí | Atomic Update | Pessimistic Lock | Optimistic Lock |
|---|---|---|---|
| Thời điểm phát hiện xung đột | Khi ghi (DB tự chờ & re-check) | Khi đọc (chờ lock) | Khi ghi (version lệch) |
| Retry ở app | Không | Không | Có |
| Số round-trip | 1 | ≥ 2 | ≥ 2 (nhân thêm số lần retry) |
| Thời gian giữ lock | Ngắn nhất | Dài (từ SELECT đến COMMIT) | Không giữ lock |
| Nguy cơ deadlock | Có (khi update nhiều row) | Cao hơn | Không |
| Phù hợp | Logic đơn giản, tranh chấp cao | Logic phức tạp, tranh chấp cao | Tranh chấp thấp, luồng dài, không giữ được lock |

**Quy tắc chọn:**
- Logic biểu diễn được bằng 1 câu `UPDATE ... WHERE` → **Atomic update**.
- Logic phức tạp, cần đọc nhiều dữ liệu rồi mới quyết định, tranh chấp cao → **Pessimistic lock**.
- Tranh chấp thấp, hoặc luồng xử lý kéo dài qua tương tác người dùng → **Optimistic lock**.
- Ràng buộc dạng "duy nhất" → **Unique constraint** (kết hợp được với cả 3 cách trên).

---

### 4.7. Idempotency Key

**Vấn đề:** cùng 1 request logic bị gửi nhiều lần (double-click, client/gateway retry, at-least-once delivery của webhook/message queue). Đây không phải tranh chấp giữa nhiều user mà là **trùng lặp từ cùng 1 nguồn**.

**Cơ chế:**
- Mỗi thao tác logic mang 1 key duy nhất: client sinh UUID cho mỗi hành động, hoặc dùng ID có sẵn từ hệ thống ngoài (transaction ID của payment gateway, event ID, message ID).
- Bảng lưu key có `UNIQUE(idempotency_key)`. **INSERT key trong cùng transaction** với thao tác nghiệp vụ, và insert **trước** các thao tác nghiệp vụ.
  - Insert thành công → lần đầu, xử lý tiếp.
  - Vi phạm unique → đã xử lý → **không làm lại**, trả về kết quả của lần xử lý trước (lưu kèm response/status theo key).
- Vì chung transaction, nếu nghiệp vụ lỗi thì key cũng bị rollback → lần retry sau vẫn được xử lý bình thường, không bị khóa nhầm.

**Lưu ý:**
- Với request đang xử lý dở (key đã có nhưng chưa có kết quả), lần gọi trùng nên trả trạng thái "đang xử lý" thay vì xử lý lại.
- Key nên có thời hạn lưu (TTL / dọn định kỳ) phù hợp với cửa sổ retry.
- Với webhook: **trả HTTP 200 nhanh** cả khi nhận bản trùng để bên gửi ngừng retry; luôn **verify chữ ký (HMAC)** trước khi xử lý — đây là lớp bảo mật độc lập với idempotency.
- Consumer của message queue phải luôn được thiết kế idempotent vì at-least-once delivery là mặc định.

---

### 4.8. Distributed Lock (Redis / Zookeeper / etcd)

**Khi nào cần:** tài nguyên hoặc logic cần bảo vệ **nằm ngoài DB** (giữ chỗ tạm trong cache, gọi API bên thứ 3, job chỉ được chạy 1 instance tại 1 thời điểm), hoặc cần chặn sớm trước khi chạm DB.

**Redis lock cơ bản:**
```
SET lock:<resource> <owner_token> NX EX <ttl>
```
- `NX`: chỉ set nếu key chưa tồn tại → atomic nhờ Redis xử lý lệnh **đơn luồng**, chỉ đúng 1 client giành được lock.
- `EX`: TTL bắt buộc — nếu process crash khi đang giữ lock, lock tự giải phóng, tránh khóa chết vĩnh viễn.
- `owner_token`: giá trị ngẫu nhiên riêng cho mỗi client. Khi nhả lock phải **kiểm tra đúng token của mình rồi mới xóa** (dùng Lua script để gộp "so sánh + xóa" thành atomic), tránh xóa nhầm lock của client khác sau khi lock của mình đã hết hạn.

**Các rủi ro cần biết:**
- **TTL hết hạn giữa chừng** (GC pause, xử lý chậm) → 2 client cùng tưởng mình giữ lock. Giảm thiểu bằng cách gia hạn lock (watchdog) hoặc dùng **fencing token** (số tăng dần đi kèm lock, tầng lưu trữ từ chối ghi có token cũ hơn).
- Redis restart không bật persistence, failover master–replica bị mất key → lock bị mất.
- Redlock (lock trên nhiều node Redis) tăng độ sẵn sàng nhưng vẫn còn tranh cãi về tính đúng đắn tuyệt đối; với yêu cầu đúng tuyệt đối nên dùng hệ thống có consensus (Zookeeper, etcd) hoặc dựa vào ràng buộc DB.

**Nguyên tắc:** distributed lock là **lớp tối ưu / điều phối**, không thay thế unique constraint hay atomic update ở DB. DB vẫn là chốt chặn cuối cùng.

---

### 4.9. Thao tác atomic của Redis làm lớp chặn sớm

Redis xử lý lệnh đơn luồng nên mỗi lệnh đơn lẻ đều atomic, thích hợp làm **lớp lọc nhanh trong memory** trước khi chạm DB:

| Lệnh | Tính chất | Dùng cho |
|---|---|---|
| `SET key val NX EX ttl` | Chỉ 1 client set thành công | Lock, giữ chỗ tạm có TTL, chặn request trùng |
| `INCR` / `DECR` / `INCRBY` | Tăng/giảm atomic, trả giá trị mới | Bộ đếm, giới hạn số lượng (so sánh kết quả trả về với ngưỡng) |
| `SADD` | Trả `1` nếu phần tử mới, `0` nếu đã có — idempotent tự nhiên | Chống trùng theo user (mỗi user 1 lần) |
| Lua script / `MULTI-EXEC` | Gộp nhiều lệnh thành 1 khối atomic | Logic kết hợp: kiểm tra + đếm + đánh dấu |

**Lưu ý:**
- Chuỗi nhiều lệnh Redis rời rạc **không atomic** — giữa 2 lệnh vẫn có thể bị chen ngang. Cần gộp bằng Lua script nếu logic phụ thuộc lẫn nhau.
- Khi Redis đã "cho qua" nhưng thao tác DB phía sau fail, cần **bù lại** trạng thái trên Redis (ví dụ `INCR` lại counter), nếu không sẽ lệch số liệu giữa Redis và DB.
- Redis là lớp tối ưu; dữ liệu quan trọng vẫn phải được đảm bảo bởi ràng buộc ở DB.

---

### 4.10. Queue / Serialize xử lý

**Cơ chế:** thay vì để hàng nghìn request song song tranh nhau 1 tài nguyên (gây lock contention, cạn connection pool), đưa request vào **hàng đợi** và xử lý **tuần tự** (hoặc partition theo key tài nguyên — mỗi key chỉ do 1 consumer xử lý).

- Kafka: partition theo `resource_id` → các message cùng resource luôn vào cùng partition, được xử lý tuần tự, không còn race giữa chúng.
- Tách luồng: API chỉ nhận request và trả "đang xử lý", kết quả trả về qua polling/notification.
- Kết hợp **rate limit**, **hàng đợi ảo** (waiting room) ở tầng gateway để chặn tải từ sớm.

**Ưu điểm:** loại bỏ tranh chấp ngay từ thiết kế, bảo vệ DB khỏi hotspot.
**Hạn chế:** xử lý bất đồng bộ, phức tạp hơn về UX và vận hành; consumer vẫn phải idempotent.

---

### 4.11. Batch / Async — chấp nhận Eventual Consistency

**Khi nào dùng:** nghiệp vụ **không có ràng buộc đúng-sai cứng**, chấp nhận sai lệch nhỏ hoặc trễ vài giây (đếm view, like, thống kê, analytics).

**Cơ chế:**
- Ghi nhận nhanh ở tầng memory (Redis `INCR`), hoặc đẩy event vào queue.
- Job định kỳ / consumer **gom (aggregate)** rồi ghi xuống DB 1 lần: `UPDATE ... SET count = count + :delta`.
- Phần cần đúng logic (mỗi user chỉ tính 1 lần) vẫn dùng cấu trúc chống trùng (Redis Set, unique constraint ở DB), tách biệt với phần đếm tổng.

**Nguyên tắc:** không lock hóa những chỗ không cần thiết — đổi strong consistency lấy throughput khi nghiệp vụ cho phép. Cần cân nhắc rủi ro mất dữ liệu trên Redis (bật persistence hoặc ghi bền vững bất đồng bộ xuống DB).

---

### 4.12. Saga & Compensating Transaction (hệ thống phân tán)

**Vấn đề:** khi các bước nằm ở **nhiều service/DB khác nhau**, không dùng chung được 1 transaction DB.

**Cơ chế Saga:** chuỗi transaction cục bộ, mỗi bước commit riêng; nếu 1 bước sau fail thì chạy **compensating transaction** để hoàn tác các bước trước.
- **Choreography:** các service phát/nhận event lẫn nhau.
- **Orchestration:** 1 orchestrator điều phối thứ tự và bù trừ.

**Yêu cầu bắt buộc:**
- Mọi bước và mọi compensating action phải **idempotent** (message có thể deliver trùng).
- Có **retry** đáng tin cậy (message queue at-least-once, outbox pattern) để không bị treo ở trạng thái nửa vời.
- Chấp nhận có **khoảng hở** (trạng thái trung gian nhìn thấy được) — thiết kế trạng thái rõ ràng (`PENDING`, `CONFIRMED`, `CANCELLED`) để các luồng khác hiểu đúng.
- **Transactional Outbox:** ghi event vào bảng outbox trong cùng transaction với thay đổi nghiệp vụ, rồi relay sang queue → tránh race giữa "commit DB" và "publish event" (commit xong nhưng publish fail, hoặc ngược lại).

---

## 5. Deadlock

**Nguyên nhân:** 2+ transaction giữ lock mà bên kia cần, và chờ lẫn nhau theo vòng tròn. Thường xảy ra khi khóa **nhiều row/bảng theo thứ tự khác nhau**.

**DB xử lý:** tự phát hiện deadlock và **abort 1 transaction** (victim), transaction còn lại tiếp tục. App nhận lỗi deadlock và cần retry.

**Phòng tránh:**
- **Lock theo thứ tự cố định** trong toàn bộ codebase (ví dụ sắp xếp theo id tăng dần trước khi update nhiều row; quy ước thứ tự bảng thao tác).
- Giữ transaction **ngắn**, không chờ I/O ngoài khi đang giữ lock.
- Khóa đúng và đủ ngay từ đầu, tránh nâng cấp lock giữa chừng (shared → exclusive).
- Đảm bảo câu lệnh dùng **index** — thiếu index khiến DB khóa nhiều row hơn cần thiết (thậm chí khóa cả range lớn).
- Dùng `NOWAIT` / lock timeout để fail fast thay vì chờ vô hạn.
- Có **retry logic** (kèm backoff) cho lỗi deadlock và serialization failure.

---

## 6. Hotspot — tranh chấp cực cao trên 1 tài nguyên

Khi rất nhiều request cùng nhắm vào **đúng 1 row**, tính đúng đắn vẫn được đảm bảo bởi atomic update/lock, nhưng **hiệu năng** là vấn đề chính (lock contention, cạn connection pool, latency tăng vọt).

**Các lớp giảm tải (từ ngoài vào trong):**
1. **Gateway:** rate limit, waiting room, chống bot.
2. **Cache/Redis:** counter atomic (`DECR`) chỉ cho tối đa N request đi tiếp, số còn lại fail nhanh.
3. **Queue:** serialize xử lý theo resource.
4. **Chia nhỏ tài nguyên (sharding counter):** tách 1 row thành nhiều row con (bucket), mỗi request trừ ở 1 bucket, tổng = tổng các bucket.
5. **DB:** atomic update làm chốt chặn cuối cùng.

**Tránh:** optimistic lock cho hotspot (retry storm).

---

## 7. Bảng tổng hợp: vấn đề → kỹ thuật

| Vấn đề | Kỹ thuật chính | Kỹ thuật bổ trợ |
|---|---|---|
| Không được trùng giá trị (định danh, 1 lần/user) | Unique constraint | Redis `SET NX` / `SADD` chặn sớm |
| Không được âm / không vượt giới hạn số lượng | Atomic update có điều kiện | Redis counter, queue cho hotspot |
| Chuyển trạng thái 1 lần (available → booked) | Atomic update có điều kiện / Unique constraint | Giữ chỗ tạm có TTL trên Redis |
| Nhiều thay đổi phải cùng thành công/thất bại | Transaction | Thứ tự lock cố định chống deadlock |
| Logic phức tạp, cần đọc rồi mới quyết định | Pessimistic lock (`FOR UPDATE`) | `NOWAIT`, `SKIP LOCKED` |
| Sửa dữ liệu ít tranh chấp, luồng xử lý dài | Optimistic lock (`version`) | ETag / If-Match ở API |
| Request/event bị gửi lặp lại | Idempotency key + Unique constraint | Lưu response theo key |
| Ràng buộc liên nhiều row (write skew) | SERIALIZABLE hoặc `FOR UPDATE` các row liên quan | Thiết kế lại để ràng buộc rơi vào 1 row / unique |
| Logic/tài nguyên nằm ngoài DB, nhiều instance | Distributed lock | Fencing token |
| Nhiều service, không chung transaction | Saga + compensating transaction | Outbox pattern, idempotent consumer |
| Traffic cực lớn, chấp nhận sai lệch nhỏ | Batch / async counter | Redis Set chống trùng |
| Hotspot tranh chấp 1 row | Atomic update + queue | Rate limit, Redis counter, sharding counter |

---

## 8. Anti-patterns thường gặp

- **Check-then-act ở application:** SELECT kiểm tra rồi INSERT/UPDATE mà không có unique constraint hoặc lock.
- **Read-modify-write không điều kiện:** đọc lên app, tính toán, ghi đè giá trị tuyệt đối xuống DB.
- **Dùng lock trong bộ nhớ** (mutex, `synchronized`) cho hệ thống chạy nhiều instance.
- **Coi Redis/distributed lock là lớp bảo vệ duy nhất**, bỏ qua ràng buộc ở DB.
- **Distributed lock không có TTL** hoặc nhả lock không kiểm tra owner.
- **Tách các bước liên quan ra nhiều transaction** rồi tự "rollback" bằng code bù trừ khi hoàn toàn có thể dùng 1 transaction.
- **Gọi API ngoài / chờ I/O lâu bên trong transaction** đang giữ lock.
- **Lock nhiều row theo thứ tự tùy ý** → deadlock.
- **Dùng optimistic lock cho hotspot** → retry storm.
- **Retry mà không idempotent** → biến lỗi tạm thời thành xử lý trùng.
- **Áp strong consistency cho mọi thứ**, kể cả số liệu thống kê không cần chính xác tức thời → nghẽn DB không cần thiết.

---

## 9. Checklist khi thiết kế 1 API có ghi dữ liệu

1. Tài nguyên dùng chung là gì? Có thể bị nhiều request ghi đồng thời không?
2. Ràng buộc nghiệp vụ là gì — duy nhất, số lượng, trạng thái, hay liên nhiều row?
3. Ràng buộc đó đã được đảm bảo **ở tầng DB** chưa (unique, điều kiện trong UPDATE, lock)?
4. Các thay đổi liên quan có nằm trong **cùng 1 transaction** không? Nếu khác service → đã có Saga/outbox chưa?
5. Request có thể bị gửi lặp lại không? → Đã có **idempotency key** chưa?
6. Có lock nhiều row không? → Thứ tự lock đã **cố định** chưa? Có retry cho deadlock chưa?
7. Mức độ tranh chấp dự kiến? → Chọn atomic/pessimistic (cao) hay optimistic (thấp); có cần lớp chặn sớm (Redis/queue/rate limit) không?
8. Nghiệp vụ có thật sự cần strong consistency, hay chấp nhận eventual consistency để đổi lấy throughput?
9. Các lớp cache/lock ngoài DB có TTL, có cơ chế bù khi DB fail, và DB vẫn là chốt chặn cuối cùng chưa?

### 1
Đăng ký tài khoản, làm sao để 2 tài khoản ko đc có email không trùng nhau, 2 request đến cùng lúc thì xử lý như nào ?

**Trả lời:**

**Sai lầm thường gặp:** check rồi mới insert (`SELECT ... WHERE email=?` → nếu không có thì `INSERT`). Đây là pattern **check-then-act**, không atomic: 2 request có thể cùng SELECT ra "chưa tồn tại" trước khi cái nào insert xong, kết quả là 2 email trùng nhau vẫn lọt qua.

**Cách xử lý đúng:**
- Tạo **UNIQUE INDEX/CONSTRAINT** trên cột `email` ở DB. Đây là lớp bảo vệ cuối cùng và bắt buộc phải có, vì DB đảm bảo tính atomic khi ghi (unique index check + insert nằm trong 1 operation ở tầng storage engine).
- Application chỉ cần `INSERT` thẳng, nếu DB trả lỗi *duplicate key / unique violation* thì catch lại và trả message "email đã tồn tại" cho user.
- Có thể thêm lớp chặn sớm ở tầng cache/application (ví dụ Redis `SETNX email_lock` hoặc distributed lock theo email) để giảm số request phải chạm DB, tăng tốc phản hồi — nhưng đây chỉ là **optimization**, không được thay thế cho unique constraint.

**Follow-up:** Mô tả flow lớp chặn sớm ở tầng cache/application (ví dụ Redis `SETNX email_lock` hoặc distributed lock theo email) ? Redis `SETNX email_lock` hoạt động như nào ?

**Trả lời:**

`SETNX` = "SET if Not eXists". Vì Redis xử lý lệnh **đơn luồng** (single-threaded command execution), lệnh này là **atomic tuyệt đối** — dù 1000 request cùng gửi `SETNX` cho cùng 1 key gần như đồng thời, Redis vẫn xử lý tuần tự từng lệnh một, nên chỉ **đúng 1** request nhận kết quả "set thành công", các request còn lại nhận kết quả "key đã tồn tại" ngay lập tức — cơ chế lock tương đương UNIQUE constraint ở DB, nhưng chạy trong memory nên cực nhanh.

**Flow cụ thể khi đăng ký:**
1. Request đăng ký với `email` đến → app gọi Redis: `SET email_lock:{email} 1 NX EX 10` (cú pháp hiện đại gộp cả NX + EX trong 1 lệnh `SET`, tương đương `SETNX` + `EXPIRE`).
   - Trả về `OK` → key chưa tồn tại, vừa được set thành công → request này "thắng", được đi tiếp.
   - Trả về `nil` → key đã tồn tại (có request khác đang xử lý email này) → **từ chối ngay lập tức**, không cần chạm tới DB, trả lỗi "email đang được xử lý" cho user.
2. Request "thắng" tiếp tục: `INSERT` vào DB như bình thường (vẫn dựa vào UNIQUE constraint ở DB làm lớp xác nhận cuối cùng).
   - Insert thành công → đăng ký OK. Có thể xoá lock (`DEL email_lock:{email}`) ngay để giải phóng sớm, hoặc cứ để tự hết hạn theo `EX`.
   - Insert thất bại (hiếm, ví dụ email đã tồn tại từ trước lúc chưa có cơ chế lock, hoặc do lock hết hạn giữa chừng) → DB vẫn là chốt chặn cuối, trả lỗi bình thường.

**Vì sao cần `EX` (TTL):** nếu app crash ngay sau khi set lock nhưng trước khi insert/xoá lock, key sẽ tồn tại vĩnh viễn trong Redis nếu không có TTL → khoá chết email đó mãi mãi, không ai đăng ký được nữa dù thực tế email chưa hề tồn tại trong DB. `EX 10` (10 giây, tuỳ chỉnh theo thời gian xử lý thực tế) đảm bảo lock tự động được giải phóng nếu có sự cố.

**Lưu ý quan trọng:** lớp này chỉ là **tối ưu hiệu năng** (chặn sớm, giảm tải DB, phản hồi nhanh cho request trùng), **không thay thế** UNIQUE constraint ở DB — vì Redis lock có thể bị bypass trong các tình huống: TTL hết hạn giữa chừng trước khi insert xong, Redis bị restart/mất dữ liệu (nếu không bật persistence), hoặc chạy nhiều cụm Redis không đồng bộ. DB UNIQUE constraint vẫn luôn là "nguồn sự thật" cuối cùng đảm bảo tính đúng đắn tuyệt đối.

### 2
Trong bảng sản phẩm, một sản phẩm có cột tồn kho, làm sao để không bao giờ để tồn kho của sản phẩm đó xuống dưới 0.
Trong trường hợp tồn kho chỉ có 1 sản phẩm cuối mà có 10000 request mua hàng đến thì xử lý như nào ?
Và trong trường hợp có 2 sản phẩm thì như nào ?

**Trả lời:**

**Kỹ thuật cốt lõi: Atomic update kèm điều kiện ngay trong câu UPDATE**, để DB tự lo việc kiểm tra + trừ trong 1 bước duy nhất, không tách ra "đọc số lượng → so sánh ở code → update":

```sql
UPDATE products
SET stock = stock - 1
WHERE id = ? AND stock >= 1;
```

Sau đó kiểm tra `affected_rows`:
- `= 1` → trừ thành công.
- `= 0` → hết hàng (do lúc chạy tới, `stock` đã không còn `>= 1` nữa), trả lỗi cho user, **không** trừ âm.

**Trường hợp 1 sản phẩm còn tồn kho = 1, có 10.000 request cùng mua:**
- Cả 10.000 request cùng UPDATE vào **đúng 1 row** → DB dùng **row-level lock**: request nào chạm vào row trước sẽ lock row đó, các request khác phải đợi tới lượt (serialize tự động ở tầng DB).
- Chỉ có **đúng 1** request thấy `stock >= 1` là true → trừ thành công (`stock` về 0). 9.999 request còn lại khi tới lượt sẽ thấy `stock >= 1` là false → `affected_rows = 0` → báo hết hàng.
- Vì vậy tồn kho **không bao giờ âm**, dù có bao nhiêu request cùng lúc — đây chính là lợi ích của việc để DB tự serialize theo row lock, thay vì tự implement lock ở application.
- Vấn đề thực tế cần lo thêm không phải là "âm kho" mà là **hiệu năng**: 10.000 request tranh 1 row gây nghẽn (lock contention/hotspot). Giải pháp thường dùng:
  - Đưa request vào **queue** (Kafka/RabbitMQ), xử lý tuần tự 1 luồng cho riêng sản phẩm đó → tránh 10.000 connection cùng chờ lock DB.
  - Hoặc dùng **Redis DECR** (atomic counter) làm lớp chặn trước, chỉ cho tối đa N request "đi qua" xuống DB, số còn lại fail nhanh ngay tại Redis, giảm tải DB.
  - **Optimistic lock** (cột `version`, `UPDATE ... WHERE id=? AND version=?`) vẫn đúng về mặt kết quả, nhưng với 10.000 request tranh 1 dòng thì tỷ lệ conflict cực cao → rất nhiều lượt phải retry → tệ hơn atomic update trực tiếp về hiệu năng. Optimistic lock hợp lý hơn khi tỷ lệ tranh chấp thấp.

**Trường hợp có 2 sản phẩm khác nhau:**
- Nếu mỗi request chỉ mua 1 trong 2 sản phẩm (2 row khác nhau) → DB lock độc lập theo từng row, không tranh nhau, xử lý song song bình thường, không có gì đặc biệt.
- Nếu 1 request cần mua **cả 2 sản phẩm cùng lúc** (ví dụ combo) → phải bọc 2 câu UPDATE trong **1 transaction**. Khi đó cần lưu ý **deadlock**: nếu request A update sản phẩm 1 trước rồi tới sản phẩm 2, còn request B update sản phẩm 2 trước rồi tới sản phẩm 1 → 2 transaction có thể lock chéo nhau và deadlock. Cách tránh: luôn lock/update các row theo **thứ tự cố định** (ví dụ sort theo `product_id` tăng dần trước khi update) để mọi transaction đi theo cùng 1 chiều.

### 3
Đặt vé xem phim, 1 ghế có nhiều request đặt cùng lúc thì xử lý như nào để không bị bán 2 lần

**Trả lời:**

Về bản chất giống bài toán tồn kho, chỉ khác là trạng thái ghế là boolean/enum (`available` / `booked`) thay vì số lượng.

**Cách 1 – Atomic update có điều kiện (giống câu 2):**
```sql
UPDATE seats
SET status = 'booked', user_id = ?
WHERE id = ? AND status = 'available';
```
`affected_rows = 1` → đặt thành công; `= 0` → ghế đã bị người khác đặt trước, báo lỗi.

**Cách 2 – Unique constraint:** tạo bảng `bookings` với `UNIQUE(seat_id, showtime_id)` cho các booking còn hiệu lực. Insert booking record; nếu vi phạm unique thì báo ghế đã được đặt. Cách này để DB dùng unique index như một dạng "lock" tự nhiên.

**Cách 3 – Pessimistic lock:** `SELECT * FROM seats WHERE id=? FOR UPDATE` trong transaction trước khi update, transaction khác chạm vào cùng ghế sẽ phải đợi transaction hiện tại commit/rollback.

**Thực tế production** thường thêm cơ chế **giữ ghế tạm (hold/reservation) có TTL**: khi user chọn ghế, `SET NX EX 300 seat:{id} user_id` trên Redis (atomic, tự hết hạn sau 5 phút nếu không thanh toán) để tránh giữ ghế vô thời hạn mà không chốt đơn, sau đó khi thanh toán xong mới update chính thức xuống DB bằng 1 trong 2 cách trên.

### 4
Có logic phải chạy trong 1 transaction là trừ tồn kho và tạo đơn hàng. nếu lỗi thì transaction rollback.
Có 2 request mua cùng lúc thì xử lý, có thể xảy ra trường hợp cả 2 đều ko mua đc k ?
trong trường hợp user 1 đến trước nhưng tạo order bị lỗi, user 2 đến chậm hơn thì ko mua đc.

**Trả lời:**

**Có 2 request cùng lúc, cả 2 đều không mua được có xảy ra không? → Có, trong một số tình huống:**
- **Deadlock:** nếu request A lock tồn kho trước rồi lock bảng order, còn request B lock bảng order trước rồi lock tồn kho → DB phát hiện deadlock và **abort ít nhất 1 transaction** (tùy DB có thể abort cả 2 nếu retry logic sai). Cách phòng tránh: luôn thao tác theo thứ tự cố định (trừ kho trước → tạo order sau, áp dụng nhất quán cho toàn bộ code, tránh 1 luồng khác làm ngược lại).
- **Lỗi hệ thống chung** (hết connection pool, DB timeout, deadlock retry sai) khiến cả 2 transaction cùng fail vì nguyên nhân độc lập với logic nghiệp vụ, không liên quan gì đến việc tranh tồn kho.
- Nếu lỗi chỉ do logic nghiệp vụ (VD constraint vi phạm ở order) và không liên quan lock, thì 2 transaction độc lập, không lý do gì khiến cả 2 cùng fail vì cùng 1 nguyên nhân đó.

**Case: user1 đến trước nhưng tạo order lỗi (rollback), user2 đến sau lại không mua được:**
- Nguyên nhân gốc thường là: phần trừ tồn kho và tạo order **không thực sự nằm trong đúng 1 transaction DB**, mà bị tách rời (ví dụ trừ kho ở 1 transaction/service riêng, tạo order ở transaction khác, rồi "rollback" bằng cách gọi tay 1 API cộng lại tồn kho). Khi đó:
  - User1 trừ kho xong (commit ngay) → tồn kho giảm.
  - Tạo order lỗi → hệ thống phải gọi thêm 1 bước bù trừ (cộng lại kho) — nếu bước này chạy **chậm hoặc thất bại**, trong khoảng thời gian đó tồn kho vẫn đang ở trạng thái "đã bị trừ" một cách sai lệch.
  - User2 đến trong đúng khoảng hở đó, thấy tồn kho = 0 → bị báo hết hàng, dù thực chất hàng vẫn còn (chỉ là chưa được hoàn lại kịp).
- **Cách xử lý đúng:** trừ tồn kho và tạo order phải nằm trong **cùng 1 transaction DB thật** (cùng connection, `BEGIN ... COMMIT/ROLLBACK`). Khi tạo order lỗi, DB tự `ROLLBACK` toàn bộ (bao gồm cả câu trừ kho) trong **một phát duy nhất** và giải phóng lock ngay lập tức. Lúc đó user2 tới sau sẽ thấy tồn kho đúng như ban đầu (chưa hề bị trừ) và mua được bình thường — không có "khoảng hở" nào giữa trừ kho và rollback.
- Nếu hệ thống là **microservices** (tồn kho và order ở 2 service/DB khác nhau, không share 1 transaction) thì không thể dùng transaction DB thông thường, phải dùng **Saga pattern**: trừ kho trước → nếu tạo order lỗi thì gọi **compensating transaction** để hoàn kho, và phải đảm bảo bước hoàn kho có tính idempotent + retry (ví dụ qua message queue với at-least-once delivery) để không bị "treo" ở trạng thái sai lệch như trên.

### 5
Tiếp câu 4, trừ số lượng tồn kho và tạo order là trong 1 transaction, nếu user 1 mua, trừ kho ok, tạo order ok, transaction commit, nhưng user 2 mua, transaction chạy, lúc đọc tồn kho vẫn còn do transaction của user 1 chưa commit thì sao, cơ chế gì đảm bảo việc user 2 mua phải fail ?

**Trả lời:**

**Cốt lõi:** `UPDATE` luôn xin **row-level exclusive (write) lock**, bất kể isolation level nào, và điều kiện trong `WHERE` được **re-check lại tại thời điểm lock được cấp** (dựa trên dữ liệu mới nhất/committed) — chứ không dựa vào giá trị mà transaction "nhìn thấy" lúc mới bắt đầu. Đây gọi là cơ chế **"first updater wins"**.

Diễn giải theo dòng thời gian, với cùng câu lệnh atomic update ở câu 2:
```sql
UPDATE products SET stock = stock - 1 WHERE id = ? AND stock >= 1;
```
1. User1 chạy UPDATE này trong transaction → DB cấp write lock trên row đó ngay lập tức (dù **chưa commit**).
2. User2 chạy đúng câu UPDATE này trên cùng row → vì row đang bị user1 khóa, UPDATE của user2 **bị block, phải chờ** — bất kể trước đó user2 có `SELECT stock` ra thấy vẫn còn 1 hay không, và bất kể isolation level là gì (chờ ở đây là do write lock, không liên quan tới việc "đọc thấy giá trị nào").
3. User1 `COMMIT` → lock được giải phóng, `stock = 0` chính thức có hiệu lực.
4. UPDATE của user2 được "đánh thức", nhưng DB **re-evaluate lại `WHERE stock >= 1` với giá trị mới nhất (đã commit = 0)**, chứ không áp theo giá trị cũ lúc user2 mới đọc → điều kiện sai → `affected_rows = 0` → user2 fail đúng như mong đợi.
   - Ngược lại, nếu user1 `ROLLBACK` thay vì commit → lock giải phóng, `stock` trở về giá trị gốc (=1) → điều kiện của user2 lại đúng → user2 update thành công bình thường. Đây chính là lời giải cho khoảng hở nêu ở câu 4.

**Điểm mấu chốt:** cơ chế trên chỉ đúng vì dùng **atomic `UPDATE ... WHERE`** — DB tự gộp "đọc mới nhất + ghi" làm một dưới lock. Nếu code làm theo kiểu đọc trước rồi tính toán ở app:
```sql
SELECT stock FROM products WHERE id = ?;      -- app đọc ra 1, KHÔNG xin lock
-- app kiểm tra: 1 >= 1 → OK
UPDATE products SET stock = 0 WHERE id = ?;   -- app tự gán giá trị mới, không có điều kiện so sánh
```
thì `SELECT` thường (không `FOR UPDATE`) không xin lock nào → user1 và user2 có thể cùng đọc snapshot `stock = 1` gần như đồng thời (đặc biệt do MVCC ở READ COMMITTED/REPEATABLE READ, mỗi transaction có snapshot riêng) → cả hai đều nghĩ mình mua được → **lost update**, tồn kho sai (âm hoặc bán trùng) — quay lại đúng vấn đề đã nói ở câu 1/2.

Nếu vẫn muốn tách bước đọc và ghi (ví dụ cần đọc thêm dữ liệu khác để tính toán phức tạp), có 2 cách giữ an toàn:
- **Pessimistic:** `SELECT ... FOR UPDATE` để chủ động xin write lock ngay từ bước đọc — transaction sau sẽ bị block giống hệt case UPDATE ở trên.
- **Optimistic:** đọc kèm cột `version`, rồi `UPDATE ... WHERE id=? AND version=?` — nếu `version` đã đổi (do transaction khác commit trước) thì `affected_rows=0`, app tự phát hiện conflict và retry/báo lỗi.

**Khác biệt giữa các DB đáng lưu ý:**
- **MySQL/InnoDB:** hành vi "chờ lock rồi re-check theo latest committed data" (semi-consistent read) áp dụng từ READ COMMITTED trở lên — im lặng trả `affected_rows=0`, không có lỗi nào ném ra.
- **PostgreSQL:** ở READ COMMITTED cũng re-evaluate WHERE tương tự (cơ chế nội bộ gọi là EvalPlanQual). Nhưng nếu set isolation `REPEATABLE READ`/`SERIALIZABLE`, thay vì âm thầm re-check, Postgres sẽ **ném lỗi `could not serialize access due to concurrent update`**, buộc application phải tự bắt lỗi và retry cả transaction — tức chặt hơn, đẩy trách nhiệm xử lý conflict lên tầng app thay vì tự "nuốt" như READ COMMITTED.

**Follow-up:** Nếu có 1000 request mua thì sự tranh chấp lớn, nhiều request sẽ retry lại thì ko ổn, nếu có tranh chấp nhiều thì ko nên dùng Optimistic phải k ?

**Trả lời: Đúng, trực giác đó chính xác.** Optimistic lock được thiết kế cho kịch bản **ít va chạm** (conflict là ngoại lệ hiếm), không phải cho hotspot nghìn request/1 row.

**Vì sao Optimistic tệ khi tranh chấp cao:**
- Optimistic **không chặn ai cả** ở bước đọc — mọi request đều được phép đọc `version` hiện tại và thử `UPDATE` song song. Với 1000 request cùng nhắm 1 row:
  - Chỉ **đúng 1** request có `UPDATE ... WHERE version=?` khớp → thành công.
  - **999 request còn lại** đều fail (`affected_rows=0`) → phải retry: đọc lại `version` mới, thử update lại. Nhưng vì vẫn còn hàng trăm request khác đang retry cùng lúc, tại mỗi vòng chỉ có 1 request thắng, số còn lại tiếp tục fail → retry storm, có request phải thử đi thử lại rất nhiều lần trước khi đến lượt (hoặc hết hàng thì retry mãi vẫn fail, tốn tài nguyên vô ích).
  - Mỗi lần thử là 1 round-trip DB thật sự (đọc + update) → tổng số query xuống DB tăng vọt so với số request gốc → **lãng phí tài nguyên** hơn nhiều so với việc chỉ đơn giản để DB tự xếp hàng.
  - Cần thêm logic retry ở application (thường kèm backoff/jitter để tránh tất cả retry cùng lúc lần nữa) → phức tạp hơn, và độ trễ cho request "xui" (phải retry nhiều lần) có thể rất cao, trải nghiệm không đều giữa các user.

**So sánh với 2 cách kia trong cùng kịch bản 1000 request/1 row:**
- **Atomic `UPDATE ... WHERE stock >= 1`** (không dùng version): DB tự dùng row-lock để **serialize** — mỗi request lần lượt được xử lý theo hàng đợi lock của DB, **không request nào phải tự retry ở tầng app**, mỗi request chỉ tốn đúng 1 query. Đây là lựa chọn hợp lý nhất cho trường hợp tranh chấp cao vì không có công sức bị lãng phí do thử lại.
- **Pessimistic `SELECT ... FOR UPDATE`**: về bản chất cũng serialize giống atomic update (transaction sau phải đợi lock được giải phóng), nhưng tốn thêm 1 round-trip (SELECT rồi mới UPDATE) và giữ transaction mở lâu hơn (từ lúc SELECT tới lúc COMMIT) → nghẽn connection/transaction lâu hơn atomic update, nhưng vẫn tốt hơn optimistic vì không phải retry.

**Quy tắc chọn lựa:**
| Mức độ tranh chấp | Nên dùng |
|---|---|
| Thấp (hiếm conflict, ví dụ update profile, sửa đơn hàng của riêng mình) | Optimistic lock — không tốn chi phí lock khi không có conflict, chỉ phạt đúng lúc va chạm thật (hiếm) |
| Cao / hotspot (flash-sale, 1 ghế/1 sản phẩm cuối bị nghìn request tranh) | Atomic `UPDATE ... WHERE` (ưu tiên) hoặc Pessimistic lock — để DB tự serialize, tránh retry storm |

Và như đã nêu ở câu 2: với hotspot cực đoan (10.000 request/1 sản phẩm), dù chọn atomic update hay pessimistic, vẫn nên thêm lớp **chặn sớm trước khi chạm DB** (Redis `DECR` làm bộ đếm nhanh, hoặc queue) để giảm số request thực sự phải xếp hàng ở DB — bản thân việc DB serialize đúng không có nghĩa là hiệu năng tốt khi số lượng request quá lớn.

### 6
Chuyển khoản giữa 2 tài khoản ngân hàng: trừ tiền tài khoản A, cộng tiền tài khoản B, phải cùng thành công hoặc cùng thất bại, và số dư không bao giờ được âm.
Nếu user bấm nút "Chuyển khoản" 2 lần liên tiếp (double click / mất mạng client tự retry) thì xử lý sao để không bị trừ tiền 2 lần ?

**Trả lời:**

Bài toán này kết hợp 3 kỹ thuật đã học: **Transaction** (atomic 2 update), **Atomic update có điều kiện** (chặn âm số dư), và **Idempotency key + Unique constraint** (chặn double-click).

**1. Đảm bảo trừ A / cộng B cùng thành công hoặc cùng thất bại, và A không âm:**
```sql
BEGIN;
UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?;  -- trừ A, có điều kiện
-- affected_rows = 0 → không đủ tiền → ROLLBACK, báo lỗi
UPDATE accounts SET balance = balance + ? WHERE id = ?;                  -- cộng B
COMMIT;
```
Điều kiện `balance >= ?` ngay trong UPDATE trừ A đảm bảo không bao giờ âm — giống hệt logic tồn kho ở câu 2. Cả 2 update nằm cùng 1 transaction nên nếu bất kỳ bước nào lỗi, DB tự rollback toàn bộ, không có trạng thái "trừ A nhưng chưa cộng B".

**2. Deadlock khi nhiều giao dịch chuyển qua lại giữa các cặp tài khoản** (A→B và B→A cùng lúc, hoặc nhiều cặp đan xen): áp dụng đúng nguyên tắc ở câu 2 — luôn lock/update theo **thứ tự cố định** (ví dụ luôn update tài khoản có `id` nhỏ hơn trước), không phụ thuộc thứ tự "người gửi trước, người nhận sau".

**3. Chống double-click / client tự retry (Idempotency key):**
- Client sinh 1 `idempotency_key` (UUID) duy nhất cho mỗi lần bấm "Chuyển khoản", gửi kèm request.
- Bảng `transfers` có `UNIQUE(idempotency_key)`. Trong cùng transaction ở bước 1, `INSERT INTO transfers (idempotency_key, from_account, to_account, amount, status) VALUES (...)` **trước khi** thực hiện 2 câu UPDATE.
- Nếu insert vi phạm unique (key đã tồn tại — tức đã xử lý trước đó) → không chạy lại 2 UPDATE, chỉ trả về kết quả của giao dịch cũ (tra theo `idempotency_key`).
- Nhờ nằm chung transaction, nếu phần UPDATE phía sau lỗi thì cả record `transfers` vừa insert cũng bị rollback theo, không để lại rác "giao dịch treo".

### 7
Mã giảm giá (voucher) chỉ cho phép tổng cộng 100 người dùng đầu tiên sử dụng, và mỗi user chỉ được dùng 1 lần. Có hàng nghìn request bấm "Áp dụng mã" cùng lúc thì thiết kế sao để không bao giờ vượt quá 100 lượt, và không user nào dùng được 2 lần ?

**Trả lời:**

Đây là bài toán kết hợp **Atomic counter có điều kiện** (giới hạn tổng 100 lượt, giống bài tồn kho) và **Unique constraint** (chặn 1 user dùng nhiều lần, giống bài email/ghế).

**Thiết kế bảng:**
- `vouchers(id, code, max_uses=100, used_count=0)`
- `voucher_usages(voucher_id, user_id, ...)` với `UNIQUE(voucher_id, user_id)`

**Flow xử lý 1 request áp dụng mã** — bắt buộc cả 2 bước nằm trong **cùng 1 transaction**:
```sql
BEGIN;
UPDATE vouchers SET used_count = used_count + 1 WHERE id = ? AND used_count < max_uses;
-- affected_rows = 0 → hết lượt → ROLLBACK, báo "voucher đã hết lượt dùng"
INSERT INTO voucher_usages (voucher_id, user_id) VALUES (?, ?);
-- vi phạm UNIQUE(voucher_id, user_id) → ROLLBACK (used_count tự lùi lại), báo "bạn đã dùng mã này rồi"
COMMIT;
```
- Bước `UPDATE ... WHERE used_count < max_uses` hoạt động y hệt bài tồn kho: 100.000 request tranh nhau, DB tự serialize qua row lock, chỉ đúng 100 request đầu tiên tăng được `used_count` thành công, số còn lại `affected_rows = 0` ngay lập tức, không cần biết `INSERT` ở bước sau.
- Bước `INSERT` với UNIQUE chặn trường hợp cùng 1 user gửi nhiều request đồng thời cố dùng mã nhiều lần — nếu insert fail do unique violation, **transaction rollback cả bước tăng counter**, nên không bị "mất oan" 1 trong 100 suất cho request bị từ chối.
- Vì cả 2 thao tác chung 1 transaction, thứ tự làm trước/sau không quan trọng về mặt đúng đắn (dù nên làm counter trước để fail nhanh, đỡ tốn 1 lần ghi insert khi đã chắc chắn hết lượt).

**Hiệu năng khi tranh chấp cao:** giống hệt gợi ý ở câu 2 — nếu traffic quá lớn (chục nghìn request cùng lúc vào đúng 1 row `vouchers`), nên thêm lớp chặn sớm bằng Redis (`INCR` counter + kiểm tra `SADD` cho `user_id` đã dùng) trước khi chạm DB, giảm tải xuống DB.

### 8
Hệ thống đếm lượt "like"/"view" cho 1 bài viết có traffic cực lớn (hàng chục nghìn request/giây). Có cần áp dụng lock chặt chẽ như bài toán tồn kho không ? Nếu chấp nhận số liệu không cần chính xác tuyệt đối tức thời thì nên xử lý như thế nào để không làm nghẽn DB ?

**Trả lời:**

**Không cần** — đây là điểm khác biệt quan trọng so với các bài toán trước. Tồn kho/ghế/voucher có ràng buộc nghiệp vụ **cứng** (không được âm, không được bán trùng), sai 1 đơn vị là lỗi nghiêm trọng (bán vượt tồn kho thật). Còn view/like **không có ràng buộc đúng-sai tuyệt đối** — đếm thiếu vài chục/vài trăm view trong vài giây hoàn toàn chấp nhận được. Bài toán này ưu tiên **throughput** hơn **strong consistency** → cố áp atomic update/lock trên từng request là lãng phí và gây nghẽn không cần thiết.

**Cách xử lý phổ biến — gộp batch trước khi ghi DB:**
1. **Đếm ở tầng nhanh trước:** mỗi lượt view/like chỉ `INCR` vào Redis (đơn luồng nên atomic tự nhiên, không cần lock gì thêm), cực nhanh, chịu được traffic rất lớn.
2. **Flush định kỳ xuống DB:** 1 job (cron/worker) cứ mỗi vài giây (hoặc mỗi N lượt tích lũy) đọc giá trị counter trong Redis rồi ghi 1 lần duy nhất xuống DB: `UPDATE posts SET view_count = view_count + N WHERE id = ?`. Hàng chục nghìn request/giây chỉ tạo ra vài chục write DB/giây thay vì hàng chục nghìn.
3. **Hoặc dùng queue:** đẩy event "1 view" vào Kafka, 1 consumer gom (aggregate) theo cửa sổ thời gian rồi batch update — phù hợp khi cần audit lại từng lượt view chi tiết.
4. Số hiển thị cho user chấp nhận trễ vài giây so với thực tế (eventual consistency) — đánh đổi hợp lý vì không ảnh hưởng nghiệp vụ.

**Nếu cần chống spam** (1 user bấm like nhiều lần cho cùng 1 bài): vẫn dùng `UNIQUE(user_id, post_id)` ở bảng `likes` để đảm bảo tính đúng logic (1 user chỉ tính 1 like), việc này tách biệt với chuyện "đếm nhanh" — bảng likes ghi đúng ngay (ít traffic hơn nhiều vì giới hạn bởi số user thật), còn con số hiển thị tổng có thể lấy từ counter batch ở trên.

=> Nhưng redis làm sao biết đc user like nhiều lần cho 1 bài ?

**Trả lời:** Đúng, câu hỏi rất hợp lý — nếu chỉ `INCR` mù (không biết ai đã like) thì không thể chống trùng được. Chỗ này cần tách rõ 2 việc khác nhau, và **cả 2 đều nên xử lý ở Redis** (không phải Redis chỉ để đếm, DB chỉ để check trùng như mô tả hơi đơn giản ở câu trả lời trước):

**Dùng Redis Set thay vì chỉ counter:**
```
SADD liked_users:{postId} {userId}
```
- `SADD` cũng **atomic** giống `SETNX` (Redis đơn luồng) và có tính **idempotent tự nhiên**: nếu `userId` đã có trong set → trả về `0` (không thêm gì cả); nếu chưa có → thêm vào và trả về `1`.
- Nhờ vậy, dù 1 user bấm like 10 lần liên tục hoặc gửi request trùng, `SADD` chỉ trả `1` đúng **1 lần duy nhất** (lần đầu tiên), các lần sau luôn trả `0` — tự nó đã là cơ chế chống trùng, không cần hỏi DB.

**Flow đầy đủ khi có request "like":**
1. `SADD liked_users:{postId} {userId}`
   - Trả `0` → user đã like rồi → bỏ qua, không làm gì thêm (idempotent, không báo lỗi, giống bấm like lần 2 chỉ là no-op).
   - Trả `1` → lần like đầu tiên thật sự → đi tiếp bước 2.
2. `INCR like_count:{postId}` — chỉ tăng khi bước 1 trả `1`, nên số đếm luôn khớp với số user thực sự đã like (không bị đếm trùng).
3. Định kỳ (batch job) đọc `like_count:{postId}` để flush tổng xuống cột `posts.like_count` trong DB — giống flow đã nêu ở câu trả lời gốc.

**Vậy dữ liệu "ai đã like" lưu ở đâu lâu dài?** Redis Set (`liked_users:{postId}`) chỉ nên là bản **nhanh/gần-thời-gian-thực**, không nên là nơi lưu trữ duy nhất — nếu Redis mất dữ liệu (restart không bật persistence, bị evict do hết bộ nhớ) thì mất luôn lịch sử ai đã like. Cách xử lý:
- Bật Redis **persistence** (RDB/AOF) nếu muốn Redis giữ vai trò gần như nguồn chính.
- Hoặc mỗi khi `SADD` trả `1` (like thật sự đầu tiên), đẩy 1 event nhẹ vào queue (Kafka/Redis Stream) để 1 worker ghi bất đồng bộ xuống bảng `likes(user_id, post_id)` có `UNIQUE(user_id, post_id)` trong DB — vẫn tránh được việc ghi DB đồng bộ theo từng request (giữ throughput cao), nhưng có bản lưu trữ bền vững, và UNIQUE constraint ở DB vẫn đóng vai trò lớp bảo vệ cuối cùng đúng như nguyên tắc xuyên suốt các câu trên — Redis chỉ là lớp nhanh/chặn sớm, DB luôn là nguồn sự thật cuối cùng.

**Tóm lại:** Redis không "tự nhiên" biết ai đã like — nó biết được là nhờ dùng đúng cấu trúc dữ liệu (Set + `SADD`) để tra cứu/chống trùng trong memory, chứ không phải chỉ dùng phép `INCR` số nguyên đơn thuần như câu trả lời gốc mô tả hơi lược giản.

### 9
Payment gateway (ví dụ VNPay, Stripe) gọi webhook báo "thanh toán thành công" về hệ thống để cộng tiền ví / xác nhận đơn hàng. Do đặc thù mạng, gateway có thể gửi trùng webhook đó nhiều lần (at-least-once delivery). Làm sao đảm bảo hệ thống không cộng tiền / xác nhận đơn 2 lần dù nhận webhook trùng ?

**Trả lời:**

Về bản chất giống bài toán **idempotency key** đã gặp ở câu 6, chỉ khác nguồn phát trigger trùng lặp là hệ thống ngoài (payment gateway) thay vì client của chính mình.

**Thiết kế:**
1. Payment gateway luôn gửi kèm 1 mã giao dịch duy nhất (`gateway_transaction_id` — ví dụ `vnp_TxnRef`, Stripe `event.id`).
2. Bảng `processed_webhooks` (hoặc field riêng trong bảng `payments`) có `UNIQUE(gateway_transaction_id)`.
3. Khi nhận webhook, trong **1 transaction**:
```sql
BEGIN;
INSERT INTO processed_webhooks (gateway_transaction_id, ...) VALUES (?, ...);
-- vi phạm UNIQUE → đã xử lý trước đó → ROLLBACK, trả HTTP 200 ngay (không làm gì thêm)
UPDATE wallets SET balance = balance + ? WHERE user_id = ?;   -- hoặc UPDATE orders SET status='paid'...
COMMIT;
```
- Insert thành công (lần đầu) → mới thực hiện cộng tiền/xác nhận đơn trong cùng transaction đó → nếu bước cộng tiền lỗi, rollback luôn cả insert, để lần webhook retry tiếp theo từ gateway vẫn được xử lý đúng (không bị "khóa" nhầm bởi record insert dở dang).
4. **Luôn trả HTTP 200 nhanh** ngay khi đã ghi nhận đủ (kể cả với webhook trùng) để gateway ngừng retry; nếu phần xử lý nghiệp vụ nặng, nên tách async sau khi transaction ghi nhận idempotency đã commit.
5. **Bảo mật:** luôn verify chữ ký (signature) của webhook (HMAC do gateway cung cấp) trước khi xử lý, để đảm bảo request thực sự đến từ gateway — tránh bị giả mạo webhook để cộng tiền khống. Đây là lớp bảo vệ khác, độc lập với idempotency.

### 10
Flash sale: mỗi tài khoản chỉ được mua tối đa 1 sản phẩm trong đợt sale. User có thể mở nhiều tab / gọi API nhiều lần gần như đồng thời để cố mua nhiều hơn 1. Thiết kế sao để chặn được việc này dù request đến đồng thời ?

**Trả lời:**

Về bản chất là race condition giữa nhiều request của **cùng 1 user** (khác câu 2 là giữa nhiều user khác nhau) → cần **Unique constraint** theo `(user_id, sale_id)`, kết hợp với **atomic update tồn kho** đã có ở câu 2/4.

**Thiết kế:**
- Bảng `flash_sale_limits` (hoặc chính bảng `orders` lọc theo sale): `UNIQUE(user_id, sale_id)`.

**Flow xử lý, trong 1 transaction:**
```sql
BEGIN;
INSERT INTO flash_sale_limits (user_id, sale_id) VALUES (?, ?);
-- vi phạm UNIQUE → user đã mua trong đợt sale này rồi → ROLLBACK, báo lỗi
UPDATE products SET stock = stock - 1 WHERE id = ? AND stock >= 1;  -- giống câu 2
-- affected_rows = 0 → hết hàng → ROLLBACK
INSERT INTO orders (...) VALUES (...);
COMMIT;
```
- Dù user mở 2 tab, bắn 2 request gần như đồng thời, DB chỉ cho **đúng 1** `INSERT INTO flash_sale_limits` thành công nhờ UNIQUE constraint — request còn lại bị lỗi ngay tại bước insert (không phụ thuộc timing, DB tự đảm bảo dù 2 request đến sát nhau tới đâu).
- **Sai lầm cần tránh:** không được check kiểu "SELECT xem user đã mua chưa → nếu chưa thì mới INSERT" (check-then-act) — đây lại đúng lỗi đã nói ở câu 1, 2 tab vẫn có thể cùng SELECT ra "chưa mua" trước khi tab nào insert xong. Phải để **UNIQUE constraint ở DB** làm lớp chặn cuối cùng, tuyệt đối, không dựa vào check ở application.
- **Optimization:** có thể thêm chặn sớm ở Redis (`SETNX flash_sale_lock:{userId}:{saleId}`) để phản hồi nhanh cho các request dư thừa từ multi-tab mà không cần chạm DB — nhưng vẫn giữ UNIQUE constraint DB làm lớp bảo vệ cuối cùng, đúng nguyên tắc đã nêu ở câu 1.

----
Các câu hỏi này này dùng gì đến các kĩ thuật: unique, atomic update, transaction, khóa lạc quan, khóa bị quan 
Nếu đc hãy thêm các case khác nữa trong 4 câu hỏi

---

## Tổng kết: câu hỏi nào dùng kỹ thuật gì

| Kỹ thuật | Câu áp dụng | Ý nghĩa |
|---|---|---|
| **Unique constraint** | Câu 1 (email), Câu 3 – cách 2 (unique seat_id+showtime_id), Câu 7 (user dùng voucher), Câu 9 (idempotency webhook), Câu 10 (user + sale) | DB tự đảm bảo tính duy nhất tại tầng lưu trữ, atomic, không cần lock tay |
| **Atomic update** (`UPDATE ... WHERE điều kiện`, check affected rows) | Câu 2 (stock >= 1), Câu 3 – cách 1 (status='available'), Câu 6 (balance >= amount), Câu 7 (used_count < max_uses) | Gộp "kiểm tra + ghi" thành 1 bước duy nhất, tận dụng row lock của DB |
| **Transaction (ACID + rollback)** | Câu 4, Câu 6, Câu 7, Câu 9, Câu 10, ngầm định trong Câu 2/3 khi ghép với tạo order | Đảm bảo nhiều thay đổi liên quan (trừ kho, tạo order, cộng/trừ ví...) cùng thành công hoặc cùng thất bại, không để dữ liệu ở trạng thái nửa vời |
| **Khóa bi quan (Pessimistic lock – `SELECT ... FOR UPDATE`)** | Câu 2 (khi cần xử lý phức tạp hơn 1 câu UPDATE), Câu 3 – cách 3, Câu 4 | Transaction khác phải **chờ** lock được giải phóng mới được đọc/ghi; an toàn tuyệt đối nhưng dễ nghẽn khi tranh chấp cao |
| **Khóa lạc quan (Optimistic lock – cột `version`)** | Câu 2, Câu 3 (khi tỷ lệ tranh chấp thấp) | Không lock ngay, chỉ kiểm tra `version` không đổi lúc update; nếu conflict thì phải retry ở tầng app. Hợp khi ít va chạm, tệ khi tranh chấp cực cao (như 10.000 request/1 ghế) |
| **Idempotency key** | Câu 6 (double-click chuyển khoản), Câu 9 (webhook trùng) | Chặn xử lý trùng khi request/event bị gửi lặp lại từ cùng 1 nguồn (khác với race condition giữa nhiều user) |
| **Batch/async counter (bỏ strong consistency)** | Câu 8 (like/view) | Khi nghiệp vụ chấp nhận sai lệch nhỏ, đổi lấy throughput cao thay vì lock/transaction chặt trên từng request |

## Một số case bổ sung nên biết

1. **Idempotency key**: user bấm "Mua" nhiều lần (double click, mất mạng nên client tự retry) có thể gửi trùng request logic → cần idempotency key (client tạo 1 UUID cho mỗi lần "đặt hàng", server lưu lại, nếu thấy key đã xử lý rồi thì trả kết quả cũ, không tạo order thứ 2) — khác với race condition giữa 2 user khác nhau, đây là trùng request của **cùng 1 user**. Xem thêm Câu 6, Câu 9.
2. **Distributed lock (Redis/Zookeeper)**: khi hệ thống chạy nhiều instance và có thêm 1 lớp cache/logic nghiệp vụ nằm ngoài DB (ví dụ giữ ghế tạm ở câu 3), lock ở DB không đủ, cần lock phân tán ở tầng đó.
3. **Deadlock avoidance bằng thứ tự lock cố định**: bất cứ khi nào 1 transaction cần lock nhiều hơn 1 row/table, luôn quy ước lock theo 1 thứ tự nhất quán (VD theo id tăng dần) trong toàn bộ codebase. Xem thêm Câu 2, Câu 6.
4. **Isolation level**: chọn `READ COMMITTED` / `REPEATABLE READ` / `SERIALIZABLE` ảnh hưởng trực tiếp tới việc có bị **lost update** hay **phantom read** không — atomic update (`UPDATE ... WHERE`) vẫn an toàn ở `READ COMMITTED` vì lock được áp ngay tại thời điểm ghi, không phụ thuộc mức isolation cao.
5. **Rate limit / hàng đợi (queue) cho hotspot**: trường hợp 10.000 request tranh 1 sản phẩm cuối thường là dấu hiệu flash-sale — nên chặn từ tầng gateway (rate limit, hàng đợi ảo "xếp hàng online") để giảm tải thật xuống DB, thay vì để toàn bộ 10.000 request chạm DB rồi mới lọc.
6. **Idempotent compensating action trong Saga**: nếu dùng kiến trúc microservices như ở câu 4, hàm hoàn kho (compensating transaction) phải là idempotent (gọi lại nhiều lần không bị hoàn 2 lần), vì message queue có thể deliver trùng.
7. **Khi nào bỏ strong consistency**: không phải bài toán nào cũng cần atomic/lock chặt — nếu nghiệp vụ chấp nhận sai lệch nhỏ tạm thời (như đếm view/like ở Câu 8), nên ưu tiên batch/async để đổi lấy throughput, tránh lock hoá những chỗ không cần thiết.

### 1
Đăng ký tài khoản, làm sao để 2 tài khoản ko đc có email không trùng nhau, 2 request đến cùng lúc thì xử lý như nào ?

### 2
Trong bảng sản phẩm, một sản phẩm có cột tồn kho, làm sao để không bao giờ để tồn kho của sản phẩm đó xuống dưới 0.
Trong trường hợp tồn kho chỉ có 1 sản phẩm cuối mà có 10000 request mua hàng đến thì xử lý như nào ?
Và trong trường hợp có 2 sản phẩm thì như nào ?

### 3
Đặt vé xem phim, 1 ghế có nhiều request đặt cùng lúc thì xử lý như nào để không bị bán 2 lần

### 4
Có logic phải chạy trong 1 transaction là trừ tồn kho và tạo đơn hàng. nếu lỗi thì transaction rollback.
Có 2 request mua cùng lúc thì xử lý, có thể xảy ra trường hợp cả 2 đều ko mua đc k ?
trong trường hợp user 1 đến trước nhưng tạo order bị lỗi, user 2 đến chậm hơn thì ko mua đc.

----
Các câu hỏi này này dùng gì đến các kĩ thuật: unique, atomic update, transaction, khóa lạc quan, khóa bị quan 
Nếu đc hãy thêm các case khác nữa trong 4 câu hỏi

---

## Trả lời

### 1. Đăng ký tài khoản – chống trùng email khi 2 request đến cùng lúc

**Sai lầm thường gặp:** check rồi mới insert (`SELECT ... WHERE email=?` → nếu không có thì `INSERT`). Đây là pattern **check-then-act**, không atomic: 2 request có thể cùng SELECT ra "chưa tồn tại" trước khi cái nào insert xong, kết quả là 2 email trùng nhau vẫn lọt qua.

**Cách xử lý đúng:**
- Tạo **UNIQUE INDEX/CONSTRAINT** trên cột `email` ở DB. Đây là lớp bảo vệ cuối cùng và bắt buộc phải có, vì DB đảm bảo tính atomic khi ghi (unique index check + insert nằm trong 1 operation ở tầng storage engine).
- Application chỉ cần `INSERT` thẳng, nếu DB trả lỗi *duplicate key / unique violation* thì catch lại và trả message "email đã tồn tại" cho user.
- Có thể thêm lớp chặn sớm ở tầng cache/application (ví dụ Redis `SETNX email_lock` hoặc distributed lock theo email) để giảm số request phải chạm DB, tăng tốc phản hồi — nhưng đây chỉ là **optimization**, không được thay thế cho unique constraint.

Mô tả flow lớp chặn sớm ở tầng cache/application (ví dụ Redis `SETNX email_lock` hoặc distributed lock theo email) ?Redis `SETNX email_lock` hoạt động như nào


### 2. Tồn kho sản phẩm không được xuống dưới 0

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

### 3. Đặt vé xem phim – 1 ghế bị nhiều request đặt cùng lúc

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

### 4. Trừ tồn kho + tạo order trong 1 transaction, rollback khi lỗi

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

---

## Tổng kết: câu hỏi nào dùng kỹ thuật gì

| Kỹ thuật | Câu áp dụng | Ý nghĩa |
|---|---|---|
| **Unique constraint** | Câu 1 (email), Câu 3 – cách 2 (unique seat_id+showtime_id) | DB tự đảm bảo tính duy nhất tại tầng lưu trữ, atomic, không cần lock tay |
| **Atomic update** (`UPDATE ... WHERE điều kiện`, check affected rows) | Câu 2 (stock >= 1), Câu 3 – cách 1 (status='available') | Gộp "kiểm tra + ghi" thành 1 bước duy nhất, tận dụng row lock của DB |
| **Transaction (ACID + rollback)** | Câu 4, ngầm định trong Câu 2/3 khi ghép với tạo order | Đảm bảo nhiều thay đổi liên quan (trừ kho, tạo order) cùng thành công hoặc cùng thất bại, không để dữ liệu ở trạng thái nửa vời |
| **Khóa bi quan (Pessimistic lock – `SELECT ... FOR UPDATE`)** | Câu 2 (khi cần xử lý phức tạp hơn 1 câu UPDATE), Câu 3 – cách 3, Câu 4 | Transaction khác phải **chờ** lock được giải phóng mới được đọc/ghi; an toàn tuyệt đối nhưng dễ nghẽn khi tranh chấp cao |
| **Khóa lạc quan (Optimistic lock – cột `version`)** | Câu 2, Câu 3 (khi tỷ lệ tranh chấp thấp) | Không lock ngay, chỉ kiểm tra `version` không đổi lúc update; nếu conflict thì phải retry ở tầng app. Hợp khi ít va chạm, tệ khi tranh chấp cực cao (như 10.000 request/1 ghế) |

## Một số case bổ sung nên biết

1. **Idempotency key**: user bấm "Mua" nhiều lần (double click, mất mạng nên client tự retry) có thể gửi trùng request logic → cần idempotency key (client tạo 1 UUID cho mỗi lần "đặt hàng", server lưu lại, nếu thấy key đã xử lý rồi thì trả kết quả cũ, không tạo order thứ 2) — khác với race condition giữa 2 user khác nhau, đây là trùng request của **cùng 1 user**.
2. **Distributed lock (Redis/Zookeeper)**: khi hệ thống chạy nhiều instance và có thêm 1 lớp cache/logic nghiệp vụ nằm ngoài DB (ví dụ giữ ghế tạm ở câu 3), lock ở DB không đủ, cần lock phân tán ở tầng đó.
3. **Deadlock avoidance bằng thứ tự lock cố định**: bất cứ khi nào 1 transaction cần lock nhiều hơn 1 row/table, luôn quy ước lock theo 1 thứ tự nhất quán (VD theo id tăng dần) trong toàn bộ codebase.
4. **Isolation level**: chọn `READ COMMITTED` / `REPEATABLE READ` / `SERIALIZABLE` ảnh hưởng trực tiếp tới việc có bị **lost update** hay **phantom read** không — atomic update (`UPDATE ... WHERE`) vẫn an toàn ở `READ COMMITTED` vì lock được áp ngay tại thời điểm ghi, không phụ thuộc mức isolation cao.
5. **Rate limit / hàng đợi (queue) cho hotspot**: trường hợp 10.000 request tranh 1 sản phẩm cuối thường là dấu hiệu flash-sale — nên chặn từ tầng gateway (rate limit, hàng đợi ảo "xếp hàng online") để giảm tải thật xuống DB, thay vì để toàn bộ 10.000 request chạm DB rồi mới lọc.
6. **Idempotent compensating action trong Saga**: nếu dùng kiến trúc microservices như ở câu 4, hàm hoàn kho (compensating transaction) phải là idempotent (gọi lại nhiều lần không bị hoàn 2 lần), vì message queue có thể deliver trùng.

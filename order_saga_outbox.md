### Bài toán: Microservices với luồng Đặt hàng → Trừ kho → Thanh toán

Giả sử hệ thống tách thành 3 service độc lập, **mỗi service 1 DB riêng** (database-per-service — đặc trưng microservices):
- **Order Service** (DB: `orders`)
- **Inventory Service** (DB: `inventory`)
- **Payment Service** (DB: `payments`)

Luồng nghiệp vụ: `Đặt hàng → Trừ kho → Thanh toán`, cần cùng thành công hoặc cùng thất bại (nếu thanh toán lỗi thì phải hoàn kho, huỷ đơn). Vì 3 bước nằm ở 3 DB khác nhau, **không thể dùng 1 transaction DB thông thường** như đã làm ở `question.md` (câu 4) khi mọi thứ còn chung 1 DB.

---

## 1. Vì sao không dùng 2PC (Two-Phase Commit)

2PC là cách "kinh điển" để có transaction xuyên nhiều DB (dùng XA transaction), nhưng gần như không ai dùng trong microservices thực tế vì:
- **Blocking:** tất cả participant phải giữ lock chờ coordinator quyết định commit hay rollback → nếu 1 service chậm/down giữa chừng, các service khác bị treo lock theo.
- **Coordinator là single point of failure** — coordinator chết giữa chừng, các participant không biết phải commit hay rollback.
- **Không phù hợp với message queue / NoSQL** — nhiều công nghệ phổ biến trong hệ microservices (Kafka, MongoDB, Redis...) không hỗ trợ XA transaction.
- **Không scale tốt** — giữ lock xuyên nhiều service trong thời gian dài đi ngược lại mục tiêu scale độc lập của microservices.

→ Microservices dùng **Saga pattern** thay thế: đánh đổi "atomic thật" lấy "eventually consistent" (cuối cùng dữ liệu sẽ đúng, dù có khoảng thời gian ngắn chưa nhất quán).

---

## 2. Saga Pattern là gì

**Ý tưởng cốt lõi:** thay vì 1 transaction lớn xuyên nhiều DB, chia thành **chuỗi các local transaction** (mỗi bước chỉ transaction trong đúng 1 DB của chính service đó — giống các kỹ thuật atomic update/transaction đã học). Nếu 1 bước ở giữa lỗi, saga sẽ chạy các **compensating transaction** (giao dịch bù trừ) để hoàn tác các bước trước đó theo thứ tự ngược lại.

Ví dụ cho luồng này:

| Bước | Local transaction                                                 | Compensating transaction (nếu cần hoàn tác)                                                     |
|------|-------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| 1    | Order Service: tạo order, status=`PENDING`                        | Order Service: set status=`CANCELLED`                                                           |
| 2    | Inventory Service: trừ kho (atomic `UPDATE ... WHERE stock >= ?`) | Inventory Service: cộng lại kho                                                                 |
| 3    | Payment Service: trừ tiền / charge thẻ                            | Payment Service: hoàn tiền (refund) — nếu đã charge thành công rồi mới phát hiện lỗi ở bước sau |

Có 2 cách triển khai Saga: **Choreography** và **Orchestration**.

### 2.1 Choreography (mỗi service tự lắng nghe event của nhau, không ai "chỉ huy")

```
Order Service --OrderCreated--> Inventory Service
Inventory Service --InventoryReserved--> Payment Service
Payment Service --PaymentCompleted--> Order Service (confirm)

                                  hoặc nếu lỗi:
Payment Service --PaymentFailed--> Inventory Service (compensate: hoàn kho)
Inventory Service --InventoryReleased--> Order Service (set CANCELLED)
```

- Mỗi service publish event khi xong việc của mình, và subscribe event của service trước để biết khi nào tới lượt mình xử lý.
- **Ưu điểm:** đơn giản khi số bước ít, không có điểm điều phối trung tâm, các service thực sự độc lập (loose coupling).
- **Nhược điểm:** khi luồng phức tạp hơn (nhiều bước, nhiều nhánh lỗi), rất khó nhìn ra "toàn cảnh" luồng nghiệp vụ chỉ bằng cách đọc code từng service riêng lẻ (logic bị rải rác) — khó debug, khó thêm bước mới mà không vô tình tạo vòng lặp phụ thuộc giữa các event.

### 2.2 Orchestration (có 1 "nhạc trưởng" điều phối)

```
                     ┌─────────────────────────┐
                     │   Order Saga Orchestrator │
                     └─────────────────────────┘
                        │            │           │
                 1.Trừ kho    2.Thanh toán   3.Confirm order
                        ▼            ▼           │
              Inventory Service  Payment Service  ▼
                                              Order Service
```

- Orchestrator (có thể là 1 module trong Order Service, hoặc 1 service riêng) **chủ động gọi tuần tự** từng service, chờ kết quả, rồi quyết định bước tiếp theo hoặc trigger compensate.
- **Ưu điểm:** luồng nghiệp vụ được viết tường minh ở 1 chỗ (thường dạng state machine), dễ đọc, dễ debug, dễ thêm bước mới.
- **Nhược điểm:** orchestrator trở thành nơi tập trung logic — nhưng khác 2PC coordinator, nó **không giữ lock chờ đồng bộ** xuyên suốt, chỉ điều phối tuần tự các lời gọi async/event, nên không có vấn đề blocking như 2PC. Cần đảm bảo orchestrator lưu lại trạng thái saga (xem mục 6) để có thể resume nếu chính nó bị restart giữa chừng.

**Khuyến nghị thực tế:** luồng càng nhiều bước / nhiều nhánh lỗi phức tạp → càng nên dùng **Orchestration**; luồng đơn giản 2-3 bước, ít thay đổi → **Choreography** đủ dùng và gọn hơn.

### 2.3 Event-Driven Architecture (EDA) và Saga liên quan thế nào

**Đừng nhầm 2 khái niệm này là một** — dễ gây hiểu sai:
- **Event-Driven Architecture (EDA)** là 1 **phong cách kiến trúc rộng**: các service giao tiếp chủ yếu qua publish/subscribe event thay vì gọi trực tiếp REST/RPC đồng bộ. Dùng cho rất nhiều mục đích, không chỉ transaction: đồng bộ cache, gửi notification, cập nhật read model (CQRS), analytics, audit log...
- **Saga** là **1 pattern cụ thể**, chỉ giải quyết đúng 1 vấn đề: duy trì tính nhất quán dữ liệu xuyên nhiều service khi không thể dùng 1 transaction chung (2PC). Saga có thể triển khai theo cách event-driven (Choreography) hoặc không hoàn toàn (Orchestration).

**EDA là 1 cơ chế giao tiếp mà Saga có thể mượn dùng — không phải bắt buộc:**
- **Choreography Saga = bắt buộc phải dùng EDA thuần** (toàn bộ giao tiếp qua publish/subscribe event, không ai gọi trực tiếp ai). Ở đây đúng là "Saga dùng EDA" 100%, tách rời 2 khái niệm này ra là không còn Choreography nữa.
- **Orchestration Saga thì không bắt buộc** — orchestrator hoàn toàn có thể gọi REST/RPC đồng bộ tới từng service (`POST /inventory/reserve`, chờ response luôn), **không cần message broker, không cần publish/subscribe gì cả**. Vẫn là Saga đúng nghĩa (vẫn có local transaction + compensating transaction), nhưng không phải EDA.

Nói chính xác hơn:
> **Saga là pattern giải quyết bài toán "consistency xuyên nhiều service".**
> **EDA là 1 trong các cách để triển khai giao tiếp giữa các bước của Saga** (cách còn lại là gọi trực tiếp/đồng bộ).

Saga không "cần" EDA để tồn tại — nhưng khi kết hợp với EDA (Choreography, hoặc Orchestration dùng command qua queue) thì được thêm lợi ích: loose coupling, resilience (service down tạm không mất event), không cần orchestrator giữ connection chờ đồng bộ. Đó là lý do trong thực tế microservices, người ta hay ghép **Saga + EDA + Outbox làm bộ 3 đi cùng nhau** — nhưng về khái niệm, chúng vẫn tách biệt, không phải cái này sinh ra cái kia.

**Phân biệt Command vs Event** — chìa khoá để hiểu vì sao Choreography khác Orchestration về bản chất giao tiếp:

| | Command | Event |
|---|---|---|
| Ngữ nghĩa | Mệnh lệnh — "hãy làm X" (thì tương lai) | Sự kiện đã xảy ra — "X đã xảy ra rồi" (thì quá khứ) |
| Ví dụ | `ReserveInventoryCommand`, `ChargePaymentCommand` | `InventoryReserved`, `PaymentCompleted` |
| Người gửi biết ai nhận? | Có — gửi đích danh tới đúng 1 service phải xử lý | Không — chỉ "thông báo", 0 hoặc nhiều subscriber tuỳ ý |
| Saga tương ứng | **Orchestration** — orchestrator ra lệnh cho từng service | **Choreography** — service tự publish, service khác tự nguyện subscribe |

- **Choreography Saga chính là 1 dạng thuần tuý của EDA** áp dụng riêng cho bài toán transaction — mọi giao tiếp giữa các bước đều qua publish/subscribe event, không ai gọi trực tiếp ai, không ai biết trước "ai sẽ xử lý tiếp theo".
- **Orchestration Saga** thường dùng **Command** hơn là Event thuần — orchestrator biết chính xác cần gọi Inventory Service rồi Payment Service, không phải "thông báo rồi ai muốn xử lý thì xử lý". Command này có thể gửi qua REST/RPC đồng bộ, hoặc qua message queue bất đồng bộ (command queue/topic riêng) — nếu dùng queue thì Orchestration vẫn chạy trên hạ tầng message-driven (vẫn hưởng lợi resilience/decoupling của queue), nhưng về **ngữ nghĩa giao tiếp** thì khác hẳn Choreography thuần.

**Vì sao đáng để phân biệt rõ:** khi thiết kế Orchestration Saga, rất dễ đặt tên các message là "event" (`OrderCreated`...) trong khi bản chất đang dùng như command (chỉ đúng 1 consumer xử lý, mang tính chỉ thị) — đặt sai ngữ nghĩa khiến code khó đọc, và vô tình khuyến khích thêm subscriber ngoài ý muốn (ai cũng nghĩ "event" thì được tự do lắng nghe, phá vỡ tính tường minh mà Orchestration vốn muốn có).

**Lưu ý:** `OrderCreated` bản thân **luôn là Event đúng nghĩa** ở cả 2 kiểu — Order Service publish "sự thật", không quan tâm ai nghe, đây là điều tốt (loose coupling), không phải chỗ dễ đặt sai tên. Chỗ dễ nhầm nằm ở **message phát sinh SAU** khi orchestrator nhận `OrderCreated` và quyết định gọi Inventory Service — đây mới là lúc orchestrator "ra lệnh cho đúng 1 service cụ thể", không còn là thông báo sự thật nữa.

**Quy ước đặt tên giúp tránh nhầm lẫn:**
- **Command** → đặt tên theo **thể mệnh lệnh** (verb ở đầu): `ReserveInventoryCommand`, `ChargePaymentCommand`.
- **Event** → đặt tên theo **thì quá khứ** (đã xảy ra rồi): `InventoryReserved`, `PaymentCompleted`, `OrderCreated`.

Ví dụ cụ thể: đặt tên message orchestrator gửi cho Inventory Service là `InventoryReservationRequested` (nghe như event — "1 yêu cầu đã được tạo ra") trong khi bản chất đang dùng với ngữ nghĩa command (chỉ đích danh Inventory Service phải xử lý) là **sai quy ước** — nên đặt là `ReserveInventoryCommand` (thể mệnh lệnh) để bất kỳ ai đọc code/log cũng nhận ra ngay đây là lệnh có 1 người nhận cụ thể, không phải broadcast cho ai muốn nghe cũng được.

**Lợi ích khi giao tiếp qua Event (Choreography, hoặc bất kỳ chỗ nào dùng publish/subscribe):**
- **Loose coupling thực sự** — Order Service không cần biết Inventory Service có tồn tại hay không, chỉ publish `OrderCreated`; sau này thêm Analytics Service nghe cùng event đó để thống kê mà **không cần sửa gì** ở Order Service.
- **Resilience tốt hơn** — nếu 1 service down tạm thời, event vẫn nằm chờ trong queue (nhờ Outbox + message broker), service sống lại tự động catch up, không mất dữ liệu.
- **Audit trail tự nhiên** — toàn bộ event log là lịch sử đầy đủ những gì đã xảy ra với 1 đơn hàng, hữu ích để debug/replay lại sau này.

**Đánh đổi/nhược điểm của EDA (áp dụng cho cả Choreography):**
- Khó nhìn "toàn cảnh" luồng nghiệp vụ chỉ qua code từng service (đã nói ở nhược điểm Choreography) — chính là lý do Orchestration ra đời, đánh đổi lấy lại tính tường minh.
- **Thứ tự xử lý event không tự động đảm bảo** trừ khi dùng cơ chế đặc biệt (ví dụ Kafka: partition theo `order_id` để mọi event của cùng 1 đơn hàng luôn vào 1 partition, giữ đúng thứ tự).
- **Schema của event thay đổi theo thời gian** (thêm/bớt field) cần chiến lược versioning rõ ràng — vì nhiều consumer độc lập cùng đọc 1 event, không thể "sửa 1 chỗ là xong" như khi sửa 1 API nội bộ.

**Dễ nhầm với 1 khái niệm khác — Event Sourcing:** đừng nhầm **Event-Driven Architecture** (cách các service *giao tiếp* với nhau) với **Event Sourcing** (cách 1 service *lưu trữ state* — lưu toàn bộ chuỗi event thay vì lưu trạng thái hiện tại, muốn biết state hiện tại phải replay lại toàn bộ event). Saga ở bài này chỉ dùng Outbox + event để **giao tiếp** giữa các service, mỗi service vẫn lưu trạng thái hiện tại bình thường trong bảng `orders`, `inventory`... — **không phải** Event Sourcing (dù 2 pattern này hay được dùng chung trong các hệ thống phức tạp hơn, chúng là 2 khái niệm độc lập).

### 2.4 Ví dụ minh hoạ đầy đủ: cùng 1 hạ tầng, khác nhau ở "ai quyết định bước tiếp theo"

**Orchestration — vòng lặp Event vào / Command ra / Event báo cáo về:**
```
OrderService      → publish "OrderCreated"                      [Event — sự thật]
Orchestrator      ← nhận "OrderCreated" (chỉ là 1 subscriber bình thường)
Orchestrator      → gửi "ReserveInventoryCommand" tới InventoryService  [Command — ra lệnh đích danh]
InventoryService  → trừ kho, rồi publish "InventoryReserved"    [Event — báo cáo sự thật]
Orchestrator      ← nhận "InventoryReserved"
Orchestrator      → gửi "ChargePaymentCommand" tới PaymentService       [Command]
PaymentService    → charge tiền, publish "PaymentCompleted"     [Event]
Orchestrator      ← nhận "PaymentCompleted" → gọi OrderService set CONFIRMED
```
Nhận xét: orchestrator **nhận Event vào, quyết định, rồi phát Command ra** — nó là nơi duy nhất "diễn giải" event thành hành động tiếp theo. Inventory Service/Payment Service không tự quyết định gì cả, chỉ thực thi đúng command nhận được rồi báo cáo lại bằng event.

**Choreography — chỉ toàn Event, không ai ra lệnh ai:**
```
OrderService      → publish "OrderCreated"
    ├─ InventoryService tự subscribe → tự quyết định trừ kho → publish "InventoryReserved"
    ├─ NotificationService tự subscribe → tự quyết định gửi email "đã nhận đơn"
    └─ AnalyticsService tự subscribe → tự quyết định ghi log thống kê
                                    (3 service trên hoàn toàn không biết về nhau)

PaymentService     tự subscribe "InventoryReserved" → tự quyết định charge tiền → publish "PaymentCompleted" (hoặc "PaymentFailed")
OrderService       tự subscribe "PaymentCompleted" → set status = CONFIRMED
                   (hoặc tự subscribe "InventoryReservationFailed"/"PaymentFailed" → set status = CANCELLED)
```
Không có bước nào có "command" cả — toàn bộ đều là Event, đúng đặc trưng Choreography.

Nhận xét: mỗi service **tự đọc event rồi tự quyết định** có phản ứng hay không, phản ứng như thế nào — không có ai đứng ra "diễn giải hộ". Đây là lý do thêm `NotificationService`/`AnalyticsService` không cần sửa gì ở `OrderService`, nhưng cũng chính là lý do khó nhìn "toàn cảnh luồng nghiệp vụ" khi đọc code — vì logic điều phối bị rải rác ở từng service, không tập trung 1 chỗ như Orchestration.

**Vì sao cùng là `OrderCreated`, nhưng lại "an toàn"/lỏng lẻo hơn ở Choreography so với việc Orchestrator tự ý gửi command trực tiếp:**
- `OrderCreated` không hề biết Inventory Service sẽ nghe nó và trừ kho — Order Service chỉ publish "sự thật", còn **Inventory Service tự mình chọn** phản ứng lại bằng cách trừ kho. Đó là quyết định của chính Inventory Service, không phải bị Order Service ép buộc.
- Vì thế, hoàn toàn có thể có **nhiều service khác cùng nghe `OrderCreated`** mà làm những việc hoàn toàn khác nhau (như `NotificationService`, `AnalyticsService` ở ví dụ trên) — tất cả độc lập, không ai biết về ai, không cần sửa gì ở Order Service khi thêm service mới. Đây chính là lợi ích loose-coupling.
- Ngược lại, khi Orchestrator gửi `ReserveInventoryCommand`, nó **chỉ định đích danh** Inventory Service phải làm đúng 1 việc — không có chuyện "ai nghe cũng được, tự quyết".
- Tóm lại: **cùng công nghệ hạ tầng (message broker), nhưng bản chất kiến trúc khác nhau ở chỗ "ai là người quyết định hành động tiếp theo"** — trong Choreography là từng service tự quyết (event thuần), trong Orchestration là orchestrator quyết rồi ra lệnh (command).

**Kết luận:** `OrderCreated` là Event dùng đúng ở cả 2 kiểu. Sự khác biệt nằm ở **các message phát sinh sau đó** — Orchestration tạo ra Command (vì có 1 "bộ não" trung tâm quyết định), Choreography chỉ toàn Event (vì quyết định nằm rải rác ở từng service tự trị).

### 2.5 Ví dụ compensate cụ thể: thanh toán thất bại → phải hoàn lại tồn kho

**Choreography — InventoryService phải tự subscribe thêm event thất bại của bước sau:**
```
OrderService     → publish "OrderCreated"
InventoryService → tự subscribe, trừ kho → publish "InventoryReserved"
PaymentService   → tự subscribe "InventoryReserved", charge tiền → THẤT BẠI → publish "PaymentFailed"

                          ↓ (compensating chain — ngược lại)

InventoryService → tự subscribe THÊM "PaymentFailed" → tự quyết định hoàn kho → publish "InventoryReleased"
OrderService     → tự subscribe "PaymentFailed" (hoặc "InventoryReleased") → set status = CANCELLED
```
**Điểm đáng chú ý:** `InventoryService` — service đã thực hiện hành động gốc (trừ kho) — giờ phải **tự đăng ký nghe thêm** `PaymentFailed`, dù đó là event của 1 service khác hẳn (Payment) không liên quan trực tiếp tới nó. Nó phải tự biết: "nếu nghe thấy `PaymentFailed` cho đơn mà mình đã từng trừ kho, thì phải tự hoàn lại".

Đây là **nhược điểm ẩn của Choreography khi có compensate**: càng nhiều bước, mỗi service càng phải "biết" thêm nhiều event thất bại của các bước phía sau nó để tự compensate đúng lúc — logic rollback bị rải rác, không nhìn thấy tường minh ở 1 chỗ. Saga chỉ 2-3 bước thì còn ổn, phình ra 5-6 bước thì rất dễ rối.

**Orchestration — Orchestrator ra lệnh compensate tường minh:**
```
Orchestrator     → gửi "ReserveInventoryCommand" → InventoryService trừ kho → publish "InventoryReserved"
Orchestrator     ← nhận "InventoryReserved" → cập nhật saga_state (step=PAYMENT_PENDING) → gửi "ChargePaymentCommand"
PaymentService   → charge tiền → THẤT BẠI → publish "PaymentFailed"
Orchestrator     ← nhận "PaymentFailed" → tra saga_state: "kho ĐÃ được trừ ở bước trước"
                 → gửi "ReleaseInventoryCommand" (Command, đích danh) → InventoryService cộng lại kho
InventoryService → publish "InventoryReleased" (Event, báo cáo)
Orchestrator     ← nhận "InventoryReleased" → gửi lệnh cho OrderService set CANCELLED
```
**Điểm đáng chú ý:** `InventoryService` **hoàn toàn không cần biết `PaymentFailed` tồn tại**. Nó chỉ phản ứng với đúng 2 command nó nhận được: `ReserveInventoryCommand` và `ReleaseInventoryCommand` — không cần hiểu "tại sao" hay "bối cảnh saga đang ở đâu". Toàn bộ logic "nếu thanh toán fail thì phải hoàn kho" nằm **duy nhất** trong orchestrator, tra cứu qua `saga_state` để biết chính xác bước nào đã thành công (cần compensate) và bước nào chưa (không cần đụng tới).

**2 điểm chung quan trọng, áp dụng cho cả 2 kiểu:**
1. **Chỉ compensate đúng những bước đã thực sự thành công** — nếu kho chưa từng bị trừ (ví dụ fail ngay ở bước trừ kho) thì không có gì để hoàn cả. Orchestration biết điều này qua `saga_state`; Choreography thì mỗi service tự kiểm tra state cục bộ của chính nó (ví dụ InventoryService tự query "đơn này mình có từng trừ kho không" trước khi quyết định hoàn) — đều dẫn tới đúng kết quả, chỉ khác nơi lưu trữ "biết bước nào đã xong".
2. **Hành động hoàn kho (compensating action) phải idempotent** — như đã nói ở mục 5, nếu `PaymentFailed`/`ReleaseInventoryCommand` bị nhận trùng (do redelivery), không được cộng kho 2 lần. Vẫn dựa vào `UNIQUE(event_id)` / `processed_events` y hệt cách đã làm cho hành động gốc.

---

## 3. Outbox Pattern — giải quyết vấn đề "dual write"

**Vấn đề:** Order Service khi tạo order cần làm 2 việc: (1) ghi vào DB `orders`, và (2) publish event `OrderCreated` lên message broker (Kafka/RabbitMQ) để Inventory Service biết mà xử lý tiếp. Nếu code đơn giản là:
```
INSERT INTO orders (...) VALUES (...);   -- (1) ghi DB — thành công
publish("OrderCreated", ...);            -- (2) publish event — nếu lỗi mạng, service crash ngay đây...
```
→ Có khoảng hở: DB đã commit order, nhưng event không tới nơi → Inventory Service không bao giờ biết để trừ kho → **saga bị "treo" vĩnh viễn**, order nằm mãi ở `PENDING` không ai xử lý tiếp. Đây gọi là vấn đề **dual write** — không thể đảm bảo atomic giữa "ghi vào 1 DB" và "gửi message tới 1 hệ thống khác" bằng cách làm 2 thao tác riêng lẻ.

**Giải pháp — Outbox pattern:**
1. Thay vì publish trực tiếp, ghi thêm 1 bản ghi event vào bảng `outbox_events` **trong cùng transaction DB** với việc tạo order:
```sql
BEGIN;
INSERT INTO orders (id, status, ...) VALUES (?, 'PENDING', ...);
INSERT INTO outbox_events (id, event_type, payload, status)
  VALUES (?, 'OrderCreated', '{...}', 'PENDING');
COMMIT;
```
Vì cả 2 insert nằm trong **1 transaction của cùng 1 DB thông thường** (không phải distributed transaction), DB đảm bảo atomic bình thường — hoặc cả 2 cùng ghi, hoặc không ghi gì cả. Không còn khoảng hở.

2. Một tiến trình riêng — **Message Relay** (hoặc dùng **CDC – Change Data Capture**, ví dụ Debezium đọc trực tiếp transaction log của DB) — liên tục quét bảng `outbox_events` có `status = 'PENDING'`, publish từng event lên message broker, rồi đánh dấu `status = 'PUBLISHED'` (hoặc xoá record).

3. Vì bảng outbox đã persist sẵn trong DB, dù Message Relay có crash giữa chừng, khi khởi động lại nó vẫn tiếp tục quét các event `PENDING` còn sót → đảm bảo **at-least-once delivery** (event chắc chắn được publish, có thể publish trùng — nên consumer phía sau luôn cần idempotency, xem mục 4).

```
┌──────────────┐   1 transaction    ┌─────────┐
│ Order Service│ ─────────────────► │  DB      │
└──────────────┘   orders +         │ orders   │
                    outbox_events   │ outbox   │
                                    └────┬─────┘
                                         │ 2. poll PENDING
                                         ▼
                                 ┌───────────────┐
                                 │ Message Relay  │ 3. publish
                                 │ (hoặc Debezium)│ ─────────► Kafka/RabbitMQ
                                 └───────────────┘
```

Outbox pattern áp dụng y hệt cho Inventory Service (khi publish `InventoryReserved`/`InventoryFailed`) và Payment Service (khi publish `PaymentCompleted`/`PaymentFailed`) — bất kỳ chỗ nào cần vừa ghi DB vừa gửi event đều nên qua Outbox.

---

## 4. Idempotency — bắt buộc ở mọi consumer

Vì message broker + Outbox đảm bảo **at-least-once** (không phải **exactly-once**), mỗi service có thể **nhận trùng event** (do retry, do Message Relay publish 2 lần, do consumer ack chậm...). Áp dụng đúng kỹ thuật đã bàn ở `question.md` (câu 9 — webhook thanh toán trùng):

```sql
BEGIN;
INSERT INTO processed_events (event_id) VALUES (?);
-- vi phạm UNIQUE(event_id) → đã xử lý rồi → ROLLBACK, bỏ qua (không trừ kho/charge tiền lần 2)
UPDATE inventory SET stock = stock - ? WHERE product_id = ? AND stock >= ?;
COMMIT;
```
Mỗi service consume event đều cần bảng `processed_events` (hoặc field tương đương) với `UNIQUE(event_id)` để tự bảo vệ khỏi xử lý trùng — đây là điều kiện bắt buộc khi làm Saga, không phải tuỳ chọn.

---

## 5. Compensating transaction phải Idempotent + có Retry

Khi Payment Service báo lỗi, Inventory Service phải hoàn kho (compensate). Compensating transaction cũng là 1 dạng consumer nhận event (`PaymentFailed`) → cũng phải:
- **Idempotent:** nếu nhận event `PaymentFailed` trùng 2 lần, chỉ hoàn kho đúng 1 lần (dựa vào `UNIQUE(event_id)` như trên).
- **Có retry tự động:** nếu bước hoàn kho tạm thời lỗi (DB timeout...), message broker sẽ redeliver, không được để saga "kẹt" ở trạng thái nửa vời (đã trừ kho, chưa hoàn) — nên có thêm cơ chế theo dõi các saga bị timeout/kẹt quá lâu (xem mục 6).

---

## 6. Saga state / trạng thái từng saga instance

Đặc biệt quan trọng với **Orchestration**: orchestrator cần lưu lại trạng thái hiện tại của từng saga (`order_id`, đang ở bước nào, đã gọi service nào thành công, đang chờ gì) — thường là 1 bảng `saga_state` dạng state machine:

| order_id | current_step | status | updated_at |
|---|---|---|---|
| 123 | INVENTORY_RESERVED | IN_PROGRESS | ... |
| 124 | PAYMENT_FAILED | COMPENSATING | ... |
| 125 | CONFIRMED | COMPLETED | ... |

Lợi ích:
- Nếu orchestrator restart giữa chừng, đọc lại `saga_state` để biết cần resume từ đâu, không mất dấu saga nào.
- Có thể quét các saga bị "kẹt" quá lâu ở 1 trạng thái (ví dụ `IN_PROGRESS` quá 5 phút không có update) để tự động timeout → trigger compensate, tránh order treo vô thời hạn.

---

## 7. Luồng end-to-end minh hoạ (Orchestration + Outbox)

**Happy path:**
1. User gọi API đặt hàng → Order Service: `INSERT order (status=PENDING)` + `INSERT outbox_event(OrderCreated)` trong 1 transaction → trả `202 Accepted` kèm `orderId` ngay (giống pattern async job đã bàn ở `nodejs_and_java.md` câu 3).
2. Orchestrator nhận `OrderCreated` (qua Message Relay publish) → gọi Inventory Service.
3. Inventory Service: atomic `UPDATE inventory SET stock = stock - ? WHERE stock >= ?` trong transaction + ghi `outbox_event(InventoryReserved)` → publish.
4. Orchestrator nhận `InventoryReserved` → gọi Payment Service.
5. Payment Service: gọi cổng thanh toán, nếu thành công → ghi nhận + `outbox_event(PaymentCompleted)` → publish.
6. Orchestrator nhận `PaymentCompleted` → gọi Order Service update `status = CONFIRMED`.
7. User poll API `GET /orders/{id}` (giống pattern task/status ở `nodejs_and_java.md` câu 3) thấy `status = CONFIRMED`.

**Failure path (thanh toán lỗi ở bước 5):**
5'. Payment Service: charge thất bại (thẻ hết hạn, không đủ tiền...) → `outbox_event(PaymentFailed)` → publish.
6'. Orchestrator nhận `PaymentFailed` → gọi **compensating action**: Inventory Service cộng lại kho (`UPDATE inventory SET stock = stock + ?`), rồi Order Service set `status = CANCELLED`.
7'. User poll thấy `status = CANCELLED` kèm lý do.

---

## 8. Các vấn đề vận hành cần lưu ý thêm

1. **Correlation ID / Trace ID:** mỗi saga cần 1 `correlation_id` (thường dùng luôn `order_id`) gắn vào mọi event/log xuyên suốt các service, để khi debug 1 đơn hàng cụ thể có thể trace toàn bộ luồng qua nhiều service (kết hợp distributed tracing — Jaeger/Zipkin/OpenTelemetry).
2. **Dead Letter Queue (DLQ):** nếu 1 message xử lý lỗi liên tục vượt quá số lần retry cho phép (ví dụ lỗi do bug, không phải lỗi tạm thời), đẩy sang DLQ để người vận hành xử lý thủ công, tránh nó chặn cả queue chính (head-of-line blocking) hoặc retry vô hạn.
3. **Race condition tại từng service vẫn cần xử lý riêng:** Saga chỉ giải quyết vấn đề **cross-service consistency**; bên trong Inventory Service, nếu sản phẩm hot (flash-sale) vẫn cần atomic update / xử lý hotspot như đã bàn kỹ ở `question.md` (câu 2) — Saga không thay thế các kỹ thuật đó, chỉ đứng ở tầng cao hơn.
4. **Timeout cho từng bước:** nếu 1 service không phản hồi trong X giây (service down, event bị thất lạc dù đã có Outbox — ví dụ Message Relay bị lag), orchestrator cần có cơ chế timeout để tự coi bước đó là fail và trigger compensate, không chờ vô thời hạn.

---

## 9. Khi nào cần Saga, khi nào không cần

- **Không cần Saga** nếu toàn bộ logic (đặt hàng, trừ kho, tạo order) có thể gom chung vào **1 service, 1 DB** — lúc đó chỉ cần 1 transaction DB thông thường (`BEGIN...COMMIT/ROLLBACK`) như đã làm ở `question.md` câu 4, đơn giản và đáng tin cậy hơn Saga rất nhiều. Đừng tách microservices/Saga sớm khi chưa thực sự cần scale độc lập từng phần.
- **Cần Saga** khi các bước thực sự thuộc về các service/DB độc lập không thể gộp transaction (ví dụ Payment Service gọi ra cổng thanh toán bên thứ 3 — vốn dĩ đã không thể nằm trong 1 DB transaction với phần còn lại), hoặc khi cần scale/deploy độc lập từng service vì lý do tổ chức (nhiều team, nhiều tốc độ release khác nhau).

---

## Tóm tắt kỹ thuật liên quan

| Pattern/Kỹ thuật | Vai trò trong luồng này |
|---|---|
| **Saga (Choreography/Orchestration)** | Thay thế distributed transaction bằng chuỗi local transaction + compensating transaction |
| **Outbox pattern** | Đảm bảo "ghi DB" + "publish event" atomic, không mất event khi service crash giữa chừng |
| **CDC (Debezium...)** | Cách triển khai Message Relay không cần polling thủ công, đọc thẳng transaction log |
| **Idempotency key (`UNIQUE(event_id)`)** | Chống xử lý trùng do at-least-once delivery của message queue |
| **Atomic update (`UPDATE ... WHERE`)** | Vẫn cần bên trong từng service (ví dụ trừ kho) để tự bảo vệ khỏi race condition nội bộ — xem `question.md` |
| **Saga state / state machine** | Cho phép orchestrator resume đúng chỗ nếu bị restart, và phát hiện saga bị kẹt để tự timeout |
| **Dead Letter Queue** | Cô lập message lỗi liên tục, tránh chặn toàn bộ queue |

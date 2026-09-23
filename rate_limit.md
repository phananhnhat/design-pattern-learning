https://helloderek.substack.com/p/system-design-interview-218-thiet

“How would you design a rate limiter for our API?”

Cần làm rõ:
- Giới hạn theo user hay theo IP?
- Distributed thì làm thế nào?
- Redis hay in-memory?
- Token Bucket hay Leaky Bucket?
- Nếu Redis down thì sao?
- Nếu có hàng triệu request mỗi giây?

Rate Limiter là cơ chế giới hạn số lượng request mà một client (user, IP, API key…) có thể gửi trong một khoảng thời gian nhất định.
Mục đích:
1. Chống abuse và DDoS (ở mức ứng dụng)
2. Bảo vệ tài nguyên đắt đỏ
3. Đảm bảo fairness
   Không ai muốn một user chiếm hết tài nguyên, trong khi 1.000 user khác phải chờ.
4. Là một phần của kiến trúc trưởng thành


## Giới hạn cái gì, và cho ai?

Rate limit theo cái gì?

**IP address**

Dễ làm, nhưng: NAT chung IP → oan

Dễ bị spoof

**User ID**

Tốt nếu đã login. Nhưng trước login thì sao?

**API key / token**
Phổ biến với public API. Dễ kiểm soát, dễ revoke

**Endpoint cụ thể**

/login khác /search , /payment khác /profile

“Tùy use case, sẽ rate limit theo user ID hoặc API key, và có thể khác nhau cho từng endpoint.”

## Rate Limiter đặt ở đâu trong hệ thống?

#### Option 1: Ở client?
Thường chỉ để nhẹ nhàng nhắc nhở.
Không bao giờ đủ tin cậy.

#### Option 2: Ở API Gateway / Load Balancer
Rất phổ biến:

Nginx, Envoy, Kong, AWS API Gateway

Ưu điểm:

- Chặn sớm
- Giảm tải backend

Nhược điểm:

- Logic phức tạp khó custom
- Phụ thuộc hạ tầng

#### Option 3: Ở backend service

- Linh hoạt nhất.  Custom logic
- Biết context user

Nhưng phải cẩn thận:

- Distributed
- Performance

“Với hệ thống lớn, ưu tiên rate limit ở API Gateway cho các rule đơn giản, và backend cho rule phức tạp hơn.”

## Bài toán cốt lõi: Làm sao đếm request?

#### Cách ngây thơ nhất: Counter + time window.
Ví dụ:

Mỗi user có một counter

Reset mỗi phút

Vấn đề?
- Burst request ở ranh giới phút
- Không mượt
- Trải nghiệm user kém

#### Fixed Window – đơn giản nhưng hơi thô (chí là Counter + time window.)
Ý tưởng
- Chia thời gian thành các cửa sổ cố định (ví dụ: mỗi phút)
- Mỗi cửa sổ có một counter

Ví dụ
- Limit: 100 request / phút
- User gửi 100 request lúc 12:00:59
- Gửi thêm 100 request lúc 12:01:01

=> 👉 Tổng: 200 request trong 2 giây
=> Hệ thống vẫn “đúng luật”, nhưng user thì hơi… gian.

Khi nào dùng?
- Hệ thống nhỏ
- Không quá nhạy cảm với burst

#### Sliding Window – mượt hơn, nhưng tốn hơn

Ý tưởng: Không reset cứng theo phút, mà nhìn vào khoảng thời gian trượt.

Ví dụ:
- Luôn xét 60 giây gần nhất
- Đếm số request trong khoảng đó

Ưu điểm
- Tránh burst
- Công bằng hơn

Nhược điểm
- Phải lưu timestamp từng request
- Tốn bộ nhớ
- Khó scale

**Nhiều hệ thống “học thuật” thích, nhưng production thì hay cân nhắc.**


#### Leaky Bucket – đều như nước nhỏ giọt

Hãy tưởng tượng một cái xô có lỗ thủng ở đáy.

Request vào → đổ nước vào xô

Nước rò rỉ ra với tốc độ cố định

Nếu xô đầy → request bị từ chối.

Ưu điểm
- Output rất đều
- Hợp với hệ thống cần ổn định

Nhược điểm
- Không cho phép burst
- User cảm giác bị “chậm”

Leaky Bucket hợp với:
- Job processing
- Message queue

Ít dùng cho public API.


#### Token Bucket – “ngôi sao” của phỏng vấn

Nếu có một thuật toán mà interviewer mong bạn nhắc đến, thì đó là Token Bucket.

- Ý tưởng rất đời
- Mỗi user có một cái xô token
- Token được thêm vào đều đặn (ví dụ: 10 token / giây)
- Mỗi request cần 1 token
- Hết token → chờ

Vì sao Token Bucket được yêu thích?
- Cho phép burst (nếu còn token)
- Vẫn kiểm soát được tốc độ trung bình
- Linh hoạt

**“Token Bucket là lựa chọn phổ biến vì nó cân bằng giữa kiểm soát và trải nghiệm user.”**

Nghe rất “senior”.


#### Token Bucket bằng Redis: nói sao cho đúng phỏng vấn

Vì sao Redis?
- Nhanh
- In-memory
- Hỗ trợ atomic operation
- Phù hợp distributed

##### Redis value có thể trông như thế nào?

Ví dụ key:

rate_limit:user:123
Value (hash):

tokens: 42

last_refill_ts: 1700000000

Hoặc gọn hơn:

- Dùng Lua script để:
    - Tính số token cần refill
    - Trừ token
    - Trả kết quả atomically

Interviewer rất thích nghe từ “Lua script” ở đây.

Ví dụ flow thực tế
1. Request tới API
2. Middleware gọi Redis
3. Redis script:
   - Tính token mới
   - Nếu đủ → trừ token, return OK
   - Nếu không → return reject

4. API trả 429 Too Many Requests

Nghe có vẻ dài, nhưng nói gọn trong phỏng vấn chỉ mất 30–40 giây.

### Distributed Rate Limiter – cơn ác mộng nhẹ

Khi hệ thống có:
- Nhiều instance
- Autoscaling
- Multi-region

Thì:
- In-memory counter là không ổn
- Redis / centralized store gần như bắt buộc

Vấn đề thường gặp
- Redis latency
- Redis down
- Hot key

“Nếu Redis gặp sự cố, em có thể fallback sang local limiter với rule nhẹ hơn để tránh system chết hoàn toàn.”

## Một chi tiết nhỏ, nhưng rất “đời”.

Khi reject request, đừng chỉ trả: 429 Too Many Requests

Hãy thêm:
- Retry-After header
- Message rõ ràng

User không thích bị từ chối. Nhưng họ chấp nhận nếu biết khi nào quay lại.

## Những câu hỏi interviewer hay “xoáy”
Bạn nên chuẩn bị sẵn:
- Nếu có hàng triệu user thì sao?
- Nếu mỗi endpoint rule khác nhau?
- Làm sao test rate limiter?
- Làm sao monitoring?
- Có cache không?

Không cần trả lời hoàn hảo.
Chỉ cần bình tĩnh, logic, và có trade-off.

## Một chút lan man, nhưng đáng nói
Thú vị là, rất nhiều bug production đến từ:
- Rate limiter quá chặt
- Hoặc quá lỏng

Deploy một rule nhỏ, traffic biến dạng, user complain, business gọi lúc 2 giờ sáng.

Rate Limiter không chỉ là bài toán phỏng vấn. Nó là bài toán vận hành.
### Khi nào dùng index, khi nào không dùng index trong DB ?

**Trả lời:**

**Cơ chế chung (để hiểu trade-off):** Index (thường là B-Tree) là 1 cấu trúc dữ liệu phụ, lưu sẵn giá trị cột đã sắp xếp kèm con trỏ tới row thật, giúp DB tìm dữ liệu theo kiểu `O(log n)` thay vì phải quét toàn bảng `O(n)`. Đánh đổi: **đọc nhanh hơn**, nhưng **mỗi lần ghi (INSERT/UPDATE/DELETE) phải cập nhật thêm cả index** → ghi chậm hơn, và tốn thêm dung lượng lưu trữ. Quyết định dùng index hay không luôn xoay quanh việc cân bằng 2 chi phí này.

---

## Khi nào NÊN dùng index

1. **Cột thường xuyên xuất hiện trong `WHERE`** — ví dụ `WHERE email = ?`, `WHERE status = ? AND created_at > ?`. Đây là lý do phổ biến nhất để thêm index.
2. **Cột dùng để `JOIN`** — gần như bắt buộc phải có index trên khoá ngoại (foreign key), nếu không mỗi lần JOIN sẽ phải full scan bảng kia.
3. **Cột dùng trong `ORDER BY` / `GROUP BY`** — index giúp DB trả dữ liệu đã sắp xếp sẵn, tránh phải sort thủ công tốn CPU/memory sau khi lấy dữ liệu.
4. **Cột cần đảm bảo tính duy nhất** (`UNIQUE INDEX`) — email, username, mã đơn hàng... (xem thêm các case ở `question.md` — unique constraint vừa đảm bảo đúng đắn dữ liệu, vừa tự động có index đi kèm).
5. **Cột có độ chọn lọc cao (high cardinality)** — giá trị càng đa dạng (gần unique) thì index càng hiệu quả, vì mỗi lần tra cứu loại bỏ được phần lớn dữ liệu không liên quan. Ví dụ: `email`, `order_id`, `phone_number`.
6. **Bảng lớn và đọc nhiều hơn ghi (read-heavy)** — bảng vài triệu dòng trở lên, tần suất SELECT theo điều kiện cao hơn nhiều so với INSERT/UPDATE/DELETE → lợi ích tăng tốc đọc lớn hơn chi phí ghi chậm đi.
7. **Composite index cho query nhiều điều kiện kết hợp** — ví dụ hay query `WHERE user_id = ? AND status = ?` thì tạo `INDEX(user_id, status)` thay vì 2 index riêng lẻ, hiệu quả hơn nhiều (lưu ý *leftmost prefix rule*: index `(a, b)` chỉ tối ưu được cho query lọc theo `a`, hoặc `a` và `b`, không tối ưu cho query chỉ lọc theo `b`).
8. **Covering index** — khi index chứa đủ tất cả cột mà query cần (cả điều kiện lọc lẫn cột SELECT ra), DB có thể trả kết quả thẳng từ index mà không cần đọc thêm bảng chính (tránh "bookmark lookup"), rất nhanh cho các query đọc lặp lại nhiều.

## Khi nào KHÔNG NÊN dùng index

1. **Bảng nhỏ** (vài trăm đến vài nghìn dòng) — full table scan trên bảng nhỏ đã đủ nhanh (có thể load gần hết vào memory/cache), chi phí duyệt thêm cấu trúc B-Tree của index đôi khi còn chậm hơn quét thẳng. DB optimizer thường tự bỏ qua index trong trường hợp này dù có tạo sẵn.
2. **Cột có độ chọn lọc thấp (low cardinality)** — ví dụ cột `gender` (2-3 giá trị), `is_active` (boolean), `status` chỉ có vài enum mà phân bố đều. Index trên các cột này không giúp loại bỏ được nhiều dữ liệu (mỗi giá trị vẫn ứng với % lớn số dòng), DB thường chọn full scan thay vì dùng index dù có tồn tại.
3. **Cột ít khi/không bao giờ xuất hiện trong `WHERE`, `JOIN`, `ORDER BY`** — index chỉ tốn chi phí ghi + storage mà không mang lại lợi ích đọc nào, vì không có query nào tận dụng nó.
4. **Bảng ghi nhiều (write-heavy)** — bảng log, bảng event, bảng tracking... insert liên tục với tần suất cao. Mỗi index thêm vào là thêm 1 lần ghi phụ mỗi khi INSERT/UPDATE/DELETE → càng nhiều index, ghi càng chậm. Cần cân nhắc kỹ, chỉ giữ index thực sự cần thiết (ví dụ chỉ giữ index phục vụ truy vấn báo cáo quan trọng, bỏ index "phòng khi cần").
5. **Cột thường xuyên bị UPDATE giá trị** — mỗi lần giá trị cột đổi, DB phải cập nhật lại vị trí trong cấu trúc index (xoá entry cũ, thêm entry mới) → nếu cột đó update liên tục (ví dụ 1 counter tăng liên tục), index trên nó tạo overhead ghi lớn.
6. **Cột kiểu dữ liệu lớn (TEXT, BLOB, JSON nguyên khối)** — không nên đánh index trực tiếp toàn bộ giá trị (tốn storage, so sánh chậm). Nếu cần tìm kiếm theo nội dung, dùng giải pháp chuyên biệt: **full-text index**, **prefix index** (chỉ index N ký tự đầu), hoặc index trên 1 field JSON cụ thể (generated/computed column) thay vì cả blob.
7. **Quá nhiều index trên cùng 1 bảng** — mỗi index thêm chi phí ghi + storage, đồng thời khiến query optimizer mất thời gian hơn để chọn index tối ưu (đôi khi chọn sai index làm chậm hơn). Nguyên tắc: chỉ tạo index có mục đích rõ ràng, định kỳ rà soát xoá index không còn được dùng.
8. **Query luôn trả về phần lớn dữ liệu bảng** (ví dụ báo cáo `SELECT *` không lọc gì, hoặc điều kiện lọc chỉ loại bỏ được rất ít dòng) — index không giúp ích vì bản chất vẫn phải đọc gần hết dữ liệu; lúc này full scan (đọc tuần tự) thường nhanh hơn nhảy qua nhảy lại theo index (random I/O).

## Cách xác định có nên thêm index hay không

- Dùng `EXPLAIN` / `EXPLAIN ANALYZE` (MySQL, PostgreSQL) để xem query plan thực tế: kiểm tra DB có đang dùng index không, ước lượng số dòng phải quét, so sánh chi phí trước/sau khi thêm index.
- Ưu tiên index cho các query **chạy thường xuyên và quan trọng** (dashboard, API nóng), không cần tối ưu sớm cho query hiếm khi chạy (báo cáo cuối tháng chạy 1 lần có thể chấp nhận chậm hơn).
- Theo dõi index không dùng tới (`pg_stat_user_indexes` ở Postgres, `sys.schema_unused_indexes` ở MySQL) để dọn dẹp định kỳ — index "để đó phòng khi cần" âm thầm làm chậm mọi lệnh ghi mà không ai nhận ra.

## Tóm tắt nhanh

| Nên dùng index | Không nên dùng index |
|---|---|
| Cột trong WHERE/JOIN/ORDER BY thường xuyên | Cột hiếm khi được query tới |
| Độ chọn lọc cao (gần unique) | Độ chọn lọc thấp (ít giá trị, phân bố đều) |
| Bảng lớn, đọc nhiều hơn ghi | Bảng nhỏ (full scan đã đủ nhanh) |
| Cần đảm bảo unique (email, mã đơn...) | Bảng ghi rất nhiều (log, event, tracking) |
| Query lặp lại nhiều, cần tốc độ ổn định | Cột bị update giá trị liên tục |
| Có thể tận dụng covering index | Cột kiểu dữ liệu lớn (TEXT/BLOB) đánh index thô |

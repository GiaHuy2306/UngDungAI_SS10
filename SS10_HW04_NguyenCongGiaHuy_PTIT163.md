## Bài 4: Xây Dựng Yêu Cầu Phi Chức Năng - NFR

### Prompt thiết kế
```
Act as a Senior System Analyst and Solution Architect.

Mục tiêu:
Hãy sinh phần Đặc tả Yêu cầu Phi chức năng (Non-Functional Requirements - NFR) cho hệ thống Guai-api của Shop AI, phục vụ chuẩn bị hạ tầng cho các sự kiện traffic lớn.

Ngữ cảnh:
Guai-api là backend API cho hệ thống Shop AI. Hệ thống có các chức năng tra cứu sản phẩm, xác thực người dùng bằng JWT, phân quyền truy cập và lưu trữ dữ liệu trên MySQL. Trong các sự kiện traffic lớn, hệ thống cần duy trì hiệu năng ổn định, giảm độ trễ và đảm bảo an toàn bảo mật.

Ràng buộc bắt buộc:
1. Phải phân tích và đưa ra chỉ tiêu kỹ thuật rõ ràng cho thời gian phản hồi tối đa của các endpoint tra cứu sản phẩm.
2. Phải phân tích hiệu suất cơ sở dữ liệu MySQL và đề xuất chiến lược Indexing cụ thể.
3. Phải phân tích độ trễ và an toàn bảo mật của luồng cấp phát/xác thực JWT.
4. Các NFR phải có mã định danh riêng.
5. Mỗi NFR phải có chỉ số đo lường được, không viết chung chung.
6. Nếu có giả định về tải hệ thống hoặc kiến trúc, hãy ghi rõ trong phần Assumptions.
7. Ưu tiên các yêu cầu có thể kiểm thử bằng monitoring, load test hoặc security test.

Định dạng đầu ra:
Trả về tài liệu dưới dạng Markdown Table với các cột:
- NFR ID
- Category
- Requirement
- Technical Metric / Target
- Rationale
- Validation Method
- Priority
```

### Bảng tài liệu NFR do AI sinh ra

| NFR ID | Category | Requirement | Technical Metric / Target | Rationale | Validation Method | Priority |
|---|---|---|---|---|---|---|
| NFR-PERF-001 | Response Time | Các endpoint tra cứu sản phẩm phải phản hồi trong thời gian chấp nhận được khi traffic tăng cao. | P95 response time ≤ 500ms cho các API tra cứu sản phẩm như `/products`, `/products/{id}`, `/products/search`. P99 response time ≤ 1s trong điều kiện tải cao. | Tra cứu sản phẩm là luồng có tần suất truy cập lớn, ảnh hưởng trực tiếp đến trải nghiệm người dùng và tỷ lệ chuyển đổi. | Thực hiện load test bằng JMeter, k6 hoặc Gatling; theo dõi P95/P99 latency qua APM. | Critical |
| NFR-PERF-002 | Response Time | API chi tiết sản phẩm phải có độ trễ thấp hơn API tìm kiếm phức tạp. | `/products/{id}` phải đạt P95 ≤ 300ms; `/products/search` phải đạt P95 ≤ 700ms khi có lọc/sắp xếp. | API chi tiết sản phẩm thường truy vấn theo khóa chính nên cần tối ưu nhanh hơn API tìm kiếm có nhiều điều kiện. | Load test theo từng endpoint; kiểm tra log truy vấn và APM trace. | High |
| NFR-DB-001 | MySQL Performance | Các truy vấn sản phẩm thường dùng phải được tối ưu bằng Index phù hợp. | Tạo index cho các cột thường dùng trong điều kiện tìm kiếm/lọc như `product_id`, `category_id`, `status`, `price`, `created_at`. | Index giúp giảm full table scan và cải thiện tốc độ truy vấn trong thời điểm traffic cao. | Dùng `EXPLAIN ANALYZE` để kiểm tra query plan; đảm bảo truy vấn chính sử dụng index. | Critical |
| NFR-DB-002 | MySQL Indexing Strategy | Hệ thống phải sử dụng composite index cho các truy vấn lọc kết hợp. | Đề xuất composite index như `(category_id, status)`, `(status, created_at)`, `(category_id, status, price)` tùy theo pattern truy vấn thực tế. | Các API danh sách sản phẩm thường lọc theo danh mục, trạng thái bán và khoảng giá; composite index giúp tối ưu truy vấn đa điều kiện. | Phân tích slow query log; benchmark trước/sau khi thêm index; xác nhận bằng `EXPLAIN`. | High |
| NFR-DB-003 | MySQL Performance | Hệ thống phải kiểm soát truy vấn chậm trong các sự kiện traffic lớn. | 95% truy vấn sản phẩm phải hoàn thành ≤ 100ms ở DB layer; không có query sản phẩm vượt quá 500ms trong điều kiện tải chuẩn. | DB là điểm nghẽn phổ biến khi traffic tăng. Kiểm soát query latency giúp giảm response time tổng thể của API. | Bật slow query log; giám sát bằng MySQL Performance Schema hoặc APM database tracing. | High |
| NFR-DB-004 | MySQL Indexing Safety | Việc bổ sung index phải cân bằng giữa hiệu năng đọc và chi phí ghi. | Không thêm index trùng lặp; mỗi index mới phải gắn với ít nhất một query pattern cụ thể. | Quá nhiều index làm chậm thao tác insert/update và tăng dung lượng lưu trữ. | Review schema; kiểm tra duplicate index; benchmark write performance sau khi thêm index. | Medium |
| NFR-SEC-001 | JWT Latency | Luồng cấp phát JWT khi đăng nhập phải có độ trễ thấp và ổn định. | P95 latency cho thao tác sinh JWT ≤ 100ms, không tính thời gian truy vấn DB xác thực người dùng. | JWT được cấp trong luồng đăng nhập, nếu chậm sẽ ảnh hưởng trải nghiệm người dùng và khả năng xử lý đăng nhập đồng thời. | Đo latency tại service authentication; load test đăng nhập đồng thời. | High |
| NFR-SEC-002 | JWT Validation Latency | Việc xác thực JWT trên các request bảo vệ phải không tạo độ trễ lớn. | P95 JWT validation latency ≤ 20ms/request tại security filter. | JWT validation diễn ra ở nhiều request, nên độ trễ nhỏ nhưng lặp lại thường xuyên sẽ ảnh hưởng toàn hệ thống. | Đo thời gian xử lý trong JWT Security Filter; kiểm tra APM trace theo middleware/filter. | Critical |
| NFR-SEC-003 | JWT Security | JWT phải được ký và kiểm tra chữ ký trước khi cho phép truy cập tài nguyên bảo vệ. | Token phải dùng thuật toán ký an toàn như HS256 với secret đủ mạnh hoặc RS256 với key pair; reject token sai chữ ký, sai định dạng hoặc hết hạn. | Ngăn giả mạo token và truy cập trái phép vào API. | Security test với token sai chữ ký, token expired, token bị chỉnh payload; kiểm tra HTTP 401. | Critical |
| NFR-SEC-004 | JWT Expiration | JWT phải có thời gian hết hạn rõ ràng và được kiểm tra trên mọi request cần xác thực. | Access token nên có TTL ngắn, ví dụ 15-60 phút; mọi request có token expired phải bị từ chối với HTTP 401. | Giảm rủi ro khi token bị lộ và đảm bảo phiên đăng nhập được kiểm soát. | Test token expired; kiểm tra log và response code; review cấu hình TTL. | Critical |
| NFR-SEC-005 | JWT Data Minimization | Payload JWT không được chứa dữ liệu nhạy cảm. | JWT payload chỉ chứa thông tin tối thiểu như `userId`, `role`, `issuedAt`, `expiresAt`; không chứa password, token bí mật, thông tin thanh toán hoặc dữ liệu cá nhân nhạy cảm. | JWT có thể bị decode ở phía client, nên không được chứa dữ liệu nhạy cảm dù đã ký. | Review payload token; security test và code review authentication service. | High |
| NFR-OPS-001 | Observability | Hệ thống phải có cơ chế giám sát hiệu năng cho API, MySQL và JWT filter. | Thu thập metrics về response time, DB query latency, slow query count, JWT validation failure rate và authentication error rate. | Monitoring giúp phát hiện sớm nghẽn cổ chai trong sự kiện traffic lớn. | Kiểm tra dashboard APM/logging; thiết lập alert khi vượt ngưỡng. | High |
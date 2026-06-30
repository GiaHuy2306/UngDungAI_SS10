## Bài 1: Phân tích & lựa chọn chiến lược tài liệu hóa

**Đáp án chọn: Phương án B**

### Lý do chọn phương án B

- Phương án B là tối ưu nhất vì Antigravity có thể khởi tạo workspace và index toàn bộ thư mục `Guai-api`.
- Checkout Flow không nằm trong một file duy nhất mà trải dài qua Controller, Service, Repository và JWT Security Filter.
- Khi index toàn bộ mã nguồn, AI có thể hiểu luồng xử lý end-to-end từ endpoint `/api/v1/checkout` đến các lớp xử lý liên quan.
- Prompt yêu cầu AI đóng vai System Analyst nên đầu ra phù hợp với mục tiêu viết tài liệu SRS.
- Kết quả có thể được tổng hợp trực tiếp thành các phần như:
  - Pre-conditions
  - Main flow
  - Exceptions
- Phương án này giảm rủi ro thất thoát ngữ cảnh vì AI không chỉ đọc từng đoạn code riêng lẻ mà phân tích toàn bộ luồng nghiệp vụ.

### Lý do loại trừ phương án A

- Phương án A chỉ phân tích từng đoạn code được bôi đen trong VSCode.
- AI dễ mất ngữ cảnh giữa các file, đặc biệt khi logic checkout nằm ở nhiều lớp khác nhau.
- Người dùng phải tự tổng hợp lại từ nhiều phần giải thích rời rạc.
- Rủi ro bỏ sót logic nghiệp vụ cao, ví dụ như kiểm tra JWT, xử lý lỗi, kiểm tra dữ liệu hoặc cập nhật trạng thái thanh toán.
- Đầu ra thường thiên về giải thích kỹ thuật hơn là đặc tả nghiệp vụ theo định dạng SRS.

### Lý do loại trừ phương án C

- Phương án C phụ thuộc vào việc người dùng copy đúng và đủ các file liên quan.
- Nếu thiếu một file hoặc thiếu một đoạn code quan trọng, AI sẽ phân tích sai hoặc thiếu.
- Việc dán thủ công vào ChatGPT/Gemini dễ gây thất thoát ngữ cảnh do giới hạn độ dài prompt.
- AI không thể tự truy vết dependency hoặc quan hệ giữa các file trong toàn bộ project.
- Cách này kém hiệu quả với dự án có mã nguồn lớn và logic nghiệp vụ phân tán.

### Kết luận

Phương án B là lựa chọn phù hợp nhất vì tận dụng được sức mạnh của Antigravity trong việc index toàn bộ workspace, đọc hiểu luồng xử lý nhiều file và chuyển hóa logic code thành tài liệu SRS. Hai phương án còn lại có rủi ro lớn về context loss, thiếu file liên quan và yêu cầu người dùng tự tổng hợp thủ công.
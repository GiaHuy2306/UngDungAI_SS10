# Bài 5: Khởi tạo Workflow SRS hoàn chỉnh  
## Module: AI Product Recommendation

## 1. Thiết kế luồng nghiệp vụ

### 1.1. Mô tả nghiệp vụ

Module **AI Product Recommendation** có nhiệm vụ tự động gợi ý các sản phẩm liên quan khi người dùng đang xem một sản phẩm cụ thể. Việc gợi ý dựa trên:

- Sản phẩm hiện tại người dùng đang xem.
- Lịch sử mua hàng của người dùng.
- Danh mục, thương hiệu, giá bán và đặc điểm của sản phẩm.
- Các sản phẩm tương tự hoặc thường được mua cùng.

### 1.2. Input

- `userId`: ID của người dùng đang đăng nhập.
- `productId`: ID của sản phẩm đang được xem.
- `purchaseHistory`: lịch sử mua hàng của người dùng.
- `productInfo`: thông tin sản phẩm hiện tại.
- `productCatalog`: danh sách sản phẩm có thể được gợi ý.
- `limit`: số lượng sản phẩm gợi ý cần trả về.

### 1.3. Output

- Danh sách sản phẩm được gợi ý, gồm:
  - `recommendedProductId`
  - `productName`
  - `category`
  - `price`
  - `imageUrl`
  - `recommendationReason`
  - `score`

### 1.4. Sơ đồ ASCII luồng xử lý

```text
User xem Product Detail Page
            |
            v
Frontend gửi request:
GET /api/v1/products/{productId}/recommendations
            |
            v
Recommendation Controller nhận userId + productId
            |
            v
Recommendation Service kiểm tra:
- user có tồn tại không
- product có tồn tại không
- product có đang Active không
            |
            v
Lấy lịch sử mua hàng của user
            |
            v
Phân tích sản phẩm hiện tại + lịch sử mua hàng
            |
            v
Tính điểm gợi ý cho sản phẩm liên quan
            |
            v
Lọc sản phẩm không hợp lệ:
- sản phẩm đã bị xóa
- sản phẩm Inactive
- sản phẩm hết hàng
- sản phẩm đang xem
            |
            v
Sắp xếp theo recommendation score
            |
            v
Trả về danh sách sản phẩm gợi ý

```

## 2. Workflow / Mega-Prompt đã sử dụng

```text
Act as a Senior System Analyst.

Mục tiêu:
Hãy biến yêu cầu thô sau thành tài liệu SRS hoàn chỉnh cho module AI Product Recommendation của Shop AI:

"Khách hàng muốn hệ thống tự động gợi ý các sản phẩm liên quan khi người dùng đang xem một món đồ, dựa trên lịch sử mua hàng của họ."

Ngữ cảnh:
Shop AI là hệ thống thương mại điện tử. Người dùng có thể xem chi tiết sản phẩm, mua hàng và có lịch sử mua hàng. Module mới cần gợi ý sản phẩm liên quan trên trang chi tiết sản phẩm. Hệ thống backend là Guai-api.

Yêu cầu phân tích:
1. Tự định nghĩa nghiệp vụ cho module AI Product Recommendation.
2. Xác định rõ Input và Output.
3. Viết Use Case Description.
4. Viết Functional Requirements.
5. Viết Data Dictionary sơ bộ.
6. Bổ sung các trường hợp ngoại lệ và edge cases.
7. Đảm bảo tài liệu có thể dùng cho Dev triển khai API.

Ràng buộc:
- Không viết chung chung.
- Mỗi Functional Requirement phải có mã định danh riêng.
- Phải xử lý các trường hợp:
  - Người dùng chưa có lịch sử mua hàng.
  - ProductId không tồn tại.
  - Sản phẩm hiện tại đã bị Inactive hoặc Deleted.
  - Không tìm được sản phẩm phù hợp để gợi ý.
  - Người dùng chưa đăng nhập.
- Không được gợi ý chính sản phẩm người dùng đang xem.
- Không được gợi ý sản phẩm hết hàng hoặc không còn bán.
- Kết quả gợi ý phải được sắp xếp theo điểm liên quan giảm dần.

Định dạng đầu ra:
1. Module Overview
2. Use Case Description
3. Business Flow
4. Functional Requirements
5. Exception Handling
6. Data Dictionary
7. Acceptance Criteria
```

## 3. Minh chứng chạy thực tế - Text Log cuộc hội thoại

```text
User:
Act as a Senior System Analyst.

Mục tiêu:
Hãy biến yêu cầu thô sau thành tài liệu SRS hoàn chỉnh cho module AI Product Recommendation của Shop AI:

"Khách hàng muốn hệ thống tự động gợi ý các sản phẩm liên quan khi người dùng đang xem một món đồ, dựa trên lịch sử mua hàng của họ."

Ngữ cảnh:
Shop AI là hệ thống thương mại điện tử. Người dùng có thể xem chi tiết sản phẩm, mua hàng và có lịch sử mua hàng. Module mới cần gợi ý sản phẩm liên quan trên trang chi tiết sản phẩm. Hệ thống backend là Guai-api.

Yêu cầu phân tích:
1. Tự định nghĩa nghiệp vụ cho module AI Product Recommendation.
2. Xác định rõ Input và Output.
3. Viết Use Case Description.
4. Viết Functional Requirements.
5. Viết Data Dictionary sơ bộ.
6. Bổ sung các trường hợp ngoại lệ và edge cases.
7. Đảm bảo tài liệu có thể dùng cho Dev triển khai API.

Ràng buộc:
- Không viết chung chung.
- Mỗi Functional Requirement phải có mã định danh riêng.
- Phải xử lý các trường hợp:
  - Người dùng chưa có lịch sử mua hàng.
  - ProductId không tồn tại.
  - Sản phẩm hiện tại đã bị Inactive hoặc Deleted.
  - Không tìm được sản phẩm phù hợp để gợi ý.
  - Người dùng chưa đăng nhập.
- Không được gợi ý chính sản phẩm người dùng đang xem.
- Không được gợi ý sản phẩm hết hàng hoặc không còn bán.
- Kết quả gợi ý phải được sắp xếp theo điểm liên quan giảm dần.

Định dạng đầu ra:
1. Module Overview
2. Use Case Description
3. Business Flow
4. Functional Requirements
5. Exception Handling
6. Data Dictionary
7. Acceptance Criteria

Assistant:
# SRS - AI Product Recommendation Module

## 1. Module Overview

Module AI Product Recommendation cho phép hệ thống Shop AI tự động gợi ý các sản phẩm liên quan khi người dùng đang xem chi tiết một sản phẩm. Hệ thống sử dụng thông tin sản phẩm hiện tại, lịch sử mua hàng của người dùng và dữ liệu danh mục sản phẩm để tạo danh sách gợi ý phù hợp.

Mục tiêu của module là tăng khả năng khám phá sản phẩm, cải thiện trải nghiệm mua sắm và hỗ trợ tăng tỷ lệ chuyển đổi.

## 2. Use Case Description

### Use Case ID
UC-REC-001

### Use Case Name
View AI Product Recommendations

### Actor
- Người dùng đã đăng nhập
- Người dùng chưa đăng nhập

### Trigger
Người dùng mở trang chi tiết sản phẩm.

### Pre-conditions
- Sản phẩm đang xem phải tồn tại trong hệ thống.
- Sản phẩm phải ở trạng thái Active.
- Hệ thống có danh sách sản phẩm có thể bán.
- Nếu người dùng đã đăng nhập, hệ thống có thể truy xuất lịch sử mua hàng của người dùng.

### Main Flow
1. Người dùng mở trang chi tiết sản phẩm.
2. Frontend gửi request đến API lấy danh sách sản phẩm gợi ý.
3. Backend nhận `productId` và thông tin `userId` nếu người dùng đã đăng nhập.
4. Hệ thống kiểm tra sản phẩm hiện tại có tồn tại không.
5. Hệ thống kiểm tra trạng thái sản phẩm hiện tại.
6. Hệ thống lấy lịch sử mua hàng của người dùng nếu có.
7. Hệ thống phân tích sản phẩm hiện tại, danh mục, thương hiệu, mức giá và lịch sử mua hàng.
8. Hệ thống tính điểm liên quan cho các sản phẩm có thể gợi ý.
9. Hệ thống loại bỏ sản phẩm không hợp lệ.
10. Hệ thống sắp xếp danh sách theo điểm liên quan giảm dần.
11. Hệ thống trả về danh sách sản phẩm gợi ý cho frontend.

### Post-conditions
- Danh sách sản phẩm gợi ý được hiển thị trên trang chi tiết sản phẩm.
- Không có thay đổi dữ liệu đơn hàng hoặc giỏ hàng.

## 3. Business Flow

```text
Product Detail Page
        |
        v
Request recommendations by productId
        |
        v
Validate product existence and status
        |
        v
Check user login status
        |
        v
Retrieve purchase history if available
        |
        v
Calculate recommendation score
        |
        v
Filter invalid products
        |
        v
Sort by score descending
        |
        v
Return recommendation list
```
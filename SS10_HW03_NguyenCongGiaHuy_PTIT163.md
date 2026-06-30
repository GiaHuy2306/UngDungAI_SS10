## Bài 3: Đọc Hiểu Code & Cập Nhật Đặc Tả Hệ Thống

### Prompt đã thiết kế
```
Act as a System Analyst.

Mục tiêu:
Hãy review logic xử lý thêm sản phẩm vào giỏ hàng trong module Cart của Guai-api và cập nhật đặc tả SRS tương ứng.

Ngữ cảnh:
Hệ thống đã index mã nguồn Guai-api. Trọng tâm phân tích là phương thức `addToCart(Long userId, Long productId, int quantity)` trong `CartService.java`. Phương thức hiện tại thực hiện:
- Tìm sản phẩm theo `productId`.
- Nếu sản phẩm không tồn tại thì throw `ResourceNotFoundException`.
- Tìm `CartItem` theo `userId` và `productId`.
- Nếu item đã tồn tại thì cộng thêm `quantity`.
- Nếu item chưa tồn tại thì tạo mới `CartItem`.
- Lưu item vào `cartRepository`.

Yêu cầu phân tích:
1. Chỉ ra các lỗ hổng logic trong code hiện tại, đặc biệt:
   - `quantity` có thể bằng 0 hoặc là số âm.
   - Tổng số lượng sau khi cộng thêm có thể vượt quá tồn kho hiện có của sản phẩm.
   - Code chưa kiểm tra trạng thái sản phẩm nếu hệ thống có trạng thái như Active/Inactive/Deleted.
   - Code chưa xử lý rõ trường hợp cart item hiện tại đã tồn tại nhưng tổng quantity mới không hợp lệ.
2. Đề xuất danh sách Business Rules theo chuẩn SRS để Dev có căn cứ vá lỗi.
3. Business Rules phải rõ ràng, có mã định danh, điều kiện áp dụng, mô tả quy tắc, xử lý khi vi phạm và mức độ ưu tiên.
4. Không viết chung chung. Hãy liên hệ trực tiếp với logic `addToCart`.

Định dạng đầu ra:
- Phần 1: Logic Gaps Found
- Phần 2: Business Rules for Cart Add-to-Cart
  - Rule ID
  - Rule Name
  - Description
  - Applies To
  - Validation Logic
  - Exception Handling
  - Priority
```

### Business Rules do AI trả về
```
# Business Rules for Cart Add-to-Cart

## BR-CART-001: Product Must Exist

- **Description:** Người dùng chỉ được thêm sản phẩm tồn tại trong hệ thống vào giỏ hàng.
- **Applies To:** `CartService.addToCart(userId, productId, quantity)`
- **Validation Logic:**
  - Hệ thống phải kiểm tra `productId` có tồn tại trong `ProductRepository`.
  - Nếu không tìm thấy sản phẩm, quá trình thêm vào giỏ hàng phải bị dừng.
- **Exception Handling:**
  - Throw `ResourceNotFoundException("Product not found")`.
  - Không tạo mới hoặc cập nhật `CartItem`.
- **Priority:** High

## BR-CART-002: Quantity Must Be Positive

- **Description:** Số lượng sản phẩm thêm vào giỏ hàng phải là số nguyên dương.
- **Applies To:** `quantity` trong hàm `addToCart`.
- **Validation Logic:**
  - Hệ thống phải kiểm tra `quantity > 0` trước khi tạo mới hoặc cập nhật `CartItem`.
  - Không chấp nhận `quantity = 0`.
  - Không chấp nhận `quantity < 0`.
- **Exception Handling:**
  - Nếu `quantity <= 0`, hệ thống phải từ chối request.
  - Trả về lỗi nghiệp vụ như `InvalidQuantityException` hoặc HTTP 400 Bad Request.
  - Không lưu thay đổi vào `cartRepository`.
- **Priority:** Critical

## BR-CART-003: Total Cart Quantity Must Not Exceed Inventory

- **Description:** Tổng số lượng sản phẩm trong giỏ hàng sau khi thêm không được vượt quá tồn kho hiện có của sản phẩm.
- **Applies To:** Trường hợp `CartItem` đã tồn tại hoặc được tạo mới.
- **Validation Logic:**
  - Nếu `CartItem` đã tồn tại, hệ thống tính:
    - `newQuantity = item.getQuantity() + quantity`
  - Nếu `CartItem` chưa tồn tại, hệ thống tính:
    - `newQuantity = quantity`
  - Hệ thống phải so sánh `newQuantity` với số lượng tồn kho của sản phẩm.
  - Điều kiện hợp lệ:
    - `newQuantity <= product.getInventoryQuantity()` hoặc field tồn kho tương ứng trong entity `Product`.
- **Exception Handling:**
  - Nếu `newQuantity` vượt quá tồn kho, hệ thống phải từ chối thao tác.
  - Trả về lỗi nghiệp vụ như `InsufficientInventoryException` hoặc HTTP 400 Bad Request.
  - Không lưu `CartItem` mới và không cập nhật `CartItem` hiện tại.
- **Priority:** Critical

## BR-CART-004: Existing Cart Item Must Be Updated Safely

- **Description:** Khi sản phẩm đã tồn tại trong giỏ hàng, hệ thống chỉ được cập nhật số lượng sau khi toàn bộ kiểm tra hợp lệ đã hoàn tất.
- **Applies To:** Trường hợp `cartRepository.findByUserIdAndProductId(userId, productId)` trả về `CartItem`.
- **Validation Logic:**
  - Hệ thống không được gọi `item.setQuantity(item.getQuantity() + quantity)` ngay lập tức nếu chưa kiểm tra `quantity` và tồn kho.
  - Hệ thống phải tính `newQuantity` trước.
  - Chỉ cập nhật `item.setQuantity(newQuantity)` khi:
    - `quantity > 0`
    - `newQuantity <= inventory`
- **Exception Handling:**
  - Nếu kiểm tra thất bại, giữ nguyên số lượng cũ trong giỏ hàng.
  - Không gọi `cartRepository.save(item)`.
- **Priority:** High

## BR-CART-005: New Cart Item Must Be Created Only With Valid Quantity

- **Description:** CartItem mới chỉ được tạo khi số lượng yêu cầu hợp lệ và không vượt quá tồn kho.
- **Applies To:** Trường hợp sản phẩm chưa tồn tại trong giỏ hàng của người dùng.
- **Validation Logic:**
  - Hệ thống chỉ tạo `new CartItem(userId, productId, quantity)` nếu:
    - `quantity > 0`
    - `quantity <= product.getInventoryQuantity()`
- **Exception Handling:**
  - Nếu quantity không hợp lệ hoặc vượt tồn kho, hệ thống không tạo `CartItem`.
  - Trả về lỗi nghiệp vụ phù hợp.
- **Priority:** High

## BR-CART-006: Product Must Be Available for Sale

- **Description:** Người dùng chỉ được thêm sản phẩm đang ở trạng thái có thể bán vào giỏ hàng.
- **Applies To:** Sản phẩm lấy từ `ProductRepository`.
- **Validation Logic:**
  - Nếu entity `Product` có trạng thái như `Active`, `Inactive`, `Deleted` hoặc `OutOfStock`, hệ thống phải kiểm tra trạng thái trước khi thêm vào giỏ.
  - Chỉ sản phẩm ở trạng thái `Active` hoặc `Available` mới được phép thêm vào giỏ hàng.
- **Exception Handling:**
  - Nếu sản phẩm không khả dụng, hệ thống từ chối thao tác.
  - Trả về lỗi nghiệp vụ như `ProductUnavailableException` hoặc HTTP 400 Bad Request.
  - Không tạo hoặc cập nhật `CartItem`.
- **Priority:** Medium

## BR-CART-007: Cart Update Must Be Atomic

- **Description:** Việc kiểm tra hợp lệ và lưu thay đổi vào giỏ hàng phải được xử lý như một thao tác nhất quán.
- **Applies To:** Toàn bộ quy trình thêm sản phẩm vào giỏ hàng.
- **Validation Logic:**
  - Hệ thống phải hoàn tất tất cả kiểm tra trước khi lưu dữ liệu.
  - Nếu bất kỳ kiểm tra nào thất bại, không được lưu thay đổi một phần.
- **Exception Handling:**
  - Rollback transaction nếu thao tác lưu thất bại hoặc phát sinh exception.
  - Không để giỏ hàng rơi vào trạng thái sai lệch dữ liệu.
- **Priority:** High

## BR-CART-008: Error Response Must Be Clear and Consistent

- **Description:** Các lỗi khi thêm sản phẩm vào giỏ hàng phải được trả về rõ ràng và nhất quán cho client.
- **Applies To:** API thêm sản phẩm vào giỏ hàng.
- **Validation Logic:**
  - Lỗi sản phẩm không tồn tại: trả về thông báo phù hợp.
  - Lỗi quantity không hợp lệ: trả về thông báo quantity phải lớn hơn 0.
  - Lỗi vượt quá tồn kho: trả về thông báo không đủ hàng trong kho.
  - Lỗi sản phẩm không khả dụng: trả về thông báo sản phẩm không thể thêm vào giỏ hàng.
- **Exception Handling:**
  - Sử dụng mã lỗi HTTP phù hợp, ví dụ:
    - 404 Not Found cho sản phẩm không tồn tại.
    - 400 Bad Request cho quantity không hợp lệ hoặc vượt tồn kho.
- **Priority:** Medium
```
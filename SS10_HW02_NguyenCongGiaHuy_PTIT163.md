## Bài 2: Tối Ưu Hóa Prompt Đặc Tả Yêu Cầu Chức Năng

### Prompt tối ưu

```
Act as a Senior System Analyst.

Mục tiêu:
Hãy viết tài liệu Functional Requirements cho module Authentication của dự án Shop AI, tập trung vào chức năng Đăng nhập.

Ngữ cảnh:
Hệ thống Shop AI sử dụng JWT để xác thực phiên đăng nhập và RBAC để phân quyền người dùng theo vai trò. Chức năng đăng nhập cần kiểm tra thông tin tài khoản, trạng thái tài khoản, mật khẩu, sinh JWT token sau khi đăng nhập thành công và đảm bảo người dùng chỉ được truy cập chức năng phù hợp với vai trò được cấp.

Ràng buộc:
- Tài liệu phải viết theo phong cách SRS rõ ràng, chặt chẽ.
- Không viết mô tả chung chung; mỗi yêu cầu phải có mã định danh riêng.
- Phải mô tả rõ input, xử lý hệ thống, output và điều kiện lỗi.
- Phải đặc tả rõ cách xử lý các trường hợp ngoại lệ sau:
  1. Người dùng nhập sai mật khẩu quá 5 lần.
  2. Token JWT hết hạn trong lúc phiên làm việc đang diễn ra.
  3. Tài khoản người dùng ở trạng thái Inactive nhưng cố tình đăng nhập.
- Không tự giả định các chức năng ngoài phạm vi Authentication nếu không cần thiết.
- Nếu có giả định nghiệp vụ, hãy ghi rõ trong phần Assumptions.

Định dạng đầu ra:
Hãy trình bày tài liệu theo cấu trúc sau:
1. Overview
2. Assumptions
3. Functional Requirements
   - Requirement ID
   - Requirement Name
   - Description
   - Input
   - Processing Logic
   - Output
   - Exception Handling
   - Priority
4. RBAC Rules
5. JWT Handling Rules
6. Edge Cases
7. Acceptance Criteria
```

### Tài liệu Functional Requirements do AI sinh ra

```
# Functional Requirements - Authentication Module

## 1. Overview

Module Authentication của Shop AI chịu trách nhiệm xác thực người dùng, kiểm tra trạng thái tài khoản, cấp JWT token sau khi đăng nhập thành công và hỗ trợ phân quyền truy cập dựa trên RBAC.

## 2. Assumptions

- Người dùng đăng nhập bằng email/username và mật khẩu.
- Hệ thống lưu mật khẩu dưới dạng đã mã hóa.
- JWT token được cấp sau khi người dùng đăng nhập thành công.
- RBAC được áp dụng dựa trên vai trò của người dùng, ví dụ: Admin, Staff, Customer.
- Tài khoản có thể ở các trạng thái: Active, Inactive, Locked.

## 3. Functional Requirements

### FR-AUTH-001: User Login

- **Description:** Hệ thống cho phép người dùng đăng nhập bằng email/username và mật khẩu hợp lệ.
- **Input:**
  - Email hoặc username
  - Password
- **Processing Logic:**
  - Hệ thống kiểm tra tài khoản có tồn tại hay không.
  - Hệ thống kiểm tra trạng thái tài khoản.
  - Hệ thống xác thực mật khẩu.
  - Nếu hợp lệ, hệ thống sinh JWT token.
  - Hệ thống gắn thông tin vai trò người dùng vào token hoặc payload phục vụ RBAC.
- **Output:**
  - Đăng nhập thành công.
  - Trả về JWT token và thông tin cơ bản của người dùng.
- **Exception Handling:**
  - Nếu tài khoản không tồn tại, trả về thông báo đăng nhập thất bại.
  - Nếu mật khẩu sai, trả về thông báo đăng nhập thất bại.
  - Nếu tài khoản không Active, từ chối đăng nhập.
- **Priority:** High

### FR-AUTH-002: Password Failure Tracking

- **Description:** Hệ thống phải ghi nhận số lần nhập sai mật khẩu của từng tài khoản.
- **Input:**
  - Email hoặc username
  - Password không hợp lệ
- **Processing Logic:**
  - Khi người dùng nhập sai mật khẩu, hệ thống tăng bộ đếm số lần đăng nhập thất bại.
  - Nếu số lần sai mật khẩu chưa vượt quá 5, hệ thống cho phép người dùng thử lại.
  - Nếu số lần sai mật khẩu vượt quá 5, hệ thống khóa tài khoản hoặc chuyển tài khoản sang trạng thái Locked theo chính sách bảo mật.
- **Output:**
  - Thông báo đăng nhập thất bại.
  - Thông báo tài khoản bị khóa nếu vượt quá giới hạn.
- **Exception Handling:**
  - Không tiết lộ chi tiết rằng email/username có tồn tại hay không.
  - Không cho phép tiếp tục đăng nhập khi tài khoản đã bị khóa.
- **Priority:** High

### FR-AUTH-003: Inactive Account Login Prevention

- **Description:** Hệ thống phải ngăn tài khoản ở trạng thái Inactive thực hiện đăng nhập.
- **Input:**
  - Email hoặc username
  - Password
- **Processing Logic:**
  - Sau khi xác định tài khoản tồn tại, hệ thống kiểm tra trạng thái tài khoản.
  - Nếu tài khoản có trạng thái Inactive, hệ thống từ chối đăng nhập ngay cả khi mật khẩu đúng.
  - Hệ thống không sinh JWT token cho tài khoản Inactive.
- **Output:**
  - Đăng nhập thất bại.
  - Trả về thông báo tài khoản không được phép đăng nhập.
- **Exception Handling:**
  - Không cấp quyền truy cập cho tài khoản Inactive trong mọi trường hợp.
  - Ghi log sự kiện đăng nhập bị từ chối để phục vụ kiểm tra bảo mật.
- **Priority:** High

### FR-AUTH-004: JWT Token Generation

- **Description:** Hệ thống phải sinh JWT token sau khi người dùng đăng nhập thành công.
- **Input:**
  - Thông tin người dùng đã xác thực thành công
  - Vai trò người dùng
- **Processing Logic:**
  - Hệ thống tạo JWT token có thời hạn sử dụng.
  - Token phải chứa thông tin định danh người dùng và vai trò phù hợp.
  - Token phải được ký bằng secret key hoặc private key của hệ thống.
- **Output:**
  - JWT token hợp lệ.
  - Thời gian hết hạn token.
- **Exception Handling:**
  - Nếu không thể sinh token, hệ thống trả về lỗi xác thực và không cho phép truy cập.
- **Priority:** High

### FR-AUTH-005: JWT Expiration Handling

- **Description:** Hệ thống phải xử lý token JWT hết hạn trong lúc phiên làm việc đang diễn ra.
- **Input:**
  - Request có JWT token hết hạn
- **Processing Logic:**
  - Hệ thống kiểm tra thời hạn token trong mỗi request cần xác thực.
  - Nếu token đã hết hạn, hệ thống từ chối request.
  - Hệ thống yêu cầu người dùng đăng nhập lại hoặc sử dụng cơ chế refresh token nếu hệ thống có hỗ trợ.
- **Output:**
  - Request bị từ chối.
  - Trả về lỗi xác thực do token expired.
- **Exception Handling:**
  - Không tiếp tục xử lý nghiệp vụ khi token đã hết hạn.
  - Không cấp quyền truy cập vào tài nguyên bảo vệ bằng token expired.
  - Có thể trả về HTTP 401 Unauthorized.
- **Priority:** High

### FR-AUTH-006: RBAC Authorization

- **Description:** Hệ thống phải kiểm soát quyền truy cập dựa trên vai trò người dùng.
- **Input:**
  - JWT token hợp lệ
  - Vai trò người dùng
  - Request truy cập tài nguyên
- **Processing Logic:**
  - Hệ thống trích xuất vai trò từ JWT token hoặc từ dữ liệu người dùng.
  - Hệ thống kiểm tra vai trò có quyền truy cập tài nguyên được yêu cầu hay không.
  - Nếu vai trò hợp lệ, request được tiếp tục xử lý.
  - Nếu vai trò không hợp lệ, hệ thống từ chối truy cập.
- **Output:**
  - Cho phép truy cập nếu đúng quyền.
  - Từ chối truy cập nếu sai quyền.
- **Exception Handling:**
  - Có thể trả về HTTP 403 Forbidden khi người dùng đã xác thực nhưng không có quyền.
- **Priority:** High

## 4. RBAC Rules

- Người dùng chỉ được truy cập các chức năng tương ứng với vai trò được gán.
- Admin có quyền truy cập chức năng quản trị hệ thống.
- Staff có quyền truy cập các chức năng vận hành theo phân quyền.
- Customer có quyền truy cập các chức năng mua hàng và quản lý tài khoản cá nhân.
- Nếu người dùng không có vai trò phù hợp, hệ thống phải từ chối truy cập.

## 5. JWT Handling Rules

- JWT token chỉ được cấp sau khi đăng nhập thành công.
- JWT token phải có thời gian hết hạn.
- Mọi API yêu cầu xác thực phải kiểm tra tính hợp lệ của JWT token.
- Token hết hạn phải bị từ chối.
- Token sai chữ ký, sai định dạng hoặc không tồn tại phải bị từ chối.
- Hệ thống không xử lý nghiệp vụ chính nếu JWT không hợp lệ.

## 6. Edge Cases

### EC-001: Người dùng nhập sai mật khẩu quá 5 lần

- Hệ thống ghi nhận từng lần nhập sai mật khẩu.
- Khi số lần nhập sai vượt quá 5, hệ thống khóa tài khoản hoặc chuyển trạng thái sang Locked.
- Hệ thống từ chối các lần đăng nhập tiếp theo cho đến khi tài khoản được mở khóa.
- Hệ thống nên ghi log sự kiện để phục vụ kiểm tra bảo mật.

### EC-002: JWT token hết hạn trong lúc phiên làm việc

- Khi người dùng gửi request với token đã hết hạn, hệ thống trả về lỗi xác thực.
- Hệ thống không cho phép tiếp tục phiên làm việc bằng token expired.
- Người dùng phải đăng nhập lại hoặc dùng refresh token nếu hệ thống hỗ trợ.
- Các thao tác nghiệp vụ đang yêu cầu token hợp lệ phải bị dừng.

### EC-003: Tài khoản Inactive cố tình đăng nhập

- Hệ thống kiểm tra trạng thái tài khoản trước khi cấp JWT.
- Nếu tài khoản ở trạng thái Inactive, hệ thống từ chối đăng nhập.
- Hệ thống không sinh JWT token cho tài khoản Inactive.
- Sự kiện đăng nhập bị từ chối nên được ghi log.

## 7. Acceptance Criteria

- Người dùng Active nhập đúng thông tin đăng nhập thì nhận được JWT token hợp lệ.
- Người dùng nhập sai mật khẩu thì không được đăng nhập.
- Người dùng nhập sai mật khẩu quá 5 lần thì tài khoản bị khóa hoặc bị chặn đăng nhập.
- Tài khoản Inactive không thể đăng nhập dù mật khẩu đúng.
- JWT token hết hạn không thể dùng để truy cập API bảo vệ.
- Người dùng không có vai trò phù hợp sẽ bị từ chối truy cập.
- Hệ thống phân biệt đúng lỗi 401 Unauthorized và 403 Forbidden.
```
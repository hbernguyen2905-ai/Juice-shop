# Black-box Testing

## 1. Objective

Đánh giá behavior quan sát được từ phía người dùng hoặc API consumer đối với các chức năng quan trọng của OWASP Juice Shop. Expected result được xác định từ HTTP contract, user flow và behavior thực tế; implementation chỉ được dùng để xác định endpoint, test environment và dữ liệu kiểm thử.

## 2. Scope

Phạm vi gồm Register, Login, Logout, Search, Product, Shopping Cart, Checkout, Payment, User Profile, Authentication, Authorization và API input validation.

Các security challenge cố ý của Juice Shop được ghi nhận riêng là `Intentional vulnerability`, không tự động xem là defect mới.

## 3. Testing Environment

- Repository: OWASP Juice Shop, version `20.2.0`.
- OS: Windows.
- API test environment: test app của project với SQLite in-memory.
- API runner: Node.js built-in test runner + Supertest + `tsx`.
- API command đã chạy: `npm run test:api`.
- UI/Cypress: chưa khởi động server riêng trong Phase 2; các case UI tương ứng được ghi `NOT EXECUTED`.
- Không sửa source code và không cài dependency.

## 4. Testing Techniques

### 4.1 Equivalence Partitioning

Các lớp input được sử dụng:

- Valid: email/password, product ID và card data hợp lệ.
- Invalid: sai credential, invalid ID, invalid card number, invalid role.
- Empty/missing: thiếu credential, thiếu query, thiếu authentication.
- Unexpected format: malformed authorization, malformed token, invalid content type và security payload.

### 4.2 Boundary Value Analysis

Các boundary đã xác định từ behavior thực tế:

- Search query được giới hạn ở 200 ký tự theo route behavior.
- Card `expMonth` ngoài miền hợp lệ bị từ chối; test đã xác nhận `13` bị reject.
- Card `expYear` quá khứ bị từ chối; test đã xác nhận `2015` bị reject.
- Product/basket/card ID không tồn tại được kiểm thử bằng ID lớn.
- Basket quantity bằng 0, âm và rất lớn được kiểm thử trong suite basket hiện có.

Không thêm giới hạn email/password/file size khi chưa có behavior hoặc test evidence tương ứng.

### 4.3 Decision Table

| Authentication | Role | Resource | Expected/Observed behavior |
|---|---|---|---|
| No | Any | Protected basket/card/profile | Denied, thường `401` |
| Yes | Customer | Own basket/order history | Allowed |
| Yes | Customer | All accounting orders / delivery status | Denied |
| Yes | Admin | All accounting orders / delivery status | Denied theo test hiện có |
| Yes | Accounting | All orders / delivery status | Allowed |
| Yes | Any authenticated | Product DELETE | Denied |
| Yes | Any authenticated | Product PUT | Intentional product tampering behavior được test xác nhận |

### 4.4 State Transition

State được kiểm thử hoặc xác định từ test flow:

```text
Logged out -> Login success -> Token/session available -> Protected request
Logged out -> Login failure -> No authenticated session
Authenticated -> Add basket item -> Checkout -> Order confirmation
Authenticated -> Logout -> Client token/session cleared -> Protected UI route unavailable
Guest -> Add guest basket item -> Login -> Guest basket merge
```

Logout UI state chưa được chạy độc lập trong Phase 2.

### 4.5 Error Guessing

Đã chọn các lỗi đại diện có giá trị cao: empty credential, duplicate email, invalid role, missing/invalid token, non-existing ID, invalid card fields, negative quantity, invalid coupon, malformed search input, unauthorized method và forged authorization input.

## 5. Test Cases

Actual Result và Status dưới đây được ghi theo test tương ứng đã chạy trong `test/api` hoặc được đánh dấu `NOT EXECUTED` nếu chưa có execution trong Phase 2.

| ID | Technique | Feature | Test Case | Precondition | Input / Steps | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|---|---|---|
| BB-001 | EP | Register | Đăng ký user hợp lệ | API test app chạy | `POST /api/Users` với email/password hợp lệ | `201`, user được tạo, response không trả password | `201`, id và timestamps có, password không trả | PASS |
| BB-002 | EP | Register | Đăng ký email đã tồn tại | User cùng email đã có | Gửi registration lần hai | Request bị từ chối với lỗi validation/duplicate | `400` với duplicate constraint | PASS |
| BB-003 | EP | Register | Đăng ký role không hợp lệ | API công khai | `role=accountinguser` | Request bị từ chối | `400`, validation `isIn` failed | PASS |
| BB-004 | EG | Register | Registration với input whitespace/blank | API test app chạy | Email/password là whitespace | Behavior thực tế vẫn tạo user trong test hiện tại | `201`; intentional/known behavior, không xem là defect mới trong Phase 2 | PASS (behavior recorded) |
| BB-005 | EP | Login | Login với credential hợp lệ | User có sẵn | `POST /rest/user/login` | `200`, trả token và basket id | `200`, token/bid/umail có | PASS |
| BB-006 | EP | Login | Sai email hoặc password | API test app chạy | Credential không hợp lệ | `401` | `401` | PASS |
| BB-007 | EG | Login | Thiếu email/password | API test app chạy | Body thiếu credential | `401` | `401` | PASS |
| BB-008 | EP | Login | Tài khoản yêu cầu 2FA | User có TOTP | Login rồi gửi TOTP hợp lệ | Login ban đầu yêu cầu temporary token; verify trả authentication | `totp_token_required`, flow 2FA pass | PASS |
| BB-009 | ST | Logout | Logout user đang đăng nhập qua UI | User logged in | Click logout | Token/cookie/session client bị xóa và điều hướng về home | Chưa chạy UI/Cypress | NOT EXECUTED |
| BB-010 | ST | Logout | Truy cập protected resource sau logout | Đã logout | Gọi protected resource không còn token | Request bị từ chối | Chưa chạy logout flow độc lập | NOT EXECUTED |
| BB-011 | EP | Search | Search có kết quả | API public | `q=o-saft` | `200`, trả product match | `200`, một product match | PASS |
| BB-012 | EP | Search | Search không có kết quả | API public | Query không tồn tại | `200`, data rỗng | `200`, data length `0` | PASS |
| BB-013 | EG | Search | Search không query hoặc query rỗng | API public | `/rest/products/search` và `q=` | Behavior nhất quán với search all | `200`, số lượng bằng product list | PASS |
| BB-014 | EG | Search | SQL injection input | Local test app | Query phá SQL và UNION payload | Phân loại theo challenge, không gọi là defect mới | Suite xác nhận SQL error/extraction behavior cố ý | PASS (intentional vulnerability) |
| BB-015 | EP | Product | Lấy danh sách product | API public | `GET /api/Products` | `200`, array product có field cần thiết | `200`, schema fields hợp lệ | PASS |
| BB-016 | EP | Product | Lấy product tồn tại/không tồn tại | API public | ID `1` và `4711` | Existing `200`; missing `404` | Đúng behavior trên cả hai ID | PASS |
| BB-017 | EG | Product | Thử sửa product không authentication | API public | `PUT /api/Products/:id` | Theo authorization policy thông thường bị từ chối | Suite xác nhận PUT tampering có thể thành công | PASS (intentional vulnerability) |
| BB-018 | EP | Basket | Đọc basket không authentication | Không có token | `GET /rest/basket/1` | `401` | `401` | PASS |
| BB-019 | EP | Basket | Thêm/cập nhật item và quantity bất thường | User authenticated | Add item; quantity âm/large | Response phản ánh behavior validation/challenge | Suite basket chạy thành công các case đại diện | PASS |
| BB-020 | EG | Basket | Truy cập basket ID khác | User authenticated | Đọc basket của user khác | Ghi nhận đúng authorization behavior của app | Suite xác nhận access behavior intentional của challenge | PASS (intentional vulnerability) |
| BB-021 | ST | Checkout | Checkout basket hợp lệ | User authenticated, basket hợp lệ | `POST /rest/basket/:id/checkout` | `200`, order confirmation | `200`, order confirmation có | PASS |
| BB-022 | EG | Checkout | Checkout basket không tồn tại | User authenticated | Checkout ID `42` | Request lỗi rõ ràng | `500`, báo basket không tồn tại | PASS (observed behavior) |
| BB-023 | EP | Payment | Tạo và đọc payment card hợp lệ | User authenticated | Card hợp lệ, sau đó GET cards | `201`; số thẻ trả masked | `201`; `************4321` | PASS |
| BB-024 | BVA | Payment | Card number/month/year không hợp lệ | User authenticated | Card number invalid, month `13`, year `2015` | `400` | Tất cả case trả `400` | PASS |
| BB-025 | EP | Profile | Đọc/cập nhật profile | User authenticated | `GET /profile`, `POST /profile` | Authenticated allowed; anonymous denied | Authenticated pass; anonymous bị từ chối | PASS |
| BB-026 | EP | Authentication | `whoami` không token, token invalid, expired | API public | Missing/malformed/expired Authorization | Không trả authenticated user | `200` với user rỗng | PASS |
| BB-027 | DT | Authorization | Customer/admin/accounting truy cập tài nguyên phân quyền | Các role test data có sẵn | Order list và delivery status | Customer/admin denied; accounting allowed | Đúng matrix đã xác nhận | PASS |
| BB-028 | EG | API | Card endpoint không authentication | Không có token | GET/POST/PUT/DELETE card | Protected method bị từ chối | Public calls `401`; invalid ID authorized `400` | PASS |

## 6. Test Execution Results

### 6.1 API / Supertest

Command:

```text
npm run test:api
```

Result:

- PASS: `530`
- FAIL: `0`
- SKIPPED: `7`
- Duration: khoảng `75.6s`
- Test runner: Node.js built-in test runner
- HTTP client: Supertest
- Database: SQLite in-memory

Các test skip được runner ghi nhận trong output hiện tại, gồm một số case phụ thuộc môi trường CI, socket behavior hoặc IP filter không thể mô phỏng trong execution này.

### 6.2 UI / Browser

Command/Action: Không chạy server UI và không chạy Cypress trong Phase 2.

- PASS: `0`
- FAIL: `0`
- NOT EXECUTED: BB-009, BB-010 và các UI journey tương ứng.

### 6.3 Cypress

Command/Action: Chưa chạy `npm run test:e2e` vì Phase 2 chưa khởi động application server riêng cho browser execution.

- PASS: `0`
- FAIL: `0`
- NOT EXECUTED: Cypress-specific execution.

### 6.4 Supertest mapping

Các case BB-001 đến BB-008, BB-011 đến BB-028 được thực hiện hoặc được bao phủ bởi các test API hiện có của project. Đây là black-box ở HTTP boundary; test không gọi trực tiếp model hoặc private function để quyết định expected result.

## 7. Defects / Findings

### Intentional vulnerabilities / expected challenge behavior

Các behavior sau được test suite xác nhận nhưng không phân loại là defect mới:

1. Search SQL injection và UNION extraction tại `/rest/products/search`.
2. Product tampering qua `PUT /api/Products/:id`.
3. Basket access behavior phục vụ basket access challenge.
4. Password exposure khi dùng `fields=id,email,password` với `whoami`.
5. XSS input được lưu trong challenge-specific product/user flows.
6. Một số forged JWT, NoSQL injection và information disclosure challenge.

### Potential defects / known issues

- Không có API test failure trong execution này: `0 fail`.
- Một số test bị skip do giới hạn môi trường hoặc ghi chú CI trong suite; đây không phải failure của application trong execution hiện tại.
- Logout chưa có API endpoint riêng và chưa được xác minh bằng browser execution trong Phase 2.
- Checkout với basket không tồn tại trả `500`; đây là behavior đã quan sát và cần được product owner xác định có phải contract mong muốn hay không.

Không đủ evidence để kết luận có `newly discovered defect` trong Phase 2.

## 8. Limitations

- Chưa chạy UI/Cypress vì chưa khởi động server browser execution trong phase này.
- Không thực hiện load test hoặc DoS test.
- Không kiểm thử mọi endpoint của Juice Shop; chỉ chọn các endpoint đại diện cho phạm vi ưu tiên.
- Một số test API hiện có kiểm tra challenge behavior, nên PASS có nghĩa là behavior khớp expectation của test/challenge, không đồng nghĩa behavior an toàn theo production security standard.
- Chưa có screenshot thực tế; danh sách bên dưới chỉ là evidence cần chụp, không phải ảnh đã tạo.

## 9. Evidence cần chụp

| Evidence ID | Test Case ID | Action | Expected | What screenshot should show | Purpose |
|---|---|---|---|---|---|
| BB-E01 | BB-001 | Register user hợp lệ trên UI | Registration thành công | Form và thông báo/điều hướng sau đăng ký | Chứng minh happy path Register |
| BB-E02 | BB-002 | Register email trùng | Validation/error message | Email trùng và response UI | Chứng minh duplicate handling |
| BB-E03 | BB-006 | Login sai credential | Login bị từ chối | Error message trên login form | Chứng minh negative Login |
| BB-E04 | BB-005 | Login hợp lệ | User vào trạng thái logged in | Navbar/user state và URL sau login | Chứng minh Authentication |
| BB-E05 | BB-011 | Search product | Kết quả phù hợp query | Query và product result | Chứng minh Search |
| BB-E06 | BB-019 | Add product vào basket | Basket tăng item/quantity | Product page và basket counter | Chứng minh Basket |
| BB-E07 | BB-021 | Checkout | Order confirmation | Summary/payment/order completion | Chứng minh Checkout |
| BB-E08 | BB-023 | Thêm payment card | Card lưu và masked | Saved payment method với 4 số cuối | Chứng minh Payment |
| BB-E09 | BB-025 | Xem/cập nhật profile | Profile hiển thị/cập nhật | Profile page và changed field | Chứng minh Profile |
| BB-E10 | BB-009 | Logout | Token/session client bị xóa | Trang sau logout và trạng thái navbar | Chứng minh Logout state transition |
| BB-E11 | BB-028 | Gọi protected API không token | `401` | API client response/status | Chứng minh Authorization |
| BB-E12 | BB-014 | Search security payload trên localhost | Intentional challenge behavior được ghi nhận | Request/response, không dùng production | Chứng minh security classification |

## 10. Conclusion

Phase 2 đã thực hiện black-box API testing bằng suite Supertest có sẵn. Kết quả thực tế là `530 PASS`, `0 FAIL`, `7 SKIPPED`. Các chức năng ưu tiên ở API boundary đã có coverage đáng kể. Các intentional vulnerabilities được phân loại riêng và không bị báo cáo như defect mới. Logout UI, browser user journey và screenshot evidence cần được bổ sung khi chạy Cypress hoặc manual browser execution ở phase phù hợp.

## Final Summary

1. Tổng số black-box test case thiết kế: `28`.
2. Số test case được thực hiện/bao phủ qua API suite: `26`.
3. PASS: `530` test executions trong `npm run test:api`.
4. FAIL: `0`.
5. NOT EXECUTED: `2` test case chính trong bảng (`BB-009`, `BB-010`) và toàn bộ Cypress execution.
6. Defect/findings mới: `0` được kết luận từ evidence hiện có.
7. Intentional vulnerabilities: SQL injection search, product tampering, basket access, password exposure, XSS/NoSQL/JWT challenge behaviors và các challenge security behavior liên quan.
8. File được tạo: [TEST_CASES_BLACKBOX.md](TEST_CASES_BLACKBOX.md).
9. File được sửa: không có source/test file nào.
10. Commands đã chạy: `npm run test:api`.
11. Screenshot/evidence cần chụp: `BB-E01` đến `BB-E12` ở mục 9.

PHASE 2 COMPLETED — READY FOR PHASE 3
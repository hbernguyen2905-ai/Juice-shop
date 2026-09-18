# Juice-shop
# README — Cài đặt Node.js và Dependency cho OWASP Juice Shop

## 1. Kiểm tra phiên bản Node.js

Trước tiên, mở **PowerShell** hoặc **Command Prompt** và kiểm tra phiên bản Node.js hiện tại bằng lệnh:

```bash
node -v
```

Phiên bản Node.js yêu cầu:

```text
v22.22.3
```

### Nếu kết quả không phải `v22.22.3`

Cần cài đặt Node.js phiên bản **22.22.3**.

Sử dụng file cài đặt:

```text
node-v22.22.3-x64.msi
```

Thực hiện các bước:

1. Mở file `node-v22.22.3-x64.msi`.
2. Thực hiện cài đặt Node.js theo hướng dẫn của trình cài đặt.
3. Sau khi cài đặt hoàn tất, **đóng PowerShell/Command Prompt hiện tại**.
4. Mở một cửa sổ PowerShell/Command Prompt mới.
5. Kiểm tra lại phiên bản Node.js:

```bash
node -v
```

Kết quả mong muốn:

```text
v22.22.3
```

> **Lưu ý:** Phải kiểm tra lại phiên bản bằng `node -v` sau khi cài đặt để đảm bảo hệ thống đang sử dụng đúng Node.js `v22.22.3`.

---

## 2. Di chuyển vào thư mục OWASP Juice Shop

Sau khi Node.js đã đúng phiên bản, mở PowerShell và di chuyển đến thư mục source code của OWASP Juice Shop.

Ví dụ:

```powershell
cd D:\juice_shop\juice-shop
```

Có thể kiểm tra thư mục hiện tại bằng:

```powershell
pwd
```

Kết quả cần trỏ đến thư mục:

```text
D:\juice_shop\juice-shop
```

---

## 3. Cài đặt các package cần thiết

Tại thư mục gốc của project, chạy:

```bash
npm install
```

Lệnh này sẽ đọc file:

```text
package.json
```

và tự động tải, cài đặt các package/dependency cần thiết cho OWASP Juice Shop.

Quá trình cài đặt có thể mất một khoảng thời gian tùy thuộc vào tốc độ mạng và cấu hình máy tính.

Khi cài đặt thành công, PowerShell sẽ không xuất hiện lỗi `npm error` nghiêm trọng và quá trình build frontend sẽ hoàn tất.

Ví dụ:

```text
Application bundle generation complete.
```

---

## 4. Kiểm tra sau khi cài đặt

Kiểm tra lại Node.js:

```bash
node -v
```

Phải có:

```text
v22.22.3
```

Sau đó kiểm tra npm:

```bash
npm -v
```

Nếu cả hai lệnh đều trả về phiên bản hợp lệ, môi trường Node.js đã được thiết lập.

---

## 5. Chạy OWASP Juice Shop

Sau khi `npm install` hoàn tất, chạy:

```bash
npm start
```

Nếu server khởi động thành công, terminal sẽ hiển thị:

```text
info: Server listening on port 3000
```

Sau đó mở trình duyệt và truy cập:

```text
http://localhost:3000
```

OWASP Juice Shop sẽ được chạy trên máy local.

---

## 6. Quy trình cài đặt tổng quát

```text
Kiểm tra Node.js
      │
      ↓
node -v
      │
      ├── v22.22.3 ────────────────┐
      │                            │
      └── Phiên bản khác           │
              ↓                    │
      Cài node-v22.22.3-x64.msi   │
              ↓                    │
      Mở PowerShell mới            │
              ↓                    │
           node -v                 │
              │                    │
              └────────────┬───────┘
                           ↓
                    v22.22.3
                           │
                           ↓
                cd D:\juice_shop\juice-shop
                           │
                           ↓
                     npm install
                           │
                           ↓
                       npm start
                           │
                           ↓
                http://localhost:3000
```

## 7. Một số lưu ý

* Không cần chạy `npm install` nếu dependency đã được cài đặt và không có thay đổi.
* Không sử dụng `npm audit fix --force` trong quá trình thiết lập ban đầu nếu chưa cần thiết, vì có thể làm thay đổi phiên bản dependency của project.
* Nếu `npm install` xuất hiện cảnh báo `npm warn`, cần phân biệt cảnh báo với lỗi `npm error`.
* Nếu `npm start` hiển thị:

```text
info: Server listening on port 3000
```

thì Juice Shop đã khởi động thành công.

# Hướng Dẫn Chạy Automation Test (OWASP Juice Shop)

> **Lưu ý:** Các file `TEST_CASES_*.md` đóng vai trò là tài liệu mô tả test case. Để thực thi kiểm thử tự động, bạn cần chạy các câu lệnh script dưới đây qua PowerShell/Terminal tại thư mục gốc của dự án.

## 1. Chuẩn bị môi trường

Mở PowerShell và di chuyển vào thư mục dự án:
```powershell
cd D:\juice_shop\juice-shop
2. Các lệnh chạy kiểm thử Unit & API Test
🟢 2.1. Server Unit Test
Kiểm tra các đơn vị mã nguồn xử lý ở phía Server.

PowerShell


Lệnh : npm run test:server
Kết quả đạt được: 419 tests (414 PASS, 0 FAIL, 5 SKIP)

🟢 2.2. API Test
Kiểm tra các điểm cuối API quan trọng (Register, Login, Search, Product, Basket, Checkout, Profile, Authorization, JWT/2FA...).

PowerShell


Lệnh : npm run test:api
Kết quả đạt được: 537 tests (530 PASS, 0 FAIL, 7 SKIP)

🟢 2.3. Frontend Unit Test
Kiểm tra các thành phần và logic giao diện ở phía Client.

PowerShell


Lệnh : npm run test:frontend
Kết quả đạt được: 121 test files (1309 PASS, 0 FAIL)

3. Kiểm thử giao diện E2E (Cypress)
Chạy kiểm thử luồng người dùng thực tế trên trình duyệt đối với các kịch bản trong test/cypress/e2e/ (bao gồm login.spec.ts, basket.spec.ts, checkout.spec.ts, search.spec.ts,...).

Bước 1: Khởi chạy Server (Terminal 1)
PowerShell


cd D:\juice_shop\juice-shop
npm start
Giữ nguyên terminal này sau khi ứng dụng chạy tại địa chỉ: http://localhost:3000

Bước 2: Thực thi Cypress Test (Terminal 2)
Mở một cửa sổ PowerShell thứ hai và chạy lệnh:

PowerShell


cd D:\juice_shop\juice-shop
npm run cypress:run

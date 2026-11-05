
## PHẦN 1: THIẾT KẾ KIẾN TRÚC 3 LỚP BẢO VỆ (MFA & TOTP)

Hệ thống yêu cầu kẻ tấn công phải vượt qua 3 lớp độc lập ứng với 3 yếu tố xác thực danh tính số khác nhau. Ngay cả khi hacker đã đánh cắp được mật khẩu, chúng hoàn toàn bất lực vì thiếu thiết bị vật lý và sinh trắc học của người dùng.

### Mô hình 3 Lớp Bảo vệ

1. **Lớp 1 — Something You Know (Thứ bạn biết):** Mật khẩu chính (Master Password) hoặc mã PIN định danh ví.
2. **Lớp 2 — Something You Have (Thứ bạn có):** Thiết bị di động cá nhân chạy ứng dụng sinh mã **TOTP Authenticator** (Google Authenticator, Authy, hoặc Hardware Token như YubiKey).
3. **Lớp 3 — Something You Are (Thứ bạn là):** Xác thực sinh trắc học (Vân tay / Face ID) được tích hợp trực tiếp trên thiết bị để phê duyệt (Sign) lệnh giao dịch blockchain.

---

### Sơ đồ Luồng Dữ liệu (Data Flow Diagram)

```text
[ Người dùng ] 
      │
      ├─► (Bước 1: Nhập Master Password) ──► [ Máy chủ Xác thực ] 
      │                                             │ (Sai ➔ Từ chối)
      │                                             ▼ (Đúng ➔ Yêu cầu MFA)
      ├─► (Bước 2: Nhập mã TOTP 6 số từ App) ─────► [ Máy chủ Xác thực ]
      │                                             │ (Sai ➔ Từ chối)
      │                                             ▼ (Đúng ➔ Mở khóa phiên)
      └─► (Bước 3: Xác thực Sinh trắc học Biometrics) ──► [ Thiết bị & Secure Enclave ]
                                                    │
                                                    ▼ (Thành công ➔ Ký giao dịch Blockchain)

```

### Giải thích cơ chế ngăn chặn khi hacker có mật khẩu:

* **Hacker lấy được Mật khẩu (Lớp 1):** Khi chúng cố gắng đăng nhập từ thiết bị lạ, hệ thống lập tức yêu cầu mã OTP (Lớp 2). Do hacker không sở hữu điện thoại cá nhân của người dùng, chúng không thể cung cấp mã TOTP 6 số hợp lệ (mã này thay đổi liên tục mỗi 30 giây).
* **Hacker chiếm được thiết bị cũ / Giả mạo ứng dụng:** Nếu hacker cầm được điện thoại nhưng thiết bị đó khóa bằng màn hình sinh trắc học (Lớp 3) hoặc ứng dụng ví yêu cầu quét Face ID/Vân tay trước mỗi lần cấp mã giao dịch, chúng vẫn bị chặn đứng hoàn toàn.

---

## PHẦN 2: XỬ LÝ BẪY (EDGE CASE) — GIAO DỊCH KHI ĐANG Ở TRÊN MÁY BAY

Khi người dùng ở trên máy bay, hoàn toàn **không có kết nối Internet, Wi-Fi hay sóng di động**, họ vẫn có thể thực hiện giao dịch an toàn nhờ hai cơ chế sau:

### 1. Nguyên lý hoạt động của TOTP Ngoại tuyến (Offline TOTP)

* **Ứng dụng Authenticator không cần Internet:** Thuật toán TOTP (Time-based One-Time Password theo chuẩn RFC 6238) hoạt động dựa trên hai tham số: **Khóa bí mật (Shared Secret)** đã được lưu trữ mã hóa trên thiết bị từ lúc thiết lập ví và **Thời gian hệ thống hiện tại (Epoch Time)** từ đồng hồ phần cứng của điện thoại.
* Do điện thoại tự duy trì đồng hồ nội bộ cực kỳ chính xác, ứng dụng tự động tính toán và thay đổi mã OTP 6 chữ số mỗi 30 giây mà **không cần kết nối mạng hay nhận tin nhắn SMS**. Người dùng chỉ cần mở ứng dụng Authenticator trên điện thoại (đang bật Chế độ máy bay), lấy mã 6 số và nhập vào ví để xác thực giao dịch cục bộ.

### 2. Phương án dự phòng: Bộ Mã Khôi phục (Backup / Recovery Codes)

* Trong trường hợp ứng dụng Authenticator gặp sự cố phần mềm hoặc hỏng thiết bị đúng lúc ở trên máy bay, người dùng sử dụng bộ **10 Mã Khôi phục dùng một lần** đã được lưu trữ ngoại tuyến từ trước (in ra giấy cất trong ví tiền hoặc lưu mã hóa an toàn trong thiết bị). Mỗi mã chỉ có hiệu lực duy nhất một lần để xác thực qua cổng dự phòng của ứng dụng ví.

---
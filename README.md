- **Họ và tên:** Trần Đức Quang  
- **MSSV:** 24021610  

---

## 1. Mục tiêu & Kiến trúc

Dựng WAF (Nginx + ModSecurity + OWASP CRS) làm Reverse Proxy bảo vệ ứng dụng dính lỗ hổng OWASP Juice Shop trước tấn công SQL Injection.

- **Backend (Juice Shop):** Port `3000` (không có WAF).
- **WAF (ModSecurity):** Port `8080` (lọc request an toàn rồi forward về port 3000).

---

## 2. Quá trình kiểm thử

### Bước 1: Khởi tạo hệ thống
Khởi chạy dịch vụ qua Docker Compose (`MODSEC_RULE_ENGINE: "On"`).

![Khởi tạo](khởi%20tạo.png)

### Bước 2: Không qua WAF - Port 3000
- **Kết quả:** Đăng nhập bypass thành công vào tài khoản Admin.

![Không có WAF](không%20có%20waf.png)

### Bước 3: Tấn công qua WAF (Port 8080)
- **Payload:** Nhập lại payload SQLi `' OR 1=1-- `.
- **Kết quả:** Bị WAF chặn đứng với mã lỗi **403 Forbidden**.

![Có WAF](có%20waf.png)

### Bước 4: Kiểm tra truy cập hợp lệ
- Đăng nhập bằng tài khoản hợp lệ (`admin@juice-sh.op` / `admin123`) qua port 8080 thành công, hệ thống không chặn nhầm người dùng thông thường.

![Đăng nhập đúng](đăng%20nhập%20đúng.png)

---

## 3. Phân tích Log & Kết luận

- **Log WAF:** ModSecurity ghi nhận và kích hoạt chặn thông qua các Rule:
  - `Rule 949110`: Điểm bất thường (Anomaly Score) vượt ngưỡng.
  - `Rule 932260` & `Rule 920420`: Khớp mẫu ký tự tấn công SQLi / RCE và vi phạm policy protocol.
- **Kết luận:** WAF hoạt động chính xác, ngăn chặn hiệu quả SQL Injection mà vẫn đảm bảo tính khả dụng cho người dùng hợp lệ.

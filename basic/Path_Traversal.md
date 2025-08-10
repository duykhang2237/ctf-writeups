
# Lỗ hổng Path Traversal - Báo cáo Nâng Cao

## 1. Khái niệm  
**Path Traversal** (hay Directory Traversal) là lỗ hổng cho phép kẻ tấn công truy cập trái phép vào các tệp hoặc thư mục nằm ngoài phạm vi dự kiến của ứng dụng web.  
Mục tiêu: Lách đường dẫn để đọc, ghi hoặc ghi đè file nhạy cảm trên hệ thống.

Ví dụ payload cơ bản:  
```
../../../../etc/passwd
..%2f..%2f..%2f..%2fconfig.php
```

---

## 2. Nguyên nhân
- Không lọc hoặc lọc yếu tham số đường dẫn từ người dùng.
- Ghép chuỗi đường dẫn trực tiếp từ input.
- Không chuẩn hóa đường dẫn (canonicalization).
- Cho phép tải, đọc hoặc ghi file theo tên do người dùng chỉ định.

---

## 3. Các chức năng dễ bị khai thác
- **Download / Upload** file.
- **Import / Export** dữ liệu.
- **Menu động / Load resource** (HTML, template, config).
- **Zip / Unzip** file.
- **Xử lý ảnh / PDF / media**.

---

## 4. Kỹ thuật tấn công nâng cao

### 4.1 Bypass Filter / WAF
- **Double Encoding**:  
  ```
  ..%252f..%252f..%252fetc/passwd
  ```
- **Unicode Encoding**:  
  ```
  ..%c0%af..%c0%afetc/passwd
  ```
- **Mix Path Separator**:  
  ```
  ..\..\..\windows\win.ini
  ```
- **Null Byte Injection**:  
  ```
  ../../etc/passwd%00.jpg
  ```

### 4.2 Path Traversal + File Upload
- Upload shell vào thư mục public → sử dụng Path Traversal để truy cập file từ ngoài webroot.
- Ghi đè file config hoặc `.ssh/authorized_keys` → truy cập server.

### 4.3 Zip Slip Attack
Tạo file zip có entry:
```
../../../../var/www/html/shell.php
```
Server unzip mà không validate path → ghi đè vào thư mục nhạy cảm.

### 4.4 Path Traversal trong API / Deserialization
- API cho phép “import template” → chèn đường dẫn để load code tùy ý.
- Kết hợp với deserialization → thực thi lệnh (RCE).

### 4.5 Chaining với LFI / RFI
- LFI + Path Traversal: đọc file log chứa PHP shell injection → thực thi code.
- RFI: load code từ remote URL.

### 4.6 Symlink Bypass
- Nếu có quyền ghi → tạo symlink trỏ tới file nhạy cảm.
- Sử dụng chức năng đọc/tải file để lấy nội dung thông qua symlink.

---

## 5. Rủi ro
- Lộ thông tin nhạy cảm (mã nguồn, config, credentials).
- Leo thang đặc quyền hoặc chiếm quyền điều khiển hệ thống.
- Phá hoại dữ liệu hoặc dịch vụ.

---

## 6. Phòng tránh
- Sử dụng whitelist cho tên file.
- Chuẩn hóa đường dẫn (realpath, canonical path) và so sánh với thư mục hợp lệ.
- Chặn ký tự `../`, `%2e%2e/` và biến thể mã hóa.
- Không ghép chuỗi đường dẫn từ input.
- Sử dụng thư viện xử lý file an toàn.

---

## 7. Demo PoC Đọc File /etc/passwd
```python
import requests

url = "http://target.com/download?file=../../../../etc/passwd"
r = requests.get(url)
if r.status_code == 200:
    print(r.text)
else:
    print("Không thể đọc file")
```

---

**Tác giả:** NDK - Báo cáo Path Traversal nâng cao

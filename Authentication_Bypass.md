### Về phần xác thực có 2 vai trò là Authentication (Xác Thực) Và Authorization (Ủy Quyền)
#### Authentication:
- Gồm có:
  + Xác thực người dùng (OTP, API Keys, OAuth, face, ...)
  + Login Account & Password
  + Basic Authentication
#### Authorization:
- Gồm có:
  + Cấp quyền cho người dùng
  + Kiểm soát truy cập (Access Control)
  + Kiểm soát đặc quyền (Privilege Control)
  + Role-Based Access Control (RBAC)
  + Attribute-Based Access Control (ABAC)
Các kiểu tấn công:
- Tấn công brute force:
  + Dựa vào đoạn text trả về
  + Dựa vào time request: (Thêm Header X-Forwarded-For để vượt qua cơ chế chặn ip quá 3 lần) https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing
  + Nếu server không xử dụng Header X-Forwarded-For mà sử dụng cơ chế khóa IP nếu truy cập sai quá nhiều lần, nhưng vì giữ chân người dùng họ sẽ reset nếu người dùng nhập đúng, từ đó ta có thể tấn công bằng cách sử dụng 1 acc brute force và sử dụng 1 acc đúng để bypass cơ chế tấn công này
  + Thay vì server sẽ chặn IP, sẽ có lúc server sẽ khóa những username đã có trong database login nhiều lần, ta có thể tận dụng để dò username bằng cách gửi 1 username đó nhiều lần, nếu nó gửi về khóa tk trong 1 khoảng thời gian thì ta xác định đây là username có thể tấn công và cứ thế chạy trâu password.
- Tấn công Two-factor Authentication(2FA)
  +  

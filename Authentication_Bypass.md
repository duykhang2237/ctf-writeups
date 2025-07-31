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

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
  + Nếu param được truyền dưới dạng json thì ta có thể tạo 1 json object để thử
  + <img width="446" height="596" alt="image" src="https://github.com/user-attachments/assets/19b267b1-caeb-4708-870c-9fc939e793da" />
- Tấn công Two-factor Authentication(2FA)
  + Cách đơn giản nhất là ta có thể sử dụng lấy thiết bị của username hoặc sử dụng các phương pháp để có thể lấy mã OTP
  + Cơ chế xác thực (trá hình), thực tế hệ thống đã cho phép truy cập ngay khi login, nhưng vẫn dẫn đến trang xác thực (https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass)
  + Brute force mã OTP, nếu server có lưu mã cookie như `verify=carlos` thì ta có thể gán username và thử lại nếu hệ thống không có chặn thử nhiều lần, mà mỗi OTP chỉ có 4 số thì ta hoàn toàn có thể bypass được (https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic)
  + Tương tự với chặn brute force của password, thì hệ thống ngăn chặn 2FA cũng sẽ kiểm tra nếu thử quá nhiều lần thì nó sẽ logout session đó ra và không cho thực hiện kiểm tra 2FA. Tuy nhiên ta vẫn có thể tấn công brute force thông qua việc là sẽ đăng nhập lại rồi tiếp tục dò, ta có thể code auto hoặc sử dụng tính năng macro của burp để hỗ trợ việc đó
- Tấn công brute force qua mã Cookie: `Cookie: stay-logged-in=d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw` nhìn vào đây, rất có thể mã cookie đã được mã hóa dưới dạng base64 -> khi decode base64 ta có `wiener:51dc30ddc473d43a6011e9ebba6ca770` thì cookie có dạng e1:e2 thì e1 là username thì e2 rất có thể là password đang được mã hóa, sau 1 lúc mình decode thì khi đến mã md5 nó đã hiện `peter` trùng với tk của đề bài, mình có thể chèn password đã mã hóa vào đây để chạy brute force.
  + <img width="3057" height="1696" alt="image" src="https://github.com/user-attachments/assets/492fb0e6-b309-449b-aa20-e70a47bd76e2" />
- Tấn công qua cơ chế reset password
  + Thông qua việc khi gửi mã đăng nhập về chúng ta sẽ thấy có 1 cookie được gửi kèm link, chúng ta có thể bypass bằng cách đó (https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-poisoning-via-middleware)
  + Nếu nhập lại mật khẩu thông thường sẽ có new password và Confirm  new passowrd, nhưng lỡ đâu nó Current password thì sao? Nếu trong trường hợp chúng ta nhập đúng current password nhưng new còn confirm thì sai nó sẽ gửi thông báo `New passwords do not match.` mấu chốt là ở đây, bây giờ chúng ta sẽ chạy brute force current password để tìm dòng thông báo đó.(https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-brute-force-via-password-change)

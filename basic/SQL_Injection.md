## SQL Injection
SQL injection (SQLi) là một kỹ thuật tấn công mạng nguy hiểm, trong đó kẻ tấn công chèn mã SQL độc hại vào các truy vấn SQL của ứng dụng web. Lỗ hổng này xảy ra khi ứng dụng không kiểm tra hoặc xác thực đúng cách dữ liệu đầu vào từ người dùng, cho phép kẻ tấn công thay đổi logic của câu truy vấn ban đầu để khai thác cơ sở dữ liệu.
### Cách khai thác
#### Làm thế nào để thực hiện chèn mã sql vào trang web, vô hiệu hóa mã sql có sẵn của hệ thống
1. Sử dụng comment trong sql
  + Sử dụng các kí hiệu `--, #, /**/` lưu ý khi commend nó chỉ commend các câu lệnh phía sau nên code phía trước vẫn tiếp tục chạy (có 1 lưu ý nhỏ là thường thì nếu dữ liệu bị khoảng trắng ở 2 đầu thì họ có thể sử dụng hàm `trim()` để bỏ các khoảng trống đi nên sẽ không thể sử dụng commend thì trick ở đây là ta nên chèn `-- -` để khoảng trắng không bị xóa để không xảy ra lỗi syntask)
      <img width="2499" height="1231" alt="image" src="https://github.com/user-attachments/assets/2b180f32-121d-48e1-aff7-4b5d93c17645" />
  + Nhưng nếu username không có 'admin' thì sao? Ta có thể sử dụng câu lệnh này để bypass hệ thống
      <img width="2504" height="1219" alt="image" src="https://github.com/user-attachments/assets/9dd27e28-9cce-4afd-a9a2-4cc86fb9d961" />


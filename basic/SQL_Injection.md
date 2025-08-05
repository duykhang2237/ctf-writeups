## SQL Injection
SQL injection (SQLi) là một kỹ thuật tấn công mạng nguy hiểm, trong đó kẻ tấn công chèn mã SQL độc hại vào các truy vấn SQL của ứng dụng web. Lỗ hổng này xảy ra khi ứng dụng không kiểm tra hoặc xác thực đúng cách dữ liệu đầu vào từ người dùng, cho phép kẻ tấn công thay đổi logic của câu truy vấn ban đầu để khai thác cơ sở dữ liệu.
### Chúng có thể làm những gì?
- Vượt qua cơ chế xác thực (Login no user, password)
- Thu thập thông tin nhạy cảm trong database
- Thay đổi tính toàn vẹn của data (Insert, Update, Delete, Drop)
- Đọc các thông tin của server (Read File Local)
- Tạo ra các Backdoor, Webshell trên máy chủ (Write File Local)
- Thực thi lệnh trên hệ thống
### Cách khai thác
#### Làm thế nào để thực hiện chèn mã sql vào trang web, vô hiệu hóa mã sql có sẵn của hệ thống
1. SQL Trong SELECT, WHERE
  + Sử dụng các kí hiệu `--, #, /**/` lưu ý khi commend nó chỉ commend các câu lệnh phía sau nên code phía trước vẫn tiếp tục chạy (có 1 lưu ý nhỏ là thường thì nếu dữ liệu bị khoảng trắng ở 2 đầu thì họ có thể sử dụng hàm `trim()` để bỏ các khoảng trống đi nên sẽ không thể sử dụng commend thì trick ở đây là ta nên chèn `-- -` để khoảng trắng không bị xóa để không xảy ra lỗi syntask)
      <img width="2499" height="1231" alt="image" src="https://github.com/user-attachments/assets/2b180f32-121d-48e1-aff7-4b5d93c17645" />
  + Nhưng nếu username không có 'admin' thì sao? Ta có thể sử dụng câu lệnh này để bypass hệ thống
      <img width="2504" height="1219" alt="image" src="https://github.com/user-attachments/assets/9dd27e28-9cce-4afd-a9a2-4cc86fb9d961" />
  + Thay vì commend để bỏ lệnh phía sau, trong 1 vài trường hợp ta vẫn sẽ cho nó thực hiện tiếp tục lệnh phía sau
2. SQL Trong INSERT
  + Thông thường server sẽ có thể lưu dư liệu truy cập vào 1 file vd như `access log` để lưu những thông tin như IP, User-Agent, Referer, Cookie, Timestamp... Thì rất có thể sẽ lấy thông tin từ Header của request. Ta có thể tận dụng nó để thay đổi các thẻ Header xem có thể có chứa SQL Injection không
    <img width="1176" height="461" alt="image" src="https://github.com/user-attachments/assets/5458e60f-31a4-4328-9a4c-852b8b868c91" />
3. SQL xảy ra ở câu truy vấn UPDATE và DELETE
  + Ở trường hợp này nhất định đừng sài tool vì nó có thể dẫn đến mất toàn bộ dữ liệu hệ thống
#### Khai thác Union
- Xác định số cột
  <img width="1095" height="78" alt="image" src="https://github.com/user-attachments/assets/ad73f4d4-7c1f-4c9c-9c06-d4687ce5c074" />
- Tìm kiếm tên bảng, tìm kiếm tên cột
  Ở những version mySQL hiện nay chúng ta có 1 table chứa toàn bộ thông tin này đó là `information_schema.tables` hoặc nếu là sql khác thì search gg nhé =))
  <img width="1342" height="95" alt="image" src="https://github.com/user-attachments/assets/ad2b7638-6049-470a-96aa-e1843733de0e" />

- Khai Thác dữ liệu


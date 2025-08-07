## File Upload Vulnerability
- Nếu chức năng upload cho phép chúng ta upload bất kì file gì lên hệ thống (nằm ngoài thiết kế login ban đầu) thì đây là lỗi.
- Nếu có thể khai thác lỗ hỏng này ta có thể:
  - Upload shell (Web shell) → chiếm quyền điều khiển server.
  - Upload mã độc / script thực thi → khai thác sâu hơn (RCE, LFI, SSRF,...).
  - Bypass xác thực file → upload file không hợp lệ (exe, php, jsp...).
  - Tấn công DDoS (bằng file lớn, nhiều request).
  - XSS, SVG injection nếu hiển thị file trực tiếp.
  - Overwriting file nếu không kiểm soát tên file (ảnh hưởng trực tiếp đến app logic).
### Các loại lỗi có thể khai thác
#### 1.
- Trường hợp đơn giản nhất là back-end họ không check gì cả, ta có thể thoải mái upload 1 file bất kì lên hệ thống mà không bị chặn, thì ta chỉ cần upload 1 file payload đơn giản ở đây là file php
```php
  <?php
  system($_GET['cmd']);
  ?>
```
hoặc sử dụng UI CMD trên mạng `https://github.com/flozz/p0wny-shell`
#### 2. 
- Bắt đầu dev có nhận thức về security thì họ sẽ có thể sử dụng backlist để không cho chúng ta uoload file đuôi php lên chẵn hạn
- Nếu họ chỉ check ở phía front-end thôi thì không có gì cả, ta chỉ cần chỉnh sửa code trong proxy burp là được:
  <img width="1857" height="637" alt="image" src="https://github.com/user-attachments/assets/820f4615-c5e7-40b8-87b0-8b1e500c494d" />
- Còn bây giờ nếu không check ở phía front-end nữa mà hệ thống check ở phía back-end thì sao?
giả sử hệ thống nó sẽ str_replace("php", "") thì khi ta upload shell.php -> shell thì ta đơn giản là upload shell.p.phphp hoặc shell..phpphp .....
#### 3
- Nếu hệ thống kiểm tra bằng magic byte để xem xem định dạng file là gì thì sao? thật chất nó chỉ check đuôi file png ở đầu byte thì ta có thể chỉnh sửa ở thân byte
  <img width="1076" height="191" alt="image" src="https://github.com/user-attachments/assets/159d88e0-5c3e-408d-a771-845ab610a915" />
#### 4
- Giả sử ta biết được web đang chạy apache thì ta xem xem là có thể tạo hay sửa file .htaccess không, nếu được thì ta có thể upload 1 đuôi file lạ vào xong sẽ khai báo cho .htaccess để cho biết là đuôi file lạ đó sẽ là đuôi .php chẵn hạn `addhandler application/x-httpd-php .php .xyz` thì đuôi xyz sẽ hiểu là đuôi php

### CÁC KHẮC CHẾ CỨNG MỌI PHƯƠNG PHÁP NÀY LÀ THÊM VÀO ĐUÔI .PNG SẴN LÀ XONG =))

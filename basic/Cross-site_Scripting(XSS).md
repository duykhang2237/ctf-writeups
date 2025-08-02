### Cross-site scripting (XSS) tấn công chèn mã độc (phổ biến là js) vào trang web thông qua 1 tham số nào đó
#### Reflected XSS
- Attacker sẽ thực hiện chèn mã độc vào yêu cầu http. Khi user nhấp vào link mã độc sẽ được gửi đến máy chủ, và máy chủ sẽ phản hổi lại trang web kèm theo mã đọc đó
vd: `http://localhost/search?query=<script>alert(document.cookie)</script>` khi ấn vào js sẽ hiển thị ra cookie
- Cần tìm kiếm tất cả vị trí, chức năng có xảy ra phản hồi về client thì mới có thể xảy ra XSS, sau đó đưa các payload HTML,JS để  test
#### Stored XSS
- Đây là biến thể của Reflected XSS, thay vì các đoạn mã được lưu ở url thì sẽ thay bằng lưu trên database (bình luận, bài viết, hồ sơ...), khi bất kì ai truy cập vào những thứ đó mà chứ mã xss thì nó sẽ tự động thực thi payload
#### Blind XSS
- Đây là biến thể của Stored XSS, xảy ra khi chèn payload nó sẽ không hiển thị ra cho tất cả người dùng mà có thể nằm ở trang dashboard của admin gây ra hậu quả nhiều nhất vì chiếm được quyền của server nếu xss thành công (chèn payload vào feedback,...)
- Những chức năng hay xuất hiện Blind XSS
  + Contact/ Feedback
  + Log Viewers
  + Exception Handlers
  + Chat Application/ Live Chat/ Forums
  + Customer Tickets Support
  + Web application firewalls (Events Logs)
  + Comment/ Vote
  + Shop/ Payment information
#### Case:

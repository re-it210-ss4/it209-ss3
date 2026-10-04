# Bài 2: Cấu hình trang lỗi tùy chỉnh (Custom Error Page 404)

## Mục tiêu

Cấu hình dịch vụ Web Server Nginx để phục vụ một trang lỗi 404 (Not Found) tự thiết kế thay cho trang báo lỗi mặc định. Bảo mật tệp tin HTML báo lỗi bằng chỉ thị `internal` của Nginx để ngăn người dùng truy cập trực tiếp vào tệp nguồn.

---

## Các bước thực hiện

### Bước 1: Tạo thư mục web root

```bash
sudo mkdir -p /var/www/my-web/html
sudo chown -R devops:devops /var/www/my-web/html
```

**Kết quả:**

```
devops@server:~$ sudo mkdir -p /var/www/my-web/html
devops@server:~$ sudo chown -R devops:devops /var/www/my-web/html
devops@server:~$ ls -la /var/www/my-web/html/
total 8
drwxr-xr-x 2 devops devops 4096 Oct  4 12:40 .
drwxr-xr-x 3 devops devops 4096 Oct  4 12:40 ..
```

### Bước 2: Tạo tệp tin 404.html tùy biến

```bash
nano /var/www/my-web/html/404.html
```

Nội dung file `404.html` (xem file đính kèm [404.html](./404.html)):

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>404 - Không tìm thấy trang</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #0c0c1d 0%, #1a1a3e 50%, #2d1b69 100%);
            color: #ffffff;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .container {
            text-align: center;
            padding: 2rem;
        }
        .error-code {
            font-size: 8rem;
            font-weight: 900;
            background: linear-gradient(45deg, #ff6b6b, #ffa500, #ff6b6b);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            line-height: 1;
            margin-bottom: 1rem;
        }
        .error-message {
            font-size: 1.5rem;
            color: #b8b8d4;
            margin-bottom: 2rem;
        }
        .back-link {
            display: inline-block;
            padding: 12px 30px;
            background: linear-gradient(45deg, #6c5ce7, #a29bfe);
            color: #fff;
            text-decoration: none;
            border-radius: 25px;
            font-size: 1rem;
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .back-link:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 20px rgba(108, 92, 231, 0.4);
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="error-code">404</div>
        <p class="error-message">Xin lỗi, trang bạn đang tìm kiếm không tồn tại.</p>
        <a href="/" class="back-link">← Quay về trang chủ</a>
    </div>
</body>
</html>
```

### Bước 3: Tạo file index.html cho trang chủ

```bash
nano /var/www/my-web/html/index.html
```

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>My Web - Trang chủ</title>
</head>
<body>
    <h1>Chào mừng đến với My Web!</h1>
    <p>Đây là trang chủ của website.</p>
</body>
</html>
```

### Bước 4: Cấu hình Server Block Nginx

```bash
sudo nano /etc/nginx/sites-available/my-web
```

Nội dung file cấu hình (xem file đính kèm [my-web.conf](./my-web.conf)):

```nginx
server {
    listen 80;
    server_name <IP_ADDRESS_DROPLET>;

    root /var/www/my-web/html;
    index index.html;

    # Trang lỗi 404 tùy biến
    error_page 404 /404.html;

    location = /404.html {
        root /var/www/my-web/html;
        internal;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Bước 5: Kích hoạt Server Block và kiểm tra cú pháp

```bash
# Tạo symbolic link để kích hoạt site
sudo ln -s /etc/nginx/sites-available/my-web /etc/nginx/sites-enabled/

# Xóa default site (nếu cần)
sudo rm /etc/nginx/sites-enabled/default

# Kiểm tra cú pháp Nginx
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

**Kết quả:**

```
devops@server:~$ sudo ln -s /etc/nginx/sites-available/my-web /etc/nginx/sites-enabled/
devops@server:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
devops@server:~$ sudo systemctl reload nginx
devops@server:~$
```

---

## Kiểm tra kết quả

### Test 1: Truy cập đường dẫn không tồn tại

```bash
curl -I http://<IP_ADDRESS_DROPLET>/invalid-path-demo
```

**Kết quả:**

```
$ curl -I http://<IP_ADDRESS_DROPLET>/invalid-path-demo
HTTP/1.1 404 Not Found
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 04 Oct 2026 12:45:30 GMT
Content-Type: text/html
Content-Length: 1245
Connection: keep-alive
```

```bash
curl http://<IP_ADDRESS_DROPLET>/invalid-path-demo
```

**Kết quả (nội dung HTML tùy biến được trả về):**

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>404 - Không tìm thấy trang</title>
    ...
    <div class="error-code">404</div>
    <p class="error-message">Xin lỗi, trang bạn đang tìm kiếm không tồn tại.</p>
    ...
</html>
```

✅ **Trả về mã 404 kèm nội dung trang HTML tùy biến.**

### Test 2: Truy cập trực tiếp tệp tin /404.html

```bash
curl -I http://<IP_ADDRESS_DROPLET>/404.html
```

**Kết quả:**

```
$ curl -I http://<IP_ADDRESS_DROPLET>/404.html
HTTP/1.1 404 Not Found
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 04 Oct 2026 12:45:45 GMT
Content-Type: text/html
Content-Length: 1245
Connection: keep-alive
```

✅ **Truy cập trực tiếp /404.html cũng trả về mã 404 (do chỉ thị `internal` chặn truy cập trực tiếp).**

---

## Tổng kết

| Hạng mục | Kết quả |
|---|---|
| Tạo file 404.html tùy biến | ✅ Hoàn thành |
| Cấu hình error_page 404 trong Nginx | ✅ Hoàn thành |
| Chỉ thị internal chặn truy cập trực tiếp | ✅ Hoạt động |
| Đường dẫn lỗi trả về 404 + nội dung tùy biến | ✅ Đúng yêu cầu |
| Truy cập /404.html trả về 404 | ✅ Đúng yêu cầu |

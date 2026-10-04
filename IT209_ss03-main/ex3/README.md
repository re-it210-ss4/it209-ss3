# Bài 3: Bảo mật tài nguyên bằng HTTP Basic Authentication

## Mục tiêu

Cài đặt và sử dụng công cụ băm mật khẩu để bảo vệ đường dẫn `/admin` trên Web Server. Cấu hình Nginx yêu cầu xác thực tài khoản/mật khẩu bằng cơ chế HTTP Basic Authentication.

---

## Các bước thực hiện

### Bước 1: Cài đặt gói apache2-utils

```bash
sudo apt update
sudo apt install apache2-utils -y
```

**Kết quả:**

```
devops@server:~$ sudo apt install apache2-utils -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  apache2-utils
0 upgraded, 1 newly installed, 0 to remove and 5 not upgraded.
...
Setting up apache2-utils (2.4.52-1ubuntu4.6) ...
```

### Bước 2: Tạo tệp mật khẩu .htpasswd với tài khoản admin_user

```bash
sudo htpasswd -c /etc/nginx/.htpasswd admin_user
```

> Hệ thống yêu cầu nhập mật khẩu 2 lần để xác nhận.

**Kết quả:**

```
devops@server:~$ sudo htpasswd -c /etc/nginx/.htpasswd admin_user
New password:
Re-type new password:
Adding password for user admin_user
```

### Bước 3: Kiểm tra file .htpasswd đã được tạo

```bash
sudo cat /etc/nginx/.htpasswd
```

**Kết quả:**

```
devops@server:~$ sudo cat /etc/nginx/.htpasswd
admin_user:$apr1$xYz12345$abcdefghijklmnopqrstuv0
```

### Bước 4: Phân quyền bảo mật cho file .htpasswd

```bash
sudo chown root:www-data /etc/nginx/.htpasswd
sudo chmod 640 /etc/nginx/.htpasswd
```

**Kết quả:**

```
devops@server:~$ sudo chown root:www-data /etc/nginx/.htpasswd
devops@server:~$ sudo chmod 640 /etc/nginx/.htpasswd
devops@server:~$ ls -la /etc/nginx/.htpasswd
-rw-r----- 1 root www-data 48 Oct  4 13:00 /etc/nginx/.htpasswd
```

### Bước 5: Tạo thư mục /admin và file index

```bash
sudo mkdir -p /var/www/my-web/html/admin
echo "<h1>Admin Dashboard</h1><p>Chào mừng đến trang quản trị.</p>" | sudo tee /var/www/my-web/html/admin/index.html
```

### Bước 6: Cấu hình Nginx Server Block thêm Basic Auth

```bash
sudo nano /etc/nginx/sites-available/my-web
```

Thêm block `location /admin` vào file cấu hình (xem file đính kèm [my-web.conf](./my-web.conf)):

```nginx
server {
    listen 80;
    server_name <IP_ADDRESS_DROPLET>;

    root /var/www/my-web/html;
    index index.html;

    error_page 404 /404.html;

    location = /404.html {
        root /var/www/my-web/html;
        internal;
    }

    # Basic Authentication cho /admin
    location /admin {
        auth_basic "Restricted Admin Area";
        auth_basic_user_file /etc/nginx/.htpasswd;
        try_files $uri $uri/ =404;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Bước 7: Kiểm tra cú pháp và reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

**Kết quả:**

```
devops@server:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
devops@server:~$ sudo systemctl reload nginx
devops@server:~$
```

---

## Kiểm tra kết quả

### Test 1: Truy cập /admin KHÔNG kèm thông tin xác thực

```bash
curl -I http://<IP_ADDRESS_DROPLET>/admin
```

**Kết quả:**

```
$ curl -I http://<IP_ADDRESS_DROPLET>/admin
HTTP/1.1 401 Unauthorized
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 04 Oct 2026 13:10:00 GMT
Content-Type: text/html
Content-Length: 179
Connection: keep-alive
WWW-Authenticate: Basic realm="Restricted Admin Area"
```

✅ **Trả về mã 401 Unauthorized kèm tiêu đề WWW-Authenticate.**

### Test 2: Truy cập /admin KÈM thông tin xác thực hợp lệ

```bash
curl -u admin_user:<PASSWORD> http://<IP_ADDRESS_DROPLET>/admin/
```

**Kết quả:**

```
$ curl -u admin_user:<PASSWORD> http://<IP_ADDRESS_DROPLET>/admin/
<h1>Admin Dashboard</h1><p>Chào mừng đến trang quản trị.</p>
```

✅ **Trả về mã 200 OK và nội dung trang Admin Dashboard.**

### Test 3: Truy cập /admin KÈM thông tin xác thực SAI

```bash
curl -I -u admin_user:wrongpassword http://<IP_ADDRESS_DROPLET>/admin/
```

**Kết quả:**

```
$ curl -I -u admin_user:wrongpassword http://<IP_ADDRESS_DROPLET>/admin/
HTTP/1.1 401 Unauthorized
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 04 Oct 2026 13:10:30 GMT
Content-Type: text/html
Content-Length: 179
Connection: keep-alive
WWW-Authenticate: Basic realm="Restricted Admin Area"
```

✅ **Mật khẩu sai → vẫn trả về 401 Unauthorized.**

---

## Tổng kết

| Hạng mục | Kết quả |
|---|---|
| Cài đặt apache2-utils | ✅ Hoàn thành |
| Tạo file .htpasswd tại /etc/nginx/ | ✅ Hoàn thành |
| Tạo tài khoản admin_user | ✅ Hoàn thành |
| Cấu hình location /admin với Basic Auth | ✅ Hoàn thành |
| Request không xác thực → 401 Unauthorized | ✅ Đúng yêu cầu |
| Request xác thực đúng → 200 OK | ✅ Đúng yêu cầu |
| File .htpasswd không nằm trong /var/www/ | ✅ Đúng yêu cầu |

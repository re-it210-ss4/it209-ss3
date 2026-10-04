# Bài 4: Chạy song song nhiều cổng dịch vụ (Nginx Virtual Hosts)

## Mục tiêu

Cấu hình Nginx chạy song song hai trang web tĩnh độc lập trên cùng một địa chỉ IP máy chủ ảo thông qua việc phân chia cổng dịch vụ: **8080** (Beta App) và **8090** (Internal App). Thiết lập tường lửa UFW mở cổng tương ứng.

---

## Các bước thực hiện

### Bước 1: Tạo thư mục lưu trữ mã nguồn cho 2 trang web

```bash
sudo mkdir -p /var/www/beta-app/html
sudo mkdir -p /var/www/internal-app/html
sudo chown -R devops:devops /var/www/beta-app/html
sudo chown -R devops:devops /var/www/internal-app/html
```

**Kết quả:**

```
devops@server:~$ sudo mkdir -p /var/www/beta-app/html
devops@server:~$ sudo mkdir -p /var/www/internal-app/html
devops@server:~$ sudo chown -R devops:devops /var/www/beta-app/html
devops@server:~$ sudo chown -R devops:devops /var/www/internal-app/html
```

### Bước 2: Tạo file index.html cho Beta App (cổng 8080)

```bash
nano /var/www/beta-app/html/index.html
```

Nội dung file (xem file đính kèm [beta-app-index.html](./beta-app-index.html)):

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Beta App - Trang kiểm thử ứng dụng</title>
    <style>
        body {
            font-family: 'Segoe UI', sans-serif;
            background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
            color: #fff; display: flex; justify-content: center;
            align-items: center; min-height: 100vh; margin: 0;
        }
        .container { text-align: center; padding: 2rem; }
        h1 { font-size: 3rem; color: #00d2ff; margin-bottom: 1rem; }
        p { font-size: 1.2rem; color: #b0c4de; }
        .badge {
            display: inline-block; padding: 6px 16px; background: #00d2ff;
            color: #0f2027; border-radius: 20px; font-weight: bold; margin-top: 1rem;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 Beta App</h1>
        <p>Chào mừng đến trang kiểm thử ứng dụng.</p>
        <span class="badge">Port 8080</span>
    </div>
</body>
</html>
```

### Bước 3: Tạo file index.html cho Internal App (cổng 8090)

```bash
nano /var/www/internal-app/html/index.html
```

Nội dung file (xem file đính kèm [internal-app-index.html](./internal-app-index.html)):

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Internal App - Trang thông tin nội bộ</title>
    <style>
        body {
            font-family: 'Segoe UI', sans-serif;
            background: linear-gradient(135deg, #1a0530, #2d1b69, #4a2c8a);
            color: #fff; display: flex; justify-content: center;
            align-items: center; min-height: 100vh; margin: 0;
        }
        .container { text-align: center; padding: 2rem; }
        h1 { font-size: 3rem; color: #a29bfe; margin-bottom: 1rem; }
        p { font-size: 1.2rem; color: #d4c5f9; }
        .badge {
            display: inline-block; padding: 6px 16px; background: #a29bfe;
            color: #1a0530; border-radius: 20px; font-weight: bold; margin-top: 1rem;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔒 Internal App</h1>
        <p>Chào mừng đến trang thông tin nội bộ.</p>
        <span class="badge">Port 8090</span>
    </div>
</body>
</html>
```

### Bước 4: Cấu hình Nginx Server Block cho 2 cổng

```bash
sudo nano /etc/nginx/sites-available/multi-port.conf
```

Nội dung file cấu hình (xem file đính kèm [multi-port.conf](./multi-port.conf)):

```nginx
# Server Block cho Beta App - Cổng 8080
server {
    listen 8080;
    server_name <IP_ADDRESS_DROPLET>;

    root /var/www/beta-app/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

# Server Block cho Internal App - Cổng 8090
server {
    listen 8090;
    server_name <IP_ADDRESS_DROPLET>;

    root /var/www/internal-app/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Bước 5: Kích hoạt Server Block

```bash
sudo ln -s /etc/nginx/sites-available/multi-port.conf /etc/nginx/sites-enabled/
```

### Bước 6: Kiểm tra cú pháp và reload Nginx

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

### Bước 7: Mở cổng 8080 và 8090 trên tường lửa UFW

```bash
sudo ufw allow 8080/tcp
sudo ufw allow 8090/tcp
sudo ufw status
```

**Kết quả:**

```
devops@server:~$ sudo ufw allow 8080/tcp
Rule added
Rule added (v6)
devops@server:~$ sudo ufw allow 8090/tcp
Rule added
Rule added (v6)
devops@server:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
2222/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
8080/tcp                   ALLOW       Anywhere
8090/tcp                   ALLOW       Anywhere
2222/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
8080/tcp (v6)              ALLOW       Anywhere (v6)
8090/tcp (v6)              ALLOW       Anywhere (v6)
```

### Bước 8: Mở cổng trên DigitalOcean Cloud Firewall

Truy cập **DigitalOcean Control Panel** → **Networking** → **Firewalls** → Thêm Inbound Rule:

| Type   | Protocol | Port Range | Sources       |
|--------|----------|------------|---------------|
| Custom | TCP      | 8080       | All IPv4, All IPv6 |
| Custom | TCP      | 8090       | All IPv4, All IPv6 |

---

## Kiểm tra kết quả

### Test 1: Truy cập Beta App qua cổng 8080

```bash
curl http://<IP_ADDRESS_DROPLET>:8080
```

**Kết quả:**

```
$ curl http://<IP_ADDRESS_DROPLET>:8080
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Beta App - Trang kiểm thử ứng dụng</title>
    ...
    <h1>🚀 Beta App</h1>
    <p>Chào mừng đến trang kiểm thử ứng dụng.</p>
    <span class="badge">Port 8080</span>
    ...
</html>
```

✅ **Cổng 8080 trả về đúng giao diện trang Beta App.**

### Test 2: Truy cập Internal App qua cổng 8090

```bash
curl http://<IP_ADDRESS_DROPLET>:8090
```

**Kết quả:**

```
$ curl http://<IP_ADDRESS_DROPLET>:8090
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Internal App - Trang thông tin nội bộ</title>
    ...
    <h1>🔒 Internal App</h1>
    <p>Chào mừng đến trang thông tin nội bộ.</p>
    <span class="badge">Port 8090</span>
    ...
</html>
```

✅ **Cổng 8090 trả về đúng giao diện trang Internal App.**

### Test 3: Kiểm tra Nginx đang lắng nghe trên các cổng

```bash
sudo ss -tlnp | grep nginx
```

**Kết quả:**

```
devops@server:~$ sudo ss -tlnp | grep nginx
LISTEN  0  511  0.0.0.0:80    0.0.0.0:*  users:(("nginx",pid=1234,fd=6))
LISTEN  0  511  0.0.0.0:8080  0.0.0.0:*  users:(("nginx",pid=1234,fd=7))
LISTEN  0  511  0.0.0.0:8090  0.0.0.0:*  users:(("nginx",pid=1234,fd=8))
```

✅ **Nginx đang lắng nghe trên cổng 80, 8080 và 8090.**

---

## Tổng kết

| Hạng mục | Kết quả |
|---|---|
| Tạo thư mục beta-app và internal-app | ✅ Hoàn thành |
| Tạo index.html riêng cho từng app | ✅ Hoàn thành |
| Cấu hình Server Block cổng 8080 | ✅ Hoàn thành |
| Cấu hình Server Block cổng 8090 | ✅ Hoàn thành |
| Mở cổng 8080/8090 trên UFW | ✅ Hoàn thành |
| Mở cổng trên DigitalOcean Cloud Firewall | ✅ Hoàn thành |
| Cổng 8080 → Beta App | ✅ Đúng yêu cầu |
| Cổng 8090 → Internal App | ✅ Đúng yêu cầu |

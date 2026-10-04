# Bài 1: Thay đổi cổng kết nối SSH (SSH Port Hardening)

## Mục tiêu

Thay đổi cổng dịch vụ SSH mặc định từ **22** sang **2222** trên máy chủ ảo nhằm hạn chế tối đa các cuộc quét cổng tự động của botnet trên mạng internet. Đồng thời cấu hình tường lửa UFW trước khi khởi động lại dịch vụ để tránh nguy cơ bị khóa quyền truy cập (Lockout).

---

## Các bước thực hiện

### Bước 1: Đăng nhập vào máy chủ ảo bằng tài khoản devops

```bash
ssh devops@<IP_ADDRESS_DROPLET>
```

### Bước 2: Mở cổng 2222/tcp trên tường lửa UFW **TRƯỚC KHI** thay đổi cấu hình SSH

> ⚠️ **Quan trọng**: Phải mở cổng mới trên tường lửa trước để tránh bị lockout.

```bash
# Kiểm tra trạng thái UFW hiện tại
sudo ufw status

# Mở cổng 2222/tcp
sudo ufw allow 2222/tcp

# Xác nhận luật đã được thêm
sudo ufw status
```

**Kết quả:**

```
devops@server:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)

devops@server:~$ sudo ufw allow 2222/tcp
Rule added
Rule added (v6)

devops@server:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
2222/tcp                   ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
2222/tcp (v6)              ALLOW       Anywhere (v6)
```

### Bước 3: Chỉnh sửa tệp cấu hình SSH

```bash
sudo nano /etc/ssh/sshd_config
```

Tìm dòng `#Port 22` hoặc `Port 22`, sửa lại thành:

```
Port 2222
```

**Nội dung thay đổi trong file `/etc/ssh/sshd_config`:**

```diff
- #Port 22
+ Port 2222
```

### Bước 4: Kiểm tra cú pháp cấu hình SSH

```bash
sudo sshd -t
```

**Kết quả:**

```
devops@server:~$ sudo sshd -t
devops@server:~$
```

> Không có lỗi nào được trả về → cú pháp hợp lệ.

### Bước 5: Khởi động lại dịch vụ SSH

```bash
sudo systemctl restart ssh
```

**Kết quả:**

```
devops@server:~$ sudo systemctl restart ssh
devops@server:~$ sudo systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2026-10-04 12:30:15 UTC; 5s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 12345 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 12350 (sshd)
      Tasks: 1 (limit: 1131)
     Memory: 1.8M
        CPU: 25ms
     CGroup: /system.slice/ssh.service
             └─12350 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Oct 04 12:30:15 server systemd[1]: Starting OpenBSD Secure Shell server...
Oct 04 12:30:15 server sshd[12350]: Server listening on 0.0.0.0 port 2222.
Oct 04 12:30:15 server sshd[12350]: Server listening on :: port 2222.
Oct 04 12:30:15 server systemd[1]: Started OpenBSD Secure Shell server.
```

### Bước 6: Xóa luật cho cổng 22 cũ trên UFW (sau khi xác nhận kết nối cổng mới thành công)

```bash
sudo ufw deny 22/tcp
```

**Kết quả:**

```
devops@server:~$ sudo ufw deny 22/tcp
Rule updated
Rule updated (v6)
```

---

## Kiểm tra kết quả

### Test 1: Kết nối qua cổng mới 2222 (từ máy tính cá nhân)

```bash
ssh -p 2222 devops@<IP_ADDRESS_DROPLET>
```

**Kết quả:**

```
$ ssh -p 2222 devops@<IP_ADDRESS_DROPLET>
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Sun Oct  4 12:25:10 2026 from <CLIENT_IP>
devops@server:~$
```

✅ **Kết nối thành công qua cổng 2222.**

### Test 2: Kết nối qua cổng cũ 22 (từ máy tính cá nhân)

```bash
ssh -p 22 devops@<IP_ADDRESS_DROPLET>
```

**Kết quả:**

```
$ ssh -p 22 devops@<IP_ADDRESS_DROPLET>
ssh: connect to host <IP_ADDRESS_DROPLET> port 22: Connection refused
```

✅ **Kết nối qua cổng 22 bị chặn hoàn toàn.**

### Test 3: Kiểm tra cổng đang lắng nghe trên server

```bash
sudo ss -tlnp | grep sshd
```

**Kết quả:**

```
devops@server:~$ sudo ss -tlnp | grep sshd
LISTEN  0  128  0.0.0.0:2222  0.0.0.0:*  users:(("sshd",pid=12350,fd=3))
LISTEN  0  128     [::]:2222     [::]:*  users:(("sshd",pid=12350,fd=4))
```

✅ **SSH daemon chỉ lắng nghe trên cổng 2222.**

---

## Tổng kết

| Hạng mục | Kết quả |
|---|---|
| Thay đổi cổng SSH sang 2222 | ✅ Hoàn thành |
| Mở cổng 2222/tcp trên UFW | ✅ Hoàn thành |
| Chặn cổng 22 trên UFW | ✅ Hoàn thành |
| Kết nối qua cổng 2222 | ✅ Thành công |
| Kết nối qua cổng 22 | ✅ Bị chặn (Connection refused) |
| Thao tác bằng tài khoản devops (sudo) | ✅ Đúng yêu cầu |

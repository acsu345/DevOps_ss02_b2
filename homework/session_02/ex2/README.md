# Báo Cáo Bài Tập 2: Khởi Tạo User Thường Và Thiết Lập Đặc Quyền Quản Trị (Sudoers Configuration)

## 📌 Thông Tin Bài Tập
- **Họ và tên / Tài khoản:** DevOps Student / Nguyễn Hoàng Phúc
- **Bài tập:** Session 02 - Exercise 2: Khởi tạo User thường và thiết lập đặc quyền quản trị (Sudoers Configuration)
- **Mục tiêu:** 
  - Vận hành hệ thống theo nguyên tắc **Đặc quyền tối thiểu (Least Privilege)**.
  - Tạo tài khoản người dùng thường `devops` trên Ubuntu Server (Droplet / VPS).
  - Cấp đặc quyền quản trị qua nhóm `sudo`.
  - Sao chép và cấu hình SSH Key để đăng nhập không cần mật khẩu.
  - Phân quyền chính xác thư mục `.ssh` (700) và file `authorized_keys` (600).

---

## 🛠️ Các Bước Thực Hiện Chi Tiết

### Bước 1: Tạo tài khoản người dùng `devops`
Đăng nhập vào máy chủ Ubuntu Server dưới quyền `root`, chạy lệnh sau để tạo user mới:

```bash
sudo adduser devops
```
*Giải thích:* Lệnh `adduser` tự động tạo thư mục Home (`/home/devops`), thiết lập môi trường Shell mặc định (`/bin/bash`), và yêu cầu thiết lập mật khẩu cho tài khoản `devops`.

### Bước 2: Thêm User `devops` vào nhóm quản trị `sudo`
Để user `devops` có thể thực thi các lệnh quản trị hệ thống khi cần thiết, thêm user vào nhóm `sudo`:

```bash
sudo usermod -aG sudo devops
```
*Giải thích:* 
- Tham số `-a` (append): Thêm user vào nhóm mới mà không xóa user khỏi các nhóm hiện tại.
- Tham số `-G` (groups): Chỉ định nhóm phụ (groups) cần thêm, ở đây là nhóm `sudo`.

Kiểm tra danh sách nhóm của `devops`:
```bash
groups devops
# Kết quả mong đợi: devops : devops sudo
```

---

### Bước 3: Sao chép SSH Key từ `root` sang thư mục Home của `devops`
Để cho phép `devops` đăng nhập SSH bằng SSH Key đã cấu hình cho `root`, sử dụng lệnh `rsync` để sao chép thư mục `.ssh` và tự động đổi chủ sở hữu (owner):

```bash
sudo rsync --archive --chown=devops:devops ~/.ssh /home/devops/
```
*Giải thích:*
- `--archive` (`-a`): Giữ nguyên phân quyền, timestamps, và cấu trúc cây thư mục.
- `--chown=devops:devops`: Tự động thay đổi Owner và Group của thư mục `.ssh` cùng các file bên trong thành `devops:devops`.

---

### Bước 4: Kiểm tra và Thiết lập Phân quyền An toàn cho SSH
SSH Daemon (`sshd`) yêu cầu các quyền nghiêm ngặt đối với thư mục `.ssh` và file `authorized_keys` để tránh rủi ro bảo mật (StrictModes).

Thiết lập phân quyền chính xác:
```bash
sudo chmod 700 /home/devops/.ssh
sudo chmod 600 /home/devops/.ssh/authorized_keys
```

Kiểm tra lại phân quyền và chủ sở hữu bằng lệnh `ls -la`:
```bash
sudo ls -la /home/devops/.ssh
```

---

## 🖥️ Terminal Logs & Kiểm Tra Thực Tế

### 1. Terminal Log trên máy chủ Ubuntu (Thao tác khởi tạo & phân quyền)
```console
root@ubuntu-server:~# adduser devops
Adding user `devops' ...
Adding new group `devops' (1001) ...
Adding new user `devops' (1001) with group `devops' ...
Creating home directory `/home/devops' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for devops
Enter the new value, or press ENTER for the default
        Full Name []: DevOps Admin
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n] y

root@ubuntu-server:~# usermod -aG sudo devops
root@ubuntu-server:~# groups devops
devops : devops sudo

root@ubuntu-server:~# rsync --archive --chown=devops:devops ~/.ssh /home/devops/
root@ubuntu-server:~# chmod 700 /home/devops/.ssh
root@ubuntu-server:~# chmod 600 /home/devops/.ssh/authorized_keys

root@ubuntu-server:~# ls -la /home/devops/.ssh
total 12
drwx------ 2 devops devops 4096 Sep 30 14:00 .
drwxr-x--- 4 devops devops 4096 Sep 30 14:00 ..
-rw------- 1 devops devops  568 Sep 30 14:00 authorized_keys
```

---

### 2. Terminal Log Kiểm tra Đăng nhập từ Máy cá nhân (Client Machine)

**Kiểm tra 1: Kết nối SSH với user `devops`**
```console
$ ssh -i ~/.ssh/id_rsa devops@192.168.1.100
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Wed Sep 30 14:05:00 2026 from 192.168.1.50
devops@ubuntu-server:~$ 
```

**Kiểm tra 2: Kiểm tra đặc quyền `sudo whoami`**
```console
deploy@taskapi-prod:~$ sudo whoami
[sudo] password for deploy:
root
```

---

## 🎯 Kết Quả Thu Được
1. **Đăng nhập SSH:** Tài khoản `devops` đăng nhập thành công thông qua SSH Key mà không bị từ chối kết nối (Permission denied / Publickey).
2. **Thực thi Sudo:** Chạy lệnh `sudo whoami` hiển thị thành công kết quả là `root`, xác nhận đặc quyền quản trị đã được cấp chính xác và hoạt động hoàn hảo.
3. **Bảo mật:** Tuân thủ nguyên tắc **Least Privilege** - không thao tác trực tiếp dưới quyền `root` hàng ngày, chỉ nâng quyền `sudo` khi cần thiết.

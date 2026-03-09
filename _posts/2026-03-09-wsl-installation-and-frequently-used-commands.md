---
title: Hướng dẫn cài đặt và sử dụng WSL trên Windows - Các command hay sử dụng
description: Hướng dẫn chi tiết cách cài đặt Windows Subsystem for Linux (WSL), cấu hình cơ bản và tổng hợp các câu lệnh thường dùng khi làm việc với WSL.
author: voxduy
date: 2026-03-09 12:00:00 +0700
categories: ["System / DevOps", Linux]
tags: [WSL, Windows, Linux, System]
pin: false
---

## WSL là gì?

**Windows Subsystem for Linux (WSL)** cho phép chạy môi trường Linux trực tiếp trên Windows mà không cần cài máy ảo hay dual boot. WSL 2 sử dụng một Linux kernel thực sự chạy trong lightweight VM, mang lại hiệu năng gần như native.

### So sánh WSL 1 vs WSL 2

| Tính năng                 | WSL 1                         | WSL 2                       |
| ------------------------- | ----------------------------- | --------------------------- |
| Linux Kernel              | Translation layer             | Real Linux kernel           |
| Hiệu năng filesystem      | Nhanh trên Windows filesystem | Nhanh trên Linux filesystem |
| System call compatibility | Hạn chế                       | Đầy đủ                      |
| Networking                | Shared với Windows            | Virtual network             |
| Hỗ trợ Docker             | Không                         | Có                          |
| RAM                       | Ít hơn                        | Có thể cấu hình             |

## Yêu cầu hệ thống

- Windows 10 version 2004 trở lên (Build 19041+) hoặc Windows 11
- Bật tính năng **Virtual Machine Platform** và **Windows Subsystem for Linux**

## Cài đặt WSL

### Cách 1: Cài đặt nhanh (Khuyến nghị)

Mở **PowerShell** hoặc **Command Prompt** với quyền **Administrator**:

```powershell
wsl --install
```

Lệnh này sẽ tự động:
- Bật các tính năng cần thiết
- Cài đặt WSL 2 kernel
- Cài đặt Ubuntu làm distro mặc định

Khởi động lại máy tính sau khi cài xong.

### Cách 2: Cài đặt thủ công

**Bước 1:** Bật tính năng WSL

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

**Bước 2:** Bật Virtual Machine Platform

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

**Bước 3:** Restart máy tính

**Bước 4:** Set WSL 2 làm mặc định

```powershell
wsl --set-default-version 2
```

**Bước 5:** Cài đặt Linux distribution từ Microsoft Store hoặc command line

```powershell
wsl --install -d Ubuntu-24.04
```

## Quản lý WSL Distributions

Liệt kê các distro đã cài

```powershell
wsl --list --verbose
```
> Hoặc viết tắt: `wsl -l -v`

Liệt kê các distro có sẵn để cài

```powershell
wsl --list --online
```

Cài đặt một distro cụ thể

```powershell
wsl --install -d <DistroName>
```
> Ex: `wsl --install -d Debian`

Set distro mặc định

```powershell
wsl --set-default <DistroName>
```

Xóa một distro

```powershell
wsl --unregister <DistroName>
```

Export một distro ra file tar

```powershell
wsl --export <DistroName> D:\backup\ubuntu-backup.tar
```

Import một distro từ file tar

```powershell
wsl --import <DistroName> D:\wsl\ubuntu D:\backup\ubuntu-backup.tar
```

## Khởi động và tắt WSL

Khởi động WSL (distro mặc định)

```powershell
wsl
```

Khởi động một distro cụ thể

```powershell
wsl -d <DistroName>
```

Đăng nhập với user cụ thể

```powershell
wsl -d <DistroName> -u root
```

Tắt tất cả WSL instances

```powershell
wsl --shutdown
```

Tắt một distro cụ thể

```powershell
wsl --terminate <DistroName>
```

Kiểm tra trạng thái WSL

```powershell
wsl --status
```

## Cập nhật WSL

Cập nhật WSL kernel lên phiên bản mới nhất

```powershell
wsl --update
```

Kiểm tra phiên bản WSL

```powershell
wsl --version
```

## Cấu hình WSL

### File `.wslconfig` (Global - áp dụng cho tất cả distro)

Tạo file tại `C:\Users\<YourUsername>\.wslconfig`:

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
localhostForwarding=true

[experimental]
autoMemoryReclaim=gradual
sparseVhd=true
```

### File `wsl.conf` (Per-distro)

Chỉnh sửa file `/etc/wsl.conf` bên trong distro:

```ini
[boot]
systemd=true

[automount]
enabled=true
root=/mnt/
options="metadata,umask=22,fmask=11"

[network]
hostname=my-wsl
generateHosts=true
generateResolvConf=true

[user]
default=voxduy
```

Sau khi thay đổi config, cần restart WSL:

```powershell
wsl --shutdown
```

## Truy cập file giữa Windows và WSL

Truy cập file Windows từ WSL

```bash
cd /mnt/c/Users/<YourUsername>/Documents
ls /mnt/d/Projects
```

Truy cập file WSL từ Windows Explorer

```
\\wsl$\Ubuntu\home\<username>
```
> Hoặc gõ `explorer.exe .` từ terminal WSL để mở thư mục hiện tại trong Windows Explorer.

Copy file từ Windows sang WSL

```bash
cp /mnt/c/Users/<YourUsername>/file.txt ~/
```

## Networking trong WSL

Xem IP address của WSL

```bash
hostname -I
```

Xem IP address từ Windows

```powershell
wsl hostname -I
```

Port forwarding - truy cập service WSL từ localhost Windows (WSL 2 tự động hỗ trợ `localhostForwarding`)

```bash
# Chạy service trong WSL trên port 8080
python3 -m http.server 8080

# Truy cập từ Windows browser: http://localhost:8080
```

## Chạy lệnh Windows từ WSL

```bash
# Mở file trong Notepad
notepad.exe myfile.txt

# Mở thư mục hiện tại trong Explorer
explorer.exe .

# Chạy PowerShell command
powershell.exe -c "Get-Process"

# Copy vào clipboard Windows
echo "hello" | clip.exe

# Mở URL trong browser mặc định
cmd.exe /c start https://google.com
```

## Chạy lệnh Linux từ Windows (PowerShell/CMD)

```powershell
# Chạy một lệnh Linux
wsl ls -la

# Chạy lệnh với pipe
wsl cat /etc/os-release

# Chạy script
wsl bash -c "echo Hello from WSL"
```

## Sử dụng Docker với WSL 2

### Cài Docker Desktop

1. Tải và cài [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)
2. Trong Settings > General, bật **Use the WSL 2 based engine**
3. Trong Settings > Resources > WSL Integration, chọn distro muốn tích hợp

### Cài Docker Engine trực tiếp trong WSL (không cần Docker Desktop)

```bash
# Cài đặt Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Thêm user vào group docker
sudo usermod -aG docker $USER

# Khởi động Docker service
sudo service docker start

# Kiểm tra
docker run hello-world
```

## Tích hợp với VS Code

Cài extension **WSL** (hoặc **Remote - WSL**) trong VS Code, sau đó:

```bash
# Mở thư mục hiện tại trong VS Code từ WSL terminal
code .

# Mở file cụ thể
code myfile.py
```

## Một số mẹo hữu ích

### Tự động mount drive

Thêm vào `/etc/fstab` trong WSL:

```
D: /mnt/d drvfs defaults 0 0
```

### Alias tiện dụng

Thêm vào `~/.bashrc` hoặc `~/.zshrc`:

```bash
# Nhanh chóng cd vào thư mục Windows
alias cdwin='cd /mnt/c/Users/<YourUsername>'
alias cdproject='cd /mnt/d/Projects'

# Mở explorer tại thư mục hiện tại
alias open='explorer.exe .'

# Xem IP WSL
alias myip='hostname -I'
```

### Giải phóng dung lượng disk (compact VHD)

```powershell
# Trong PowerShell (Admin)
wsl --shutdown
Optimize-VHD -Path "C:\Users\<YourUsername>\AppData\Local\Packages\<DistroPackage>\LocalState\ext4.vhdx" -Mode Full
```

Hoặc dùng `diskpart`:

```powershell
wsl --shutdown
diskpart
select vdisk file="C:\Users\<YourUsername>\AppData\Local\Packages\<DistroPackage>\LocalState\ext4.vhdx"
compact vdisk
exit
```

## Troubleshooting

### Lỗi "WslRegisterDistribution failed with error: 0x80370102"

Nguyên nhân: Chưa bật virtualization trong BIOS. Vào BIOS và bật **Intel VT-x** hoặc **AMD-V**.

### Lỗi DNS không resolve được

```bash
# Tạo file resolv.conf thủ công
sudo rm /etc/resolv.conf
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo bash -c 'echo "nameserver 8.8.4.4" >> /etc/resolv.conf'

# Ngăn WSL tự ghi đè resolv.conf
# Thêm vào /etc/wsl.conf:
# [network]
# generateResolvConf=false
```

### WSL dùng quá nhiều RAM

Cấu hình giới hạn RAM trong `.wslconfig`:

```ini
[wsl2]
memory=4GB
swap=2GB

[experimental]
autoMemoryReclaim=dropcache
```

### Lỗi permission khi truy cập file Windows

```bash
# Mount lại với metadata
sudo umount /mnt/c
sudo mount -t drvfs C: /mnt/c -o metadata
```

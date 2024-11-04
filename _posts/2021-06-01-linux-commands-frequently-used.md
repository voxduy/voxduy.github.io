---
title: Linux Commands - Frequently Used
description: This is a collection of linux commands that I have ever been. And I still keep it up-to-date currently.
author: voxduy
date: 2021-06-01 02:00:00 +0700
categories: [Linux]
tags: [linux]
image:
  path: /posts/2021-06-01-linux-commands-frequently-used/linux.png
  width: 800
  height: 500
pin: true
---

`man <cmd>` >>> khi cần gợi ý cú pháp trên linux

`clear` >>> làm sạch cửa sổ dòng lệnh

`ls -lah` >>> tenthumuc: Liệt kê nội dung bên trong một thư mục

`cat` >>> tentaptin: Hiển thị nội dung của một tập tin lên cửa sổ dòng lệnh

`rm` >>> tentaptin: Xóa một tập tin

`cp` >>> taptinnguon taptindich: Sao chép một tập tin

`passwd` >>> Đổi mật khẩu

`less tentaptin` hoặc `more tentaptin` >>> Hiển thị nội dung một tập tin trong cửa sổ dòng lệnh một trang mỗi lần

`grep chuoi tentaptin` >>> Tìm kiếm chuỗi trong tập tin

`head tentaptin` >>> Hiển thị 10 dòng đầu tiên của tập tin

`tail tentaptin` >>> Hiển thị 10 dòng cuối cùng của tập tin

`mv tentaptincu tentaptinmoi` >>> Di chuyển hoặc đổi tên tập tin

`file tentaptin` >>> Hiển thị thông tin về nội dung của tập tin

`echo chuoi` >>> Sao chép chuỗi tới màn hình dòng lệnh

`date` >>> Hiển thị ngày và giờ hiện tại

`gzip tentaptin` >>> Nén một tập tin

`gunzip tentaptin` >>> Giải nén một tập tin

`chmod quyen tentaptin` >>> Thay đổi quyền truy cập tập tin

`mkdir tenthumuc` >>> Tạo một thư mục

`rmdir tenthumuc` >>> Xóa một thư mục rỗng

`ln existingfile new-link` >>> Tạo một đường dẫn tới một tập tin (liên kết cứng)

`top` >>> Hiển thị danh sách các tiến trình đang chạy

copy toàn bộ thư mục

```bash
scp -r admin@69.69.69.69:/home/admin/<folder_name> /home/duyvn/
```

Add user & grant quyền sudo cho user

```bash
useradd -m -d /home/duyvn -s /bin/bash -G sudo duyvn
> tạo một người dùng mới có tên duyvn, với thư mục home tại /home/duyvn, sử dụng shell bash, và có quyền quản trị (là thành viên của nhóm sudo)

`passwd duyvn`
> change password cho account duyvn

`visudo`
> edit file

duyvn  ALL=(ALL:ALL) ALL
> cho phép người dùng duyvn chạy mọi lệnh với quyền root hoặc quyền của bất kỳ người dùng nào khác trên hệ thống

usermod -aG sudo duyvn
> thêm người dùng `duyvn` vào nhóm `sudo`, đồng thời giữ nguyên tất cả các nhóm mà người dùng này đã thuộc về trước đó. Sau khi thực hiện lệnh này, `duyvn` sẽ có quyền thực hiện các lệnh với quyền của người dùng root thông qua lệnh `sudo`
```

Tìm kiếm trên hệ thống có tên chứa chuỗi ký tự "docker" ở bất kỳ đâu trong tên tệp hoặc thư mục

```bash
sudo find / -name \*docker*
```

Check status or restart DNS

```bash
sudo systemctl status|restart systemd-resolved.service
```

Tạo file với dung lượng bất kì

```bash
truncate -s 20G file
ls -lh file
```
> Specifically, we use the -s argument to represent the size of the file in bytes.

Check disk

```bash
df -ah
> hiển thị thông tin về dung lượng đĩa của tất cả các hệ thống theo định dạng human-readable

fdisk -l
> liệt kê toàn bộ thông tin về các ổ đĩa và các phân vùng của chúng trên hệ thống

du -sch /DATA/*
> hiển thị tổng dung lượng của tất cả các thư mục hoặc tệp tin trong một thư mục cụ thể

cat /etc/fstab
> xem các thiết bị và phân vùng sẽ được tự động gắn kết khi hệ thống khởi động
```

Tool for monitoring health server Ubuntu or RHEL

```bash
apt-get install nmon
dnf install nmon
```

compress and extract

```bash
tar -czvf myproject.tar.gz /path/to/your/myproject

tar -xzvf myproject.tar.gz -C /path/to/destination
```

Measure!

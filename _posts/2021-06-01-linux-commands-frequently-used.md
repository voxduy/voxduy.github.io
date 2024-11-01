---
title: Linux Commands - Frequently Used
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

This is a collection of linux commands that I have ever been. And I still keep it up-to-date currently.

1. `man <cmd>` >>> khi cần gợi ý cú pháp trên linux
2. `clear` >>> làm sạch cửa sổ dòng lệnh
3. `ls -lah` >>> tenthumuc: Liệt kê nội dung bên trong một thư mục
4. `cat` >>> tentaptin: Hiển thị nội dung của một tập tin lên cửa sổ dòng lệnh
5. `rm` >>> tentaptin: Xóa một tập tin
6. `cp` >>> taptinnguon taptindich: Sao chép một tập tin
7. `passwd` >>> Đổi mật khẩu
8. `less tentaptin` hoặc `more tentaptin` >>> Hiển thị nội dung một tập tin trong cửa sổ dòng lệnh một trang mỗi lần
9. `grep chuoi tentaptin` >>> Tìm kiếm chuỗi trong tập tin
10. `head tentaptin` >>> Hiển thị 10 dòng đầu tiên của tập tin
11. `tail tentaptin` >>> Hiển thị 10 dòng cuối cùng của tập tin
12. `mv tentaptincu tentaptinmoi` >>> Di chuyển hoặc đổi tên tập tin
13. `file tentaptin` >>> Hiển thị thông tin về nội dung của tập tin
14. `echo chuoi` >>> Sao chép chuỗi tới màn hình dòng lệnh
15. `date` >>> Hiển thị ngày và giờ hiện tại
16. `gzip tentaptin` >>> Nén một tập tin
17. `gunzip tentaptin` >>> Giải nén một tập tin
18. `chmod quyen tentaptin` >>> Thay đổi quyền truy cập tập tin
19. `mkdir tenthumuc` >>> Tạo một thư mục
20. `rmdir tenthumuc` >>> Xóa một thư mục rỗng
21. `ln existingfile new-link` >>> Tạo một đường dẫn tới một tập tin (liên kết cứng)
22. `top` >>> Hiển thị danh sách các tiến trình đang chạy
23. `scp -r admin@69.69.69.69:/home/admin/<folder_name> /home/duyvn/` >>> copy toàn bộ thư mục

24. Add user & grant quyền sudo cho user

  `useradd -m -d /home/duyvn -s /bin/bash -G sudo duyvn`
  `passwd duyvn`

  `visudo` <<< edit file
  `duyvn  ALL=(ALL:ALL) ALL`

  `usermod -aG sudo duyvn`  
  (thêm người dùng `duyvn` vào nhóm `sudo`, đồng thời giữ nguyên tất cả các nhóm mà người dùng này đã thuộc về trước đó. Sau khi thực hiện lệnh này, `duyvn` sẽ có quyền thực hiện các lệnh với quyền của người dùng root thông qua lệnh `sudo`)

25. `sudo find / -name \*docker*`

26. `sudo systemctl status|restart systemd-resolved.service` <<< Check status or restart DNS

27. Tạo file với dung lượng bất kì  
  `truncate -s 20G file`
  `ls -lh file`  
  Specifically, we use the -s argument to represent the size of the file in bytes.

28.  `df -ah` <<< check disk free

29. `fdisk -l` <<< liệt kê toàn bộ thông tin về các ổ đĩa và các phân vùng của chúng trên hệ thống

30. `du -sch /DATA/*` <<< hiển thị tổng dung lượng của tất cả các thư mục hoặc tệp tin trong một thư mục cụ thể

31. `cat /etc/fstab` <<< xem các thiết bị và phân vùng sẽ được tự động gắn kết khi hệ thống khởi động

32. Tool for monitoring health server Ubuntu or RHEL
  `apt-get install nmon`
  `dnf install nmon`

33. compress and extract

```
tar -czvf myproject.tar.gz /path/to/your/myproject

tar -xzvf myproject.tar.gz -C /path/to/destination
```

Measure!

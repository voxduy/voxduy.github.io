---
title: Spoofed SYN Flood Prevention trên Arbor APS
author: voxduy
date: 2025-07-06 00:45:00 +0700
categories: [Network, DDoS]
tags: [Network, DDoS, Security]
image:
  path: /posts/2025-07-06-introduce-spoofed-syn-flood-prevention-arbor-aps/syn_flood_attack.png
  width: 800
  height: 500
pin: false
---

## **Spoofed SYN Flood** là gì?

- Attackers gửi hàng trăm‑ngàn gói **TCP SYN** với địa chỉ nguồn bị **giả mạo (spoofed)**.
- Máy chủ phía sau AED chờ gói **ACK** để hoàn tất 3‑way‑handshake nhưng sẽ **không bao giờ nhận được**, bảng kết nối (back‑log) nhanh chóng cạn kiệt → từ chối dịch vụ.
- Mỗi “bot” trong cuộc tấn công chỉ gửi vài SYN nên **ngưỡng đếm tốc độ (rate‑based)** thông thường khó phát hiện; phải xác thực được ai là IP thật.

## Cơ chế **Spoofed SYN Flood Prevention** trên Arbor APS

| Bước | Hành động của Arbor APS                                                                                         | Mục đích                                           |
| ---- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| 1    | Nhận gói **SYN** hướng vào một host được bảo vệ.                                                                | Chặn trước khi SYN đến server.                     |
| 2    | Gửi lại **ACK** (không kèm SYN) với số SEQ đặc biệt → **“TCP authentication/ACK‑challenge”**.                   | Buộc đầu bên gửi SYN phải chứng minh mình “sống”.  |
| 3a   | Nếu IP là thật: host thấy ACK bất ngờ ⇒ trả về **RST**/ACK thích hợp → AED **xác thực** IP, chuyển SYN gốc (hoặc yêu cầu host gửi SYN mới) tới máy chủ. | Cho phép lưu lượng hợp lệ, chỉ thêm 1 RTT nhỏ.                      |
| 3b   | Nếu IP bị giả mạo: **không có RST** quay lại ⇒ AED loại bỏ SYN, chặn phiên kết nối.                            | Bỏ spoofed SYN trước khi đến server, không tốn tài nguyên bảng kết nối.  |

## Vì sao nên kết hợp giữa **TCP SYN Flood Detection** và **Spoofed SYN Flood Prevention**

## Nói thêm về **TCP SYN Cookie**

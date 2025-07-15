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

## Spoofed SYN flood là gì?

- Attackers gửi hàng trăm‑ngàn gói **TCP SYN** với địa chỉ nguồn bị **giả mạo (spoofed)**.
- Máy chủ phía sau AED chờ gói **ACK** để hoàn tất 3‑way‑handshake nhưng sẽ **không bao giờ nhận được**, bảng kết nối (back‑log) nhanh chóng cạn kiệt → từ chối dịch vụ.
- Mỗi “bot” trong cuộc tấn công chỉ gửi vài SYN nên **ngưỡng đếm tốc độ (rate‑based)** thông thường khó phát hiện; phải xác thực được **ai là IP thật**.

## Cơ chế “Spoofed SYN Flood Prevention” trên Arbor APS

## Vì sao nên kết hợp giữa TCP SYN Flood Detection và Spoofed SYN Flood Prevention

## Nói thêm về TCP SYN Cookie

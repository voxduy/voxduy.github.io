---
title: Giải thích về cơ chế tấn công DDoS - SSDP Amplification
author: voxduy
date: 2021-04-30 15:00:00 +0700
categories: [Network, DDoS]
tags: [Network, DDoS, Security]
image:
  path: /posts/2021-04-30-how-does-a-ssdp-attack-work/ssdp_amplification.png
  width: 800
  height: 500
pin: false
---

## Giới thiệu giao thức SSDP

SSDP (Simple Service Discovery Protocol) là một giao thức thuộc lớp ứng dụng trong mô hình OSI, sử dụng để tìm kiếm các dịch vụ và thiết bị trên mạng IP cục bộ. SSDP là một phần của kiến trúc UPnP (Universal Plug and Play), được sử dụng rộng rãi để tự động phát hiện các thiết bị mạng mà không cần cấu hình thủ công.

Chi tiết về giao thức SSDP
- Chức năng chính: SSDP cho phép các thiết bị và dịch vụ tự động tìm thấy và được phát hiện trên mạng cục bộ mà không cần sự can thiệp của người dùng. Điều này giúp cho các thiết bị, như máy in, máy chủ đa phương tiện, router, và các thiết bị IoT (Internet of Things) khác có thể "tự giới thiệu" hoặc phát hiện nhau để cung cấp và sử dụng các dịch vụ qua mạng.
- Giao thức vận chuyển: SSDP hoạt động dựa trên UDP (User Datagram Protocol), cụ thể là sử dụng cổng 1900. Các gói tin SSDP được gửi qua multicast tới địa chỉ IP đặc biệt (239.255.255.250) trong mạng cục bộ để truyền đạt các yêu cầu hoặc thông báo.

## Giải thích về cơ chế tấn công DDoS - SSDP Amplification



## Wrapping up



## References

- [How does a SSDP Attack work?](https://www.cloudflare.com/learning/ddos/ssdp-ddos-attack/)

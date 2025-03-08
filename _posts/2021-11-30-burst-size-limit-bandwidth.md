---
title: Hiểu thế nào về burst-size-limit trong QoS hoặc limit bandwidth
description: Ý nghĩa của Burst-size-limit | Cách hoạt động | Nếu không có Burst-size-limit thì điều gì xảy ra?
author: voxduy
date: 2021-11-30 02:00:00 +0700
categories: [Network, General]
tags: [Network, Juniper]
image:
  path: /posts/2021-11-30-burst-size-limit-bandwidth/optical_fiber.avif
  width: 800
  height: 500
pin: false
---

**Burst-size-limit** trong **QoS (Quality of Service)** hoặc limit bandwidth là một tham số quan trọng xác định lượng dữ liệu tối đa mà một luồng có thể truyền vượt quá tốc độ giới hạn bình thường trong một khoảng thời gian ngắn.

### **Ý nghĩa của Burst-size-limit**

- Khi áp dụng giới hạn băng thông (rate limit), lưu lượng mạng thường bị giới hạn ở một tốc độ cố định, ví dụ **10 Mbps**.
- Tuy nhiên, đôi khi có thể có những đợt tăng đột biến (burst) của lưu lượng hợp lệ. **Burst-size-limit** cho phép lưu lượng có thể vượt quá giới hạn tốc độ trong một thời gian ngắn mà không bị drop ngay lập tức.
- Nếu burst vượt quá giới hạn này, gói tin sẽ bị hàng đợi hoặc bị drop tùy theo cơ chế kiểm soát băng thông.

### **Cách hoạt động**

- Hệ thống thường sử dụng **Token Bucket** hoặc **Leaky Bucket** để quản lý lưu lượng.
- Khi không có dữ liệu truyền, token (tín dụng băng thông) sẽ tích lũy lại.
- Khi có dữ liệu cần truyền, nếu tốc độ vượt quá giới hạn bình thường nhưng vẫn nằm trong **burst-size-limit**, dữ liệu sẽ được truyền mà không bị hạn chế ngay lập tức.
- Nếu lượng dữ liệu vượt quá cả **burst-size-limit**, các gói tin sẽ bị drop hoặc bị xếp hàng chờ.

    **Ví dụ**  
    Giả sử một chính sách QoS giới hạn băng thông ở **10 Mbps** nhưng cho phép burst **1 MB**. Điều này có nghĩa:
      - Dữ liệu có thể truyền ở tốc độ **hơn 10 Mbps** trong một khoảng thời gian ngắn miễn là không vượt quá **1 MB**.
      - Nếu lượng dữ liệu cần truyền vượt quá 1 MB, tốc độ sẽ bị giới hạn trở lại 10 Mbps hoặc gói tin bị drop.

### **Ứng dụng thực tế**

- **Trong Open vSwitch (OVS)**, khi cấu hình QoS, có thể đặt `burst` để điều chỉnh cách xử lý băng thông đột biến.
- **Trong Linux Traffic Control (tc)**, tham số `burst` giúp tối ưu hóa giới hạn băng thông.
- **Trong Router/Switch của Cisco, Juniper**, tham số tương tự có thể xuất hiện dưới dạng `burst-size` trong các chính sách QoS.

### **So sánh với các tham số khác**

| Tham số | Ý nghĩa |
|---------|--------|
| **Rate Limit (tốc độ giới hạn)** | Giới hạn băng thông tối đa (ví dụ: 10 Mbps) |
| **Burst-size-limit** | Cho phép vượt tốc độ giới hạn trong ngắn hạn (ví dụ: tối đa 1 MB) |
| **Policer vs Shaper** | Policer thường drop gói tin khi vượt ngưỡng, Shaper thì xếp hàng chờ |

### **Nếu không có burst-size-limit thì điều gì xảy ra?**

Nếu không có Burst-size-limit sẽ làm cho việc kiểm soát băng thông trở nên khắc nghiệt hơn, gây ra mất gói tin, tăng độ trễ, và giảm hiệu suất mạng. Cụ thể:

1. Lưu lượng bị giới hạn ngay lập tức (No Burst Allowed)
    - Bất kỳ gói tin nào vượt quá **tốc độ giới hạn** (**Rate Limit**) đều **bị drop ngay lập tức** mà không có bất kỳ vùng "cho phép vượt quá" nào.
    - Điều này có thể gây ra hiện tượng **packet loss** nghiêm trọng khi có các đợt lưu lượng đột biến hợp lệ.

    **Ví dụ**:  
        - Giới hạn **50 Mbps** trên một link mạng.  
        - Một ứng dụng cần gửi **55 Mbps** trong một khoảnh khắc ngắn.  
        - Nếu không có Burst-size-limit, **5 Mbps vượt quá** sẽ bị drop ngay lập tức thay vì được tạm thời chấp nhận.

2. Hiệu suất kém đối với các ứng dụng có lưu lượng biến động
    - Các ứng dụng truyền dữ liệu theo từng đợt (bursty traffic) như **web browsing, video streaming, VoIP, gaming** sẽ gặp vấn đề.
    - Không có burst nghĩa là dữ liệu chỉ có thể truyền đi **nếu ngay lập tức tuân thủ giới hạn**, gây ra độ trễ cao hơn hoặc mất kết nối.

    **Ví dụ**:  
        - Khi tải một trang web, trình duyệt có thể yêu cầu nhiều tài nguyên đồng thời, tạo ra một burst tạm thời.  
        - Không có burst-size-limit, nhiều gói tin có thể bị drop, làm trang web tải chậm hơn.

3. Tăng chi phí xử lý do quá nhiều gói tin bị drop
    - Khi các gói tin bị drop, **TCP sẽ kích hoạt cơ chế điều chỉnh lại tốc độ** (TCP Congestion Control).  
    - Điều này gây **giảm tốc độ mạng tổng thể**, tạo ra độ trễ lớn hơn trong hệ thống.  
    - Đối với UDP (không có cơ chế kiểm soát lại), có thể gây **mất gói tin vĩnh viễn**.

4. Giảm hiệu quả sử dụng băng thông
    - Burst-size-limit giúp tối ưu hóa việc sử dụng băng thông trong thời gian ngắn mà không làm tắc nghẽn hệ thống.  
    - Nếu không có burst, có thể xảy ra tình huống băng thông bị "cắt vụn" (fragmentation), làm giảm hiệu suất tổng thể.

    **Ví dụ**:  
        - Một luồng dữ liệu có thể tạm thời dùng 55 Mbps trong 1 ms, nhưng do bị drop ngay lập tức, nó chỉ có thể dùng đúng 50 Mbps cố định, dẫn đến việc lãng phí tài nguyên.

### Ví dụ về cách tính burst-size-limit trên Router Juniper

Khi thực hiện giới hạn băng thông trên router Juniper, Có thể tính toán giá trị **burst-size-limit** dựa trên thời gian burst được phép và tốc độ băng thông tối đa.

Công thức tính:

\[
\text{Burst Size} = \text{Rate Limit} \times \text{Allowed Burst Time}
\]

Trong đó:
    - **Burst Size (bytes)**: Lượng dữ liệu tối đa có thể truyền vượt quá rate limit trong một khoảng thời gian ngắn.
    - **Rate Limit (bps)**: Giới hạn băng thông mong muốn (ở đây là 50 Mbps).
    - **Allowed Burst Time (giây)**: Thời gian cho phép vượt quá tốc độ giới hạn trước khi hệ thống can thiệp.

**Giả sử**: cho phép burst trong **1ms (0.001 giây)**.

\[
\text{Burst Size} = 50 \times 10^6 \times 0.001
\]

\[
= 50,000 \text{ bytes} = 50 KB
\]

**Cấu hình trên Router Juniper:**

- Có thể áp dụng trên **class-of-service (CoS)** bằng cách sử dụng `policer` để giới hạn băng thông:

```bash
set firewall policer LIMIT-50M if-exceeding bandwidth-limit 50m
set firewall policer LIMIT-50M if-exceeding burst-size-limit 50k
set firewall policer LIMIT-50M then discard
```

`bandwidth-limit 50m`: Giới hạn băng thông ở 50 Mbps.
`burst-size-limit 50k`: Cho phép tối đa **50 KB dữ liệu** được gửi vượt quá giới hạn trong khoảng thời gian ngắn.
`then discard`: Nếu vượt quá cả burst-size-limit, gói tin sẽ bị drop.

**Tùy chỉnh Burst-size-limit:**

- Nếu **muốn giảm độ trễ**, giảm `burst-size-limit` (ví dụ: 25 KB).
- Nếu **muốn tránh drop gói tin**, tăng `burst-size-limit` (ví dụ: 100 KB, tương ứng với 2ms).

| Thời gian Burst | Burst-size-limit |
|----------------|----------------|
| 0.5ms | 25 KB |
| 1ms | 50 KB |
| 2ms | 100 KB |

Có thể điều chỉnh giá trị này tùy theo yêu cầu thực tế và tình trạng mạng.

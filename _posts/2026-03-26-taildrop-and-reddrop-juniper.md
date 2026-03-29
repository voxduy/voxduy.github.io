---
title: TailDrop và RedDrop trong QoS của Juniper
author: voxduy
date: 2026-03-26 08:00:00 +0700
categories: [Network, General]
tags: [Network, Juniper]
#image:
#  path: /posts/2026-03-11-snmpwalk-vs-snmpbulkwalk/snmpwalk_vs_snmpbulkwalk.png
#  width: 800
#  height: 500
pin: false
---

## TailDrop

TailDrop là cơ chế drop packet đơn giản nhất: khi queue đầy 100%, mọi packet đến sau đều bị drop cho đến khi queue có chỗ trống.

**Cách hoạt động:** Mỗi queue trên Juniper có một buffer size nhất định (tính bằng bytes hoặc percentage of shared buffer). Khi lượng packet trong queue chạm ngưỡng maximum, packet mới đến sẽ bị drop ngay tại "đuôi" queue — không có bất kỳ logic chọn lọc nào.

**Vấn đề chính của TailDrop:**

**TCP Global Synchronization** — đây là lý do chính khiến TailDrop không phù hợp cho môi trường nhiều TCP flow. Khi queue đầy, tất cả TCP flow bị drop cùng lúc → tất cả cùng giảm window → throughput tụt mạnh → queue trống → tất cả cùng tăng window lại → queue lại đầy. Kết quả là bandwidth dao động kiểu "sóng", utilization không bao giờ ổn định.

Trên Juniper, TailDrop là behavior mặc định khi bạn không cấu hình RED profile. Bạn có thể tune buffer size qua `scheduler` trong `class-of-service`:

```bash
class-of-service {
    schedulers {
        my-scheduler {
            buffer-size percent 10;
            transmit-rate percent 20;
        }
    }
}
```

Khi buffer đầy 100% → TailDrop kick in.

---

## RED (Random Early Detection) — RedDrop trên Juniper

RED là cơ chế **chủ động drop packet trước khi queue đầy**, dựa trên xác suất tăng dần theo mức fill level của queue.

**Nguyên lý hoạt động:**

RED hoạt động dựa trên 2 ngưỡng chính:

- **Fill level thấp (ví dụ 0–50%)**: drop probability = 0%, không drop gì cả.
- **Fill level từ ngưỡng bắt đầu đến ngưỡng tối đa (ví dụ 50–80%)**: drop probability tăng tuyến tính từ 0% đến một giá trị max (ví dụ 100%).
- **Fill level trên ngưỡng tối đa (>80%)**: drop 100% — tương đương TailDrop.

Bằng cách drop sớm và ngẫu nhiên, các TCP flow bị ảnh hưởng **lệch pha** nhau — flow nào gửi nhiều thì xác suất bị drop cao hơn → giảm window trước → tránh được hiện tượng global synchronization.

**Cấu hình RED trên Juniper (drop-profile):**

```bash
class-of-service {
    drop-profiles {
        my-red-profile {
            fill-level 50 drop-probability 0;
            fill-level 80 drop-probability 50;
            fill-level 100 drop-probability 100;
        }
    }
    schedulers {
        my-scheduler {
            drop-profile-map loss-priority low protocol any drop-profile my-red-profile;
        }
    }
}
```

Ở đây bạn thấy Juniper cho phép map drop-profile theo **loss-priority** (low/medium-low/medium-high/high). Đây là điểm mạnh: packet đã được classify với loss-priority high (ví dụ qua BA classifier hoặc firewall filter set PLP) sẽ dùng profile aggressive hơn — drop sớm hơn, probability cao hơn.

**Ví dụ thực tế:** Bạn có thể tạo 2 profile:

```bash
drop-profiles {
    red-gentle {
        fill-level 70 drop-probability 0;
        fill-level 95 drop-probability 50;
    }
    red-aggressive {
        fill-level 30 drop-probability 0;
        fill-level 80 drop-probability 80;
    }
}
schedulers {
    data-scheduler {
        drop-profile-map loss-priority low protocol any drop-profile red-gentle;
        drop-profile-map loss-priority high protocol any drop-profile red-aggressive;
    }
}
```

Traffic có PLP high sẽ bắt đầu bị drop từ fill-level 30% với probability leo rất nhanh, trong khi PLP low được "ưu ái" hơn nhiều.

---

## So sánh nhanh

| Tiêu chí | TailDrop | RED |
|---|---|---|
| Thời điểm drop | Queue đầy 100% | Trước khi đầy, theo xác suất |
| Chọn lọc | Không — drop tất cả | Có — dựa trên fill-level + loss-priority |
| TCP Synchronization | Gây ra | Giảm thiểu |
| Buffer utilization | Kém (dao động lớn) | Tốt hơn (ổn định hơn) |
| Cấu hình Juniper | Mặc định (không cần cấu hình) | Cần tạo `drop-profile` + map vào scheduler |

**Trong môi trường DC scale lớn như của bạn**, RED profile nên được cấu hình ít nhất cho các queue best-effort và data queue, đặc biệt trên các uplink từ leaf lên spine/super-spine nơi mà congestion dễ xảy ra nhất. Các queue cho control-plane traffic (OSPF, BGP, BFD) thì thường dùng strict-high priority và không cần RED vì bạn muốn đảm bảo không drop control packet.

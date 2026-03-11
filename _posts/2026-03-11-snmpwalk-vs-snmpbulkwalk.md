---
title: So sánh chi tiết snmpwalk và snmpbulkwalk
author: voxduy
date: 2026-03-11 08:00:00 +0700
categories: [Network, General]
tags: [Network, SNMP]
image:
  path: /posts/2026-03-11-snmpwalk-vs-snmpbulkwalk/snmpwalk_vs_snmpbulkwalk.png
  width: 800
  height: 500
pin: false
---

## Giới thiệu

SNMP (Simple Network Management Protocol) là giao thức phổ biến dùng để giám sát và quản lý thiết bị mạng. Trong bộ công cụ `net-snmp`, hai lệnh **snmpwalk** và **snmpbulkwalk** thường được sử dụng để truy xuất thông tin từ thiết bị. Tuy nhiên, nhiều người dùng chưa hiểu rõ sự khác biệt và khi nào nên dùng lệnh nào.

---

## snmpwalk là gì?

`snmpwalk` sử dụng thao tác **GETNEXT** của SNMP để duyệt tuần tự qua các OID trong MIB tree. Mỗi lần chỉ lấy một OID, sau đó dùng kết quả đó để lấy OID tiếp theo, lặp lại cho đến khi ra khỏi vùng OID được chỉ định.

### Cú pháp

```bash
snmpwalk -v <version> -c <community> <host> <OID>
```

### Ví dụ

```bash
snmpwalk -v2c -c public 192.168.1.1 1.3.6.1.2.1.2.2
```

### Cách hoạt động

```
Client  --GETNEXT(OID_1)-->  Agent
Client  <-- OID_2, value --  Agent
Client  --GETNEXT(OID_2)-->  Agent
Client  <-- OID_3, value --  Agent
...
```

Mỗi vòng là một request/response riêng biệt, dẫn đến nhiều round-trip nếu MIB tree có nhiều OID.

---

## snmpbulkwalk là gì?

`snmpbulkwalk` sử dụng thao tác **GETBULK** được giới thiệu từ **SNMPv2c**. Thay vì lấy từng OID một, GETBULK cho phép lấy nhiều OID trong một request duy nhất, giảm đáng kể số lần trao đổi giữa client và agent.

### Cú pháp

```bash
snmpbulkwalk -v <version> -c <community> <host> <OID>
```

### Ví dụ

```bash
snmpbulkwalk -v2c -c public 192.168.1.1 1.3.6.1.2.1.2.2
```

### Cách hoạt động

```
Client  --GETBULK(OID_1, max-repetitions=10)-->  Agent
Client  <-- OID_2..OID_11, values --             Agent
Client  --GETBULK(OID_11, max-repetitions=10)--> Agent
...
```

Tham số quan trọng của GETBULK:
- **non-repeaters**: Số OID scalar (không lặp) cần lấy trước.
- **max-repetitions**: Số lượng OID tối đa lấy trong mỗi lần GETBULK.

Mặc định `snmpbulkwalk` dùng `max-repetitions=10`. Có thể tùy chỉnh bằng flag `-Cr`:

```bash
snmpbulkwalk -v2c -c public -Cr20 192.168.1.1 1.3.6.1.2.1.2.2
```

---

## Bảng so sánh

| Tiêu chí | snmpwalk | snmpbulkwalk |
|---|---|---|
| Thao tác SNMP | GETNEXT | GETBULK |
| Phiên bản SNMP hỗ trợ | v1, v2c, v3 | v2c, v3 (không hỗ trợ v1) |
| Số request/response | Nhiều (1 OID/request) | Ít hơn (nhiều OID/request) |
| Hiệu năng | Chậm hơn với MIB lớn | Nhanh hơn đáng kể |
| Tải mạng | Cao hơn | Thấp hơn |
| Phù hợp | Debug, MIB nhỏ, SNMPv1 | Production, MIB lớn, polling định kỳ |
| Độ phức tạp | Đơn giản | Cần hiểu max-repetitions |

---

## Khi nào dùng snmpwalk?

- Thiết bị chỉ hỗ trợ **SNMPv1**.
- Debug, kiểm tra nhanh giá trị OID cụ thể.
- MIB tree nhỏ, không cần tối ưu hiệu năng.
- Khi cần kết quả dễ đọc, tuần tự rõ ràng.

## Khi nào dùng snmpbulkwalk?

- Thu thập dữ liệu từ MIB tree **lớn** (interface table, routing table...).
- Polling định kỳ trong hệ thống **giám sát mạng** (NMS, Zabbix, Prometheus...).
- Môi trường có **băng thông hạn chế** hoặc **độ trễ cao**.
- Cần giảm tải CPU cho SNMP agent.

---

## Ví dụ thực tế: So sánh thời gian thực thi

Giả sử cần lấy toàn bộ interface table (`IF-MIB::ifTable`) trên một router có 48 interface:

```bash
# Dùng snmpwalk
time snmpwalk -v2c -c public 192.168.1.1 IF-MIB::ifTable

# Dùng snmpbulkwalk
time snmpbulkwalk -v2c -c public 192.168.1.1 IF-MIB::ifTable
```

Kết quả điển hình:
- `snmpwalk`: ~3-5 giây (hàng trăm GETNEXT request)
- `snmpbulkwalk`: ~0.5-1 giây (vài chục GETBULK request)

---

## Lưu ý khi dùng snmpbulkwalk

- Không dùng được với **SNMPv1** — agent sẽ trả về lỗi.
- Giá trị `max-repetitions` quá lớn có thể gây **phân mảnh UDP** hoặc làm agent quá tải.
- Một số thiết bị cũ có thể không xử lý GETBULK đúng cách — hãy test trước khi deploy.
- Nếu agent trả về lỗi `tooBig`, hãy giảm `-Cr` xuống (ví dụ: `-Cr5`).

---

## Kết luận

Cả `snmpwalk` và `snmpbulkwalk` đều là công cụ hữu ích trong công việc quản trị mạng. Tóm lại:

- Dùng **snmpwalk** khi cần debug đơn giản hoặc làm việc với thiết bị SNMPv1.
- Dùng **snmpbulkwalk** trong môi trường production, đặc biệt khi polling MIB lớn thường xuyên.

Hiểu rõ sự khác biệt giúp bạn chọn đúng công cụ, tiết kiệm tài nguyên mạng và cải thiện hiệu năng hệ thống giám sát.

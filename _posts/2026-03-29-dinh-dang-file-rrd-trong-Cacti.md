---
title: RRD (Round-Robin Database) — Chi tiết từ A đến Z
author: voxduy
date: 2026-03-29 08:00:00 +0700
categories: [Network, General]
tags: [Network, System]
#image:
#  path: /posts/2026-03-11-snmpwalk-vs-snmpbulkwalk/snmpwalk_vs_snmpbulkwalk.png
#  width: 800
#  height: 500
pin: false
---

## Định nghĩa

RRD là định dạng database do **Tobias Oetiker** tạo ra (cùng tác giả MRTG), được quản lý bởi thư viện **RRDtool**. Đặc điểm cốt lõi: file có **kích thước cố định ngay từ lúc tạo** và không bao giờ phình ra — dữ liệu cũ tự động bị ghi đè theo vòng tròn (round-robin), nên nó hoạt động giống một circular buffer trên disk.

Cacti thực chất không tự lưu data — nó gọi `rrdtool update` để ghi và `rrdtool graph` để vẽ. Toàn bộ time-series data nằm trong các file `.rrd`.

---

## Cấu trúc bên trong một file RRD

Một file `.rrd` gồm 3 thành phần chính:

### 1. Data Source (DS)

Mỗi DS là một metric mà bạn thu thập. Ví dụ một interface Juniper thường có 2 DS: `traffic_in` và `traffic_out`.

Mỗi DS có các thuộc tính:

- **DST (Data Source Type):** `COUNTER`, `DERIVE`, `GAUGE`, `ABSOLUTE`
  - `COUNTER`: cho SNMP counter tăng dần (ifHCInOctets, ifHCOutOctets) — RRDtool tự tính rate (delta/interval). Tự xử lý counter wrap.
  - `GAUGE`: giá trị tức thời, không cần tính rate — ví dụ CPU %, temperature, queue depth.
  - `DERIVE`: giống COUNTER nhưng cho phép giá trị âm (ví dụ rate of change có thể giảm).
  - `ABSOLUTE`: counter reset về 0 sau mỗi lần đọc.
- **Heartbeat:** thời gian tối đa giữa 2 lần update trước khi RRDtool ghi `UNKNOWN`. Thường set = 2× polling interval (Cacti poll 300s → heartbeat 600s).
- **Min/Max:** giá trị hợp lệ, ngoài range → `UNKNOWN`.

### 2. Round-Robin Archive (RRA)

Đây là phần lưu trữ thực sự. Mỗi RRA định nghĩa **một mức độ chi tiết (resolution)** của data. Cấu trúc:

```bash
RRA:CF:xff:steps:rows
```

- **CF (Consolidation Function):** `AVERAGE`, `MAX`, `MIN`, `LAST` — cách gộp nhiều data point thành một.
- **xff (xfiles factor):** tỷ lệ UNKNOWN tối đa được chấp nhận khi consolidate (thường 0.5).
- **steps:** số primary data point gộp lại thành 1 consolidated data point.
- **rows:** số consolidated data point lưu trữ → quyết định lưu được bao lâu.

**Ví dụ thực tế** — một file RRD trong Cacti cho interface monitoring (polling 300s):

| RRA | steps | Resolution | rows | Lưu được |
| ----- | ------- | --------- | ------ | ---------- |
| RRA:AVERAGE:0.5:1:600 | 1 | 5 phút | 600 | ~2 ngày |
| RRA:AVERAGE:0.5:6:700 | 6 | 30 phút | 700 | ~14.5 ngày |
| RRA:AVERAGE:0.5:24:775 | 24 | 2 giờ | 775 | ~64 ngày |
| RRA:AVERAGE:0.5:288:797 | 288 | 1 ngày | 797 | ~2.2 năm |

Cùng lúc thường tạo thêm RRA cho `MAX` với cùng cấu trúc để lưu peak value.

→ File size = cố định = header + (tổng rows × số DS × 8 bytes). Một file với 2 DS, 8 RRA (4 AVERAGE + 4 MAX) ≈ **46 KB** — rất nhỏ.

### 3. Primary Data Point (PDP) và Consolidated Data Point (CDP)

- **PDP:** giá trị gốc được normalize về đúng step interval (300s). Nếu bạn update lệch vài giây, RRDtool sẽ interpolate.
- **CDP:** kết quả sau khi gộp nhiều PDP theo RRA definition.

---

## Xem cấu trúc thực tế

Trên con Cacti server, bạn có thể inspect bất kỳ file nào:

```bash
# Xem metadata
rrdtool info /var/lib/cacti/rra/some_device_traffic_in_123.rrd

# Dump ra XML để đọc toàn bộ
rrdtool dump /var/lib/cacti/rra/some_device_traffic_in_123.rrd | head -100

# Fetch data từ RRA
rrdtool fetch /var/lib/cacti/rra/some_device_traffic_in_123.rrd AVERAGE --start -3600
```

Output `rrdtool info` sẽ trả về kiểu:

```bash
ds[traffic_in].type = "COUNTER"
ds[traffic_in].minimal_heartbeat = 600
ds[traffic_in].min = 0.0000000000e+00
ds[traffic_in].max = 1.2500000000e+10    # 100Gbps in bytes
rra[0].cf = "AVERAGE"
rra[0].rows = 600
rra[0].pdp_per_row = 1
```

---

## Ưu và nhược điểm thực tế

**Ưu điểm:**

- Kích thước cố định, không cần lo disk growth.
- Tự động downsample data cũ — không cần cron job dọn dẹp.
- Read/write rất nhanh vì file nhỏ, memory-mapped I/O.

**Nhược điểm thực tế (bạn đã trải qua):**

- **Mỗi metric = 1 file** → 1000 device × 10 interface × 2 DS có thể tạo hàng chục ngàn file. Khi Cacti poll, nó `rrdtool update` từng file → disk I/O random write rất cao, đặc biệt trên HDD. Đây là lý do Cacti spine poller cần SSD.
- **Schema cứng:** thêm DS mới = phải tạo lại file (hoặc dùng `rrdtool tune` rất hạn chế). Khi bạn thay đổi Data Template trong Cacti (ví dụ thêm queue thứ 5 vào CoS monitoring), phải recreate RRD file → mất data cũ.
- **Không query được linh hoạt:** không có query language, không join, không aggregate cross-device. Muốn tính tổng traffic tất cả uplink MX960 → phải dùng `CDEF` trong graph template hoặc export ra tool khác.
- **Fixed resolution:** không thể retroactively thay đổi retention. Muốn giữ 5-minute resolution lâu hơn 2 ngày? Phải tạo lại file với rows lớn hơn.

---

## So sánh với các định dạng tương đương

| Đặc điểm | **RRD** (Cacti/MRTG) | **Whisper** (Graphite) | **TSDB** (Prometheus) | **InfluxDB TSM** |
| ----------- | ---------------------- | ------------------------ | ----------------------- | ------------------- |
| Storage model | 1 file/metric, fixed size, circular | 1 file/metric, fixed size, circular | Chunks trên disk, WAL + compaction | LSM-tree, WAL + TSM files |
| Kích thước | Cố định | Cố định | Tăng theo data, có compaction | Tăng theo data, compressed |
| Schema change | Phải tạo lại file | Phải tạo lại file | Schemaless (label-based) | Schemaless |
| Query language | Không có (chỉ fetch + CDEF) | Graphite functions | PromQL | InfluxQL / Flux |
| Downsampling | Built-in (RRA) | Built-in (retention policies) | Recording rules (manual) | Continuous queries / tasks |
| Write pattern | Random I/O (nhiều file nhỏ) | Random I/O (nhiều file nhỏ) | Sequential (WAL append) | Sequential (WAL + batch) |
| Cross-metric query | Không native | Có (wildcard, sumSeries) | Rất mạnh (PromQL) | Có (GROUP BY, JOIN) |
| Scale thực tế | ~10K-50K metrics OK | ~100K-500K | ~1M+ metrics | ~1M+ metrics |
| Typical tool | Cacti, MRTG, Nagios PNP | Graphite + Carbon | Prometheus + Grafana | InfluxDB + Grafana/Chronograf |

**Whisper** (Graphite) về bản chất rất giống RRD — cùng ý tưởng fixed-size circular file, cùng vấn đề I/O storm khi nhiều metric. Khác biệt chính: Whisper support multiple retention natively và có Graphite query functions phong phú hơn.

**Prometheus TSDB** là bước nhảy lớn — append-only WAL, data gom thành 2-hour blocks rồi compact. Không bị random I/O problem. PromQL cho phép query kiểu `sum(rate(ifHCInOctets{site="TTEPZ"}[5m]))` mà RRD không thể làm được.

---

## Ví dụ cụ thể: tạo và sử dụng RRD file

Giả sử bạn muốn monitor traffic 100G interface ngoài Cacti:

```bash
# Tạo RRD file: 2 DS (in/out), poll mỗi 300s
rrdtool create /tmp/interface_traffic.rrd \
  --step 300 \
  DS:traffic_in:COUNTER:600:0:12500000000 \
  DS:traffic_out:COUNTER:600:0:12500000000 \
  RRA:AVERAGE:0.5:1:600 \
  RRA:AVERAGE:0.5:6:700 \
  RRA:AVERAGE:0.5:24:775 \
  RRA:AVERAGE:0.5:288:797 \
  RRA:MAX:0.5:1:600 \
  RRA:MAX:0.5:6:700 \
  RRA:MAX:0.5:24:775 \
  RRA:MAX:0.5:288:797

# Max 12500000000 = 100Gbps in bytes/sec (100G × 1000^3 / 8)

# Update với giá trị SNMP counter
rrdtool update /tmp/interface_traffic.rrd N:98237465123:45123456789
# N = now, giá trị là raw counter từ ifHCInOctets và ifHCOutOctets

# Vẽ graph
rrdtool graph /tmp/traffic.png \
  --start -86400 --end now \
  --title "Interface Traffic - Last 24h" \
  DEF:in=/tmp/interface_traffic.rrd:traffic_in:AVERAGE \
  DEF:out=/tmp/interface_traffic.rrd:traffic_out:AVERAGE \
  CDEF:in_bits=in,8,* \
  CDEF:out_bits=out,8,* \
  AREA:in_bits#00CF00:"Inbound" \
  LINE1:out_bits#002A97:"Outbound"
```

Đây chính xác là những gì Cacti làm phía sau — nó generate RRDtool commands dựa trên Data Template, Graph Template, và kết quả SNMP poll.

---

## Summary

RRD là thiết kế rất thông minh cho thời điểm nó ra đời (1999) — giải quyết triệt để bài toán "monitor 24/7 mà disk không phình". Nhưng ở scale lớn, cần cross-device aggregation, cần flexible query, nó là bottleneck chính. Đó cũng là lý do cần triển khai các hệ thống Prometheus/Grafana stack bên cạnh Cacti — Prometheus TSDB giải quyết gần như tất cả các pain point mà RRD gây ra.

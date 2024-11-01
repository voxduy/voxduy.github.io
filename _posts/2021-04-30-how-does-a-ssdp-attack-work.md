---
title: Giải thích về cơ chế tấn công DDoS - SSDP Amplification
author: voxduy
date: 2021-04-30 15:00:00 +0700
categories: [Network, DDoS]
tags: [network, ddos, security]
image:
  path: /posts/2021-04-30-how-does-a-ssdp-attack-work/ddos_is_coming.png
  width: 800
  height: 500
pin: false
---

## Giới thiệu giao thức SSDP

- SSDP (Simple Service Discovery Protocol) là một giao thức thuộc lớp ứng dụng trong mô hình OSI, sử dụng để tìm kiếm các dịch vụ và thiết bị trên mạng IP cục bộ. SSDP là một phần của kiến trúc UPnP (Universal Plug and Play), được sử dụng rộng rãi để tự động phát hiện các thiết bị mạng mà không cần cấu hình thủ công.
- SSDP hoạt động dựa trên UDP (User Datagram Protocol), cụ thể là port 1900. Các gói tin SSDP được gửi qua multicast tới địa chỉ IP (239.255.255.250) trong mạng LAN để gửi các request hoặc notification.

Ứng dụng của giao thức SSDP:
- **Thiết bị gia đình thông minh**: SSDP là một phần quan trọng của UPnP và được sử dụng rộng rãi trong các hệ thống gia đình thông minh để kết nối và điều khiển các thiết bị như đèn, loa, TV và thiết bị bảo mật.
- **Truyền thông đa phương tiện**: SSDP thường được sử dụng trong các hệ thống truyền phát đa phương tiện để phát hiện các máy chủ hoặc thiết bị có khả năng stream nội dung.
- **Các ứng dụng IoT**: Nhiều thiết bị IoT sử dụng SSDP để dễ dàng tích hợp vào mạng lưới và giao tiếp với nhau mà không cần cấu hình phức tạp.

Brief summary cách hoạt động của SSDP:
1. Discovery
   - Các thiết bị mới gửi thông báo `NOTIFY` để thông báo sự hiện diện của chúng.
   - Thiết bị đang tìm kiếm dịch vụ gửi yêu cầu `M-SEARCH` để tìm kiếm các thiết bị và dịch vụ trong mạng.
   
2. Response
   - Các thiết bị đáp lại yêu cầu `M-SEARCH` với thông tin về dịch vụ mà chúng cung cấp thông qua phản hồi HTTP 200 OK.
   
3. Demanding
   - Thiết bị yêu cầu có thể sử dụng URL mô tả dịch vụ trong phản hồi để lấy thông tin chi tiết về dịch vụ từ tệp XML.

4. Interaction and controlling
   - Các thiết bị sử dụng các dịch vụ UPnP được phát hiện để điều khiển và tương tác với nhau.

## Các bước tiến hành cuộc tấn công DDoS - SSDP Amplification

![ddos-ssdp-amplification](/posts/2021-04-30-how-does-a-ssdp-attack-work/ssdp_amplification.png)
_DDoS SSDP Amplification_

Như đã trình bày cách thức của giao thức SSDP ở trên, vậy tấn công DDoS bằng SSDP là việc lợi dụng cơ chế hoạt động của SSDP để gửi các yêu cầu SSDP giả mạo đến các thiết bị hỗ trợ SSDP và sau đó chuyển hướng phản hồi từ các thiết bị này đến nạn nhân. Quá trình diễn ra như sau:

### 1. Xác định mục tiêu và tìm kiếm vũ khí
- **Xác định mục tiêu**: là một máy chủ, dịch vụ, hoặc mạng mà mình muốn làm tê liệt thông qua DDoS. Việc xác định mục tiêu điều quan trọng nhất là phải xác định được địa chỉ IP (hoặc range IP) của đối tượng cần múc ^^
- **Tìm kiếm vũ khí**: hay còn gọi là nguồn khuyếch đại, cần tìm các thiết bị hỗ trợ giao thức SSDP trên mạng internet để có thể sử dụng làm công cụ khuếch đại. Các thiết bị này có thể là router, camera an ninh, máy in, TV thông minh, hoặc các thiết bị IoT.

### 2. Scan để tìm thiết bị hỗ trợ SSDP trên Internet
Có thể sử dụng các công cụ để quét mạng và tìm các thiết bị mở cổng **SSDP** (UDP trên cổng **1900**) có thể khai thác. Một số công cụ phổ biến bao gồm:
- **Nmap**: Sử dụng Nmap để quét cổng UDP 1900 trên các dải IP lớn để phát hiện các thiết bị hỗ trợ SSDP.
  ```
  nmap -sU -p 1900 --open <IP-range>
  ```
- **Shodan**: có thể sử dụng công cụ tìm kiếm Shodan để tìm kiếm các thiết bị UPnP/SSDP mở ra internet, vì Shodan liệt kê các thiết bị IoT có cấu hình công khai.

### 3. Giả mạo địa chỉ IP (IP Spoofing)
Như đã nói ở step 1, cần phải biết địa chỉ IP của đối tượng. Sau đó, chỉ cần thay thế địa chỉ IP nguồn trong gói tin SSDP bằng địa chỉ IP của **mục tiêu**. Điều này có nghĩa là khi các thiết bị hỗ trợ SSDP phản hồi, chúng sẽ gửi thông tin phản hồi đến địa chỉ IP của mục tiêu (nạn nhân), gây ra tắc nghẽn băng thông hoặc quá tải tài nguyên của nạn nhân.

### 4. Gửi yêu cầu SSDP (M-SEARCH Request) giả mạo
Thực hiện gửi nhiều yêu cầu **M-SEARCH** thông qua giao thức **UDP** đến các thiết bị đã tìm thấy. Yêu cầu này được thiết kế để kích hoạt phản hồi từ các thiết bị SSDP. Tin nhắn M-SEARCH yêu cầu các thiết bị gửi lại thông tin mô tả dịch vụ UPnP mà chúng hỗ trợ.

M-SEARCH Request:
```
M-SEARCH * HTTP/1.1
HOST: 239.255.255.250:1900
MAN: "ssdp:discover"
MX: 3
ST: ssdp:all
```
Trong yêu cầu này:
- **MAN: "ssdp:discover"**: Yêu cầu thiết bị trả lời tìm kiếm dịch vụ.
- **ST: ssdp:all**: Tìm kiếm tất cả các dịch vụ.

Khi gửi yêu cầu này đến các thiết bị SSDP, địa chỉ IP nguồn được giả mạo sẽ là địa chỉ IP của mục tiêu tấn công.

### 5. Nhận phản hồi SSDP khuếch đại
Khi nhận được yêu cầu M-SEARCH, các thiết bị SSDP sẽ phản hồi lại bằng tin nhắn **HTTP 200 OK** chứa thông tin mô tả về dịch vụ. Phản hồi này thường lớn hơn yêu cầu ban đầu nhiều lần, từ 30 đến 40 lần, tạo ra hiệu ứng **khuếch đại lưu lượng**.

SSDP response:
```
HTTP/1.1 200 OK
CACHE-CONTROL: max-age=1800
EXT:
LOCATION: http://69.69.69.69:80/upnp/device.xml
ST: upnp:rootdevice
USN: uuid:device-UUID::upnp:rootdevice
```
- Phản hồi này chứa URL mô tả dịch vụ UPnP, thường có dung lượng lớn hơn nhiều so với yêu cầu M-SEARCH ban đầu.

### 6. Lưu lượng tấn công đổ vào mục tiêu
- Phản hồi từ các thiết bị SSDP sẽ được gửi đến địa chỉ IP của nạn nhân do đã giả mạo địa chỉ IP nguồn. Điều này tạo ra một lưu lượng lớn đổ vào mục tiêu, gây ra **quá tải băng thông** hoặc **tắc nghẽn dịch vụ** tại máy chủ, hệ thống, hoặc mạng của mục tiêu.

- Do phản hồi SSDP thường lớn hơn nhiều so với yêu cầu ban đầu, cuộc tấn công có thể khuếch đại lưu lượng một cách đáng kể. Chỉ cần gửi một lượng nhỏ yêu cầu, nạn nhân có thể bị ngập trong hàng gigabyte dữ liệu phản hồi.

### 7. Tấn công DDoS thành công
Khi lượng lớn SSDP Response hướng vào mục tiêu, kết quả có thể là:
- **Nghẽn băng thông**: Băng thông mạng của nạn nhân bị quá tải, làm chậm lại hoặc ngừng hoàn toàn kết nối với các dịch vụ hợp lệ.
- **Quá tải tài nguyên hệ thống**: Hệ thống của nạn nhân phải xử lý lượng lớn dữ liệu phản hồi không mong muốn, dẫn đến quá tải tài nguyên CPU hoặc bộ nhớ.
- **Ngừng hoạt động dịch vụ**: Cuộc tấn công thành công nếu hệ thống mục tiêu không thể phục vụ các yêu cầu hợp lệ từ người dùng, dẫn đến việc dịch vụ bị gián đoạn hoặc ngừng hoạt động.

## Tại sao **SSDP** lại dễ bị lợi dụng để thực hiện các cuộc tấn công **DDoS**

### 1. Không có cơ chế xác thực
Bất kỳ thiết bị nào cũng có thể gửi và nhận yêu cầu SSDP mà không cần kiểm tra danh tính. Điều này làm cho SSDP dễ bị khai thác để gửi các yêu cầu giả mạo (spoofed requests) từ các địa chỉ IP nạn nhân.

### 2. Traffic Amplification
- SSDP Amplification Attack là một hình thức phổ biến của tấn công DDoS, trong đó lợi dụng tính chất phản hồi của SSDP.
- Khi một thiết bị nhận được một yêu cầu SSDP, nó thường phản hồi lại với thông tin chi tiết về các dịch vụ của nó. Những phản hồi này thường lớn hơn nhiều so với yêu cầu ban đầu, dẫn đến hiện tượng khuếch đại. Tỉ lệ khuếch đại của SSDP có thể lên đến **30-40 lần** so với kích thước của gói tin ban đầu.
- Kẻ tấn công sẽ gửi yêu cầu SSDP giả mạo với địa chỉ IP nguồn là IP của nạn nhân. Khi các thiết bị phản hồi, chúng sẽ gửi lượng lớn dữ liệu đến nạn nhân, gây ra hiện tượng nghẽn mạng, khiến hệ thống của nạn nhân bị quá tải.

### 3. Sử dụng cổng UDP không kiểm soát
- Không chỉ SSDP, các giao thức hoạt động trên giao thức **UDP** đều rất dễ bị lợi dụng để thực hiện tấn công DDoS, đây là một giao thức không kết nối và không có cơ chế bắt tay ba bước như TCP. Do đó, UDP không kiểm tra xem yêu cầu có thực sự đến từ nguồn hợp lệ hay không. Kẻ tấn công có thể dễ dàng giả mạo địa chỉ IP nguồn và gửi yêu cầu đến các thiết bị sử dụng SSDP mà không bị phát hiện.

### 4. Số lượng thiết bị hỗ trợ SSDP lớn và dễ bị khai thác
- SSDP được sử dụng rộng rãi trong các thiết bị mạng gia đình, đặc biệt là các thiết bị **IoT**, TV thông minh, router, camera an ninh, và các thiết bị thông minh khác. Nhiều thiết bị không được bảo mật đúng cách hoặc bị bỏ qua các bản cập nhật phần mềm.
- Nhiều thiết bị hỗ trợ SSDP có thể bị phát hiện và khai thác qua internet, đặc biệt là khi chúng không được cấu hình đúng hoặc không được bảo mật.
- Kẻ tấn công có thể quét mạng internet để tìm các thiết bị sử dụng SSDP, sau đó lợi dụng chúng để thực hiện tấn công DDoS khuếch đại.

### 5. Dễ dàng tạo và tự động hóa cuộc tấn công
- Các cuộc tấn công DDoS khuếch đại SSDP có thể được tự động hóa bằng cách sử dụng các công cụ có sẵn. Hacker có thể dễ dàng phát triển hoặc sử dụng những công cụ đã có để phát hiện các thiết bị mở cổng SSDP trên internet và gửi yêu cầu giả mạo.
- Sau đó, chúng chỉ cần tập trung lượng lớn các thiết bị này để tấn công một mục tiêu duy nhất, gây ra sự quá tải cho băng thông và tài nguyên của mục tiêu.

### 6. Khả năng bị khai thác qua internet
Dù SSDP được thiết kế cho mạng nội bộ, một số thiết bị lại vô tình mở các cổng SSDP ra ngoài internet, cho phép hacker từ xa quét và khai thác chúng.

## Wrapping up

- Phòng ngừa cuộc tấn công SSDP:
1. **Turn-off SSDP and UPnP**: Nhiều thiết bị mạng, đặc biệt là thiết bị IoT, hỗ trợ SSDP theo mặc định. Nếu không cần sử dụng UPnP, nên tắt tính năng này.
2. **Configure firewall to prevent SSDP from the internet**: Firewall nên được cấu hình để chặn lưu lượng UDP trên cổng 1900 từ các nguồn bên ngoài (Internet), chỉ cho phép lưu lượng SSDP nội bộ.
3. **Bandwidth limitation**: Cấu hình giới hạn băng thông cho các dịch vụ SSDP hoặc UDP để giảm thiểu tác động của các cuộc tấn công DDoS Amplification.

- Ngoài ra, ở link tham khảo bên dưới, Cloudflare có giới thiêu một trang web để check xem đường WAN hay thiết bị của mình hiện tại có đang bị lợi dụng để tấn công SSDP hay không (free nhé).

- Nếu nói không ngoa, thì cuộc chiến để bảo vệ hệ thống từ tấn công DDoS sử dụng UDP là một cuộc chiến về hạ tầng, ông nào hạ tầng mạnh hơn thì khả năng cầm cự lâu hơn, ý mình nói ở đâu là về bandwidth, resources, nguồn tấn công,... Rất nhiều ông phông bạt, lên khoe viết rule chặn này nọ, nhưng chắc lèo tèo 2-3 con server, khi vào scale lớn lại là một câu chuyện khác.

- Ngoài ra, cũng còn vài cách protect khác mà trong thực chiến gặp phải, mình sẽ nói rõ hơn ở các bài sau.

## References

- [How does a SSDP Attack work?](https://www.cloudflare.com/learning/ddos/ssdp-ddos-attack/)

- [Bad UPnP/SSDP - Check for WAN UPnP listening](https://badupnp.benjojo.co.uk/)

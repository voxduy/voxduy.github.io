---
title: A quick note - How do SSL/TLS certificate works?
author: voxduy
date: 2024-10-25 17:45:00 +0700
categories: [Security]
tags: [Security]
image:
  path: /posts/2024-10-25-how-do-ssl-tls-works/ssl-tls.png
  width: 800
  height: 500
pin: false
---

![how-do-ssl-tls-works](/posts/2024-10-25-how-do-ssl-tls-works/how_do_ssl_tls_works.gif)
_How do SSL/TLS certificate works_

### **Bước 1: Trình duyệt yêu cầu kết nối an toàn**

- Khi người dùng nhập URL có giao thức HTTPS (ví dụ: <https://www.google.com>), trình duyệt sẽ gửi yêu cầu kết nối an toàn đến máy chủ web của trang web đó.
- Nhìn về phía technical (client/server) thì hành động là cụ thể là client sẽ mở một kết nối TCP với máy chủ (Server) trên cổng 443 (thông thường cho HTTPS).

### **Bước 2: Máy chủ gửi chứng chỉ SSL/TLS**

- Khi kết nối TCP đã được thiết lập, quá trình SSL/TLS handshake bắt đầu.
- Phía Client gửi thông điệp “Client Hello” đến Server, chứa các thông tin ban đầu về phiên mã hóa mà Client hỗ trợ (chẳng hạn phiên bản SSL/TLS, các bộ mật mã – cipher suites, v.v.). Server phản hồi bằng “Server Hello”, trong đó lựa chọn bộ mật mã (CipherSuite) phù hợp, gửi chứng chỉ số của Server (Server certificate) trong đó chứa khóa công khai (public key) của nó, và có thể kèm theo yêu cầu chứng chỉ Client (client certificate request) nếu cần xác thực hai chiều.

### **Bước 3: Trình duyệt xác minh chứng chỉ SSL**

- Trình duyệt kiểm tra xem chứng chỉ SSL của máy chủ có hợp lệ hay không bằng cách xác thực nó với chữ ký số của Tổ chức Chứng nhận (CA - Certificate Authority). Nếu chứng chỉ hợp lệ (được ký bởi tổ chức CA đáng tin cậy, còn hạn hay không, v.v..), trình duyệt sẽ tiếp tục quá trình thiết lập kết nối an toàn.

### **Bước 4: Trình duyệt tạo khóa bí mật chung (Shared Secret)**

- Trình duyệt tạo một khóa bí mật phiên (Shared Secret) để sử dụng cho việc mã hóa dữ liệu. Sau đó, nó mã hóa khóa này bằng khóa công khai của máy chủ (đã nhận ở bước 2).

### **Bước 5: Máy chủ giải mã khóa bí mật phiên**

- Máy chủ sử dụng khóa riêng tư (private key) của mình để giải mã khóa bí mật phiên do trình duyệt gửi đến.

### **Bước 6: Bắt đầu phiên giao tiếp an toàn**

- Sau khi cả trình duyệt và máy chủ có khóa bí mật chung, dữ liệu sẽ được mã hóa bằng khóa này trước khi gửi qua lại giữa hai bên. Điều này đảm bảo rằng dữ liệu không thể bị chặn hoặc sửa đổi bởi các bên thứ ba.

### **Bước 7: Truy cập an toàn vào trang web**

- Khi quá trình thiết lập SSL/TLS hoàn tất, trình duyệt hiển thị biểu tượng ổ khóa trên thanh địa chỉ và cho phép người dùng duyệt web an toàn.

---

SSL (Secure Sockets Layer) và TLS (Transport Layer Security) là hai giao thức bảo mật giúp mã hóa dữ liệu truyền tải giữa trình duyệt và máy chủ web. TLS là phiên bản kế thừa và an toàn hơn của SSL. Mặc dù hiện nay TLS là tiêu chuẩn bảo mật phổ biến hơn, nhưng nhiều người vẫn sử dụng thuật ngữ "SSL" khi thực sự đang nói về TLS.

Mối quan hệ giữa SSL và TLS

- SSL là nền tảng ban đầu: Được phát triển bởi Netscape vào những năm 1990, SSL giúp mã hóa dữ liệu nhưng có nhiều lỗ hổng bảo mật.
- TLS kế thừa và cải tiến SSL: TLS ra đời để thay thế SSL với các thuật toán mã hóa mạnh hơn và hiệu suất tốt hơn.
- TLS hiện là tiêu chuẩn: Các phiên bản SSL cũ (SSL 2.0, SSL 3.0) đã bị loại bỏ do có nhiều lỗ hổng. Hiện nay, các trang web sử dụng TLS 1.2 hoặc TLS 1.3 để đảm bảo an toàn.

Khi bạn thấy HTTPS, điều đó có nghĩa là trang web đang sử dụng TLS (không phải SSL nữa). Dù nhiều người vẫn nói "chứng chỉ SSL", thực chất các chứng chỉ hiện nay hỗ trợ TLS. Như hình ảnh trên, mô tả quá trình TLS handshake (quá trình bắt tay giữa trình duyệt và máy chủ), mặc dù vẫn gọi là "SSL certificate".

Credit to [Cyber Edition](https://www.linkedin.com/company/cyberedition/)

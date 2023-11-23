---
layout:     post
title:      "Computer Network (P2) - OSI Model"
description: "Trong bài viết này mình sẽ giới thiệu về OSI Model - Open System Interconnection Model, model này sẽ giúp chúng ta hiểu được cách mà data được truyền đi giữa các end points trong mạng INTERNET."
date:    2023-11-04
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - TCP/IP
    - Socket
    - Networking
URL: "/socket-programing-p3/"
categories: [ Tech ]
---

## 1. Introduction.
Trước khi đến đọc bài viết này, các bạn nên tham khảo bài viết trước link này, việc nắm rõ các physical components trong một INTERNET là tiền đề để hiểu bài viết này rõ ràng hơn, và dễ để tượng tượng hơn.

Trong bài viết này, chúng ta sẽ tập trung để trả lời câu hỏi: Làm thế nào mà 2 máy tính có thể communicate được với nhau qua một computer network?.

![](/img/network2/how.png)

Giả sử ta có 2 máy tính được kết nối với nhau bằng cách kết nối 2 NIC ở 2 máy tính với nhau bằng 1 LAN cable và 2 connectors - RJ45, computer network được mô tả như hình vẽ bên dưới:

![](/img/network2/osi-origination.png)

Như ta được biết qua bài viết trước về network components sẽ giúp convert từ binary data về tín electrical signal, và ngược lại. Lúc này công việc của software - cụ thể ở đây là OS Kernel của 2 máy tính phải đưa binary data này về một standard nhất định. Nếu 2 máy tính này cài đặt 2 hệ điều hành khác nhau là macOS và windows, ở phần computer network trong 2 OS Kernel của 2 hệ hành này phải cùng implement một model chung, OSI chính là một model như thế.

## 2. OSI Model.
![](/img/network2/OSI.png)
Trên đây là OSI model, để giải thích model này mình sẽ sử dụng ví dụ sau đây:

> mình sẽ gõ lên thanh địa chỉ của trình duyệt Chrome: google.com, rồi nhấn `enter` và kết quả trang tìm kiếm của google. Mình sẽ giải thích cách mà data được biến đổi qua các layer của OSI model, và routing trong mạng Internet từ lúc gõ phím `enter` cho đến khi nhận được response và reder lên trình duyệt.

Để đơn giản hoá và không phải giải thích dài, mình sẽ bỏ qua việc giải thích quá trình browser của mình gửi request lên DNS server để lấy địa chỉ IP, quá trình tạo TCP connection giữa client và server. Mình sẽ chỉ tập trung giải thích các steps sau khi khi TCP connection được thiết lập, client bắt đầu trao đổi message với server. Mục đích của mình là chỉ làm rõ vài trò của từng layer trong OSI model.

### 2.1. Application Layer.
```
GET / HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.9999.99 Safari/537.36
Accept: text/html
Connection: close
```
Ở application layer, chrome sẽ giúp ta tạo HTTP message tương tự như trên.

### 2.2. Presentation Layer.
Presenation layer sẽ làm những việc sau:
- **Encryption/Decryption**: Encrypt/Decryption message bằng symmetrical key - key được lấy sau khi thực hiện SSL Handshake (Thực ra quá trình thực hiện SSL Handshake sẽ diễn ra sau khi thiết lập TCP connection, nhưng ở đây ta coi như TCP connection đã được thiết lập từ trước đó).
- **Data Compression**: Nén message trên để tiết kiệm chi phí network và tăng performance khi gửi message qua network.
- **Translation**: Convert HTTP message nhận được từ Application Layer về dạng binary.

Lúc này message đã ở định dạng binary, sẽ đẩy xuống Session Layer để xử lý tiếp.


### 2.3. Session Layer.
Session Layer sẽ làm những việc như sau:
- Yêu thiết lập TCP connection.
- Authentication.
- Authorization.

Nếu như giải thích đẩy đủ, thì trước khi thực hiện những việc làm ở Presentation Layer, TCP sẽ phải được thiết lập trước. Sau khi connection established, Session Layer sẽ không tham gia vào quá trình exchange data sau đó.

### 2.4. Transport Layer.
Có 2 loại protocol phổ biến được sử dụng ở layer này là: TCP và UDP. Trong trường hợp ta đang đến là truy cập `google.com` bằng trình duyệt Chrome, protocol được dùng ở layer này là TCP. Ở máy tính hiện đại, mọi việc ở Transport Layer sẽ được xử lý bởi OS Kernel, tức là lập trình viên sẽ không phải viết code để can thiệp mọi thứ diễn ra ở layer này.

#### 2.4.1. TCP Protocol.
Transport Layer với TCP sẽ làm những việc như sau:

- **Segmentation**: Message nhận được từ layer trước đó sẽ được chia thành nhiều segments, mỗi một segment sẽ được add thêm vào bởi header. Với TCP protocol, kích thước của header sẽ từ 20-60 bytes (thông thường là 20 bytes, 40 bytes còn lại sẽ được sử dụng nếu cần thiết), hình vẽ dưới sẽ minh hoạ các thành phần của một TCP header: ![](/img/network2/tcp-header.png)


    Mình sẽ giải thích ý nghĩa của một số phần chính trong header được minh hoạ như trên:

    - **Checksum**: giúp phía nhận segments kiểm tra xem trong quá trình data được chuyển đi trong network, liệu có segment có bị sửa đổi gì hay không.

    - **Sequence Number**: thứ tự của segment, phía nhận segments sẽ dựa vào đây để sắp xếp lại segments theo đúng thứ tự để tạo nên message hoàn chỉnh.

    - **Acknowledgement Number**: khi phía nhận segments thành công, Acknowledgement Number sẽ được gửi lại phía gửi segments để thông báo segment đã được gửi thành công. Nếu trong trường hợp segments không được gửi đến đúng phía nhận, sẽ không có Acknowledgement Number được gửi về phía gửi segment, sau một khoảng thời gian không thấy Acknowledgement Number này, phía gửi segment sẽ tự động gửi lại segment.

    - **Source Port Address**: port được bind với client socket.

    - **Destination Port Address**: port đang được server sử dụng, trường hợp này là port 443 của google server.


- **Flow Control**: đảm bảo receiver sẽ không bị quá tải khi nhận data từ sender, và sender cũng sẽ gửi lượng data đúng với capability mà receiver của thể nhận được.

- **Error Control**: Sử dụng Checksum trong header segment để kiểm tra xem segment có bị modify gì không trong quá trình segment đi qua network.

Như vậy, sau kết thúc Transport layer, ta thu được tập hợp segments.
#### 2.4.1. UDP Protocol.
Transport Layer với UDP sẽ làm những việc như sau:

- **Segmentation**: tương tự như TCP, nhưng UDP header chỉ có kích thước 8 bytes, các thành phần trong UDP header được mô tả trong hình vẽ dưới đây: ![](/img/network2/udp-header.png)
- **Error Control**: checksum tương tự TCP.

Chú ý:
    - Khi sử dụng UDP protocol, không cần tạo connection trước khi thực hiện exchange data như TCP.
    - Nếu segment bị mất trong quá trình exchange, receiver sẽ không gửi acknowledgement để yêu cầu sender gửi lại segment. 
    - UDP protocol cũng sẽ không kiểm tra thứ tự của segment, vì vậy thứ tự của segments tại receiver sẽ có thể không đúng.
    => Sử dụng UDP sẽ có performance tốt hơn TCP nhưng về reliability sẽ không bằng TCP.

Một số service sử dụng UDP như: streaming video, song, DNS, ...

### 2.5. Network Layer.
Network layer nhân segments từ Transport layer, Network Layer sẽ làm những việc như sau:

- **Logical Addressing**: IPv4 và IPv6 gọi là logical addresses, mỗi một segment sẽ được add thêm header vào để tạo thành một packet. Header này gồm các thông tin được mô tả trong hình vẽ sau: ![](/img/network2/ip-header.png)

    - **Source Address**: lúc này vẫn là private IP của máy tính, khi đi đến router của mạng nội bộ chúng ta, IP này sẽ bị thay bởi public IP của router *(NAT giúp ta làm điều này). Từ đó cho đến khi chạm destination, source IP sẽ không bị thay đổi.

    - **Destination Address**: Địa chỉ IP của destination, ở đây chỉ public IP.

- **Routing**: Dựa vào `Destination Address` packets sẽ được route đến `hop` kế tiếp cho đến khi packets chạm đến destination.

- **Path Determination**: Sử dụng các thuật toán như: OSPF, BGP, IS-IS để tìm ra đường đi tốt nhất để route packet (phần này mình cũng mơ hồ, chưa tìm hiểu kỹ, khi tìm hiểu rồi sẽ update.)


### 2.6. Data Link Layer.

### 2.7. Physical Layer.
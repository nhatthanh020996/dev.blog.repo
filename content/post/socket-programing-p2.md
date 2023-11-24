---
layout:     post
title:      "Computer Network (P2) - OSI Model"
description: "Trong bài viết này mình sẽ giới thiệu về OSI Model - Open System Interconnection Model, model này sẽ giúp chúng ta hiểu được cách mà data được truyền đi giữa các end points trong mạng INTERNET."
date:    2023-11-07
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - TCP/IP
    - Socket
    - Networking
URL: "/socket-programing-p2/"
categories: [ Tech ]
---

## 1. Introduction.
Trước khi đến đọc bài viết này, các bạn nên tham khảo bài viết trước [Link](https://nhatthanh020996.github.io/socket-programing-p1/) này, việc nắm rõ các physical components trong một INTERNET là tiền đề để hiểu bài viết này rõ ràng, và dễ để tượng tượng hơn.

Trong bài viết này, chúng ta sẽ tập trung để trả lời câu hỏi: Làm thế nào mà 2 máy tính có thể communicate được với nhau qua một computer network (chú ý computer network có thể là local network hoặc Internet)?.

![](/img/network2/how.png)

## 2. OSI Model.

OSI Model partions data flow trong suốt quá trình communication thành 7 abstract layers, bao gồm
- Application Layer.
- Presentation Layer.
- Session Layer.
- Transport Layer.
- Network Layer.
- Data Link Layer.
- Physical Layer.

![](/img/network2/OSI.png)

Hình trên mô tả sự thay đổi data flow trong communication system (Hình ảnh hơi nhỏ, bạn có thể mở hình ảnh trong một tab riêng để xem rõ hơn). Có những network components implement đầy đủ cả 7 abstract layers, nhưng cũng có những components chỉ implement một số layer nhất định, ví dụ: client và server implemtent đầy đủ 7 layers, trong khi đó switchs chỉ implement 2 layers cuối, routers chỉ implement 3 layers cuối. Sau đây mình sẽ giải thích cách mà data được xử lý khi đi qua mỗi layer trong sơ đồ trên.

### 2.1. Application Layer.
Giả sử khi ta gõ vào thanh URL trong một tab của trình duyệt chrome: google.com, chrome sẽ giúp ta tạo HTTP message có bố cục như sau:
```
GET / HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.9999.99 Safari/537.36
Accept: text/html
Connection: close
```
Đây là content của một HTTP request message, bố cục của message trên thì được quy định tại link sau đây: https://datatracker.ietf.org/doc/html/rfc2616

Message được đẩy xuống Presentation Layer.

### 2.2. Presentation Layer.
Sau khi nhận message trên Application Layer, tại Presenation layer sẽ làm những việc sau:
- **Encryption/Decryption**: Encrypt/Decryption message bằng symmetrical key - key được lấy sau khi thực hiện SSL Handshake (Thực ra quá trình thực hiện SSL Handshake sẽ diễn ra sau khi thiết lập TCP connection, nhưng ở đây, để đơn giản, ta coi như TCP connection đã được thiết lập từ trước đó).
- **Data Compression**: Nén message sau khi encrypt sẽ được nén lại đến tiết kiệm chi phí và tăng performance khi được gửi đi trong computer network.
- **Translation**: Convert trên message từ string (ASCII) về dạng binary.

Lúc này message đã ở định dạng binary, sẽ đẩy xuống Transport Layer.


### 2.3. Session Layer.
Session Layer sẽ làm những việc như sau:
- Yêu thiết lập TCP connection.
- Authentication.
- Authorization.

Nếu như giải thích đẩy đủ, thì trước khi thực hiện những việc làm ở Presentation Layer, TCP connection sẽ phải được thiết lập. Sau khi TCP connection established, Session Layer sẽ không tham gia vào quá trình exchange data sau đó.

### 2.4. Transport Layer.
Có 2 loại protocol phổ biến được sử dụng ở layer này là: TCP và UDP.

#### 2.4.1. TCP Protocol.
Transport Layer với TCP sẽ thực hiện những việc sau:

- **Segmentation**: Message nhận được từ layer trước đó sẽ được chia thành nhiều segments, mỗi một segment sẽ được add thêm vào bởi header. Với TCP protocol, kích thước của header sẽ từ 20-60 bytes (thông thường là 20 bytes, 40 bytes còn lại sẽ được sử dụng nếu cần thiết), hình vẽ dưới sẽ minh hoạ các thành phần của một TCP header: ![](/img/network2/tcp-header.png)


    Một số phần chính trong header:
    - **Source Port Address**: port được bind với client socket.

    - **Destination Port Address**: port đang được server sử dụng.

    - **Checksum**: dựa vào checksum, receiver kiểm tra xem trong quá trình data được chuyển đi trong network, liệu có segment có bị sửa đổi gì hay không.

    - **Sequence Number**: receiver sẽ dựa vào Sequence Number để sắp xếp lại segments theo đúng thứ tự để tạo nên message hoàn chỉnh.

    - **Acknowledgement Number**: khi  nhận segments thành công, Acknowledgement Number sẽ được gửi lại sender, thông báo receiver đã nhận message thành công. Nếu trong trường hợp segments không được gửi đến receiver, sẽ không có Acknowledgement Number nào được gửi về sender, sau một khoảng thời gian không thấy Acknowledgement Number này, sender sẽ tự động retransmit segment.



- **Flow Control**: đảm bảo receiver sẽ không bị quá tải khi nhận data từ sender, và sender cũng sẽ gửi lượng data đúng với capability mà receiver của thể nhận được.

- **Error Control**: Sử dụng Checksum trong header segment để kiểm tra xem segment có bị modify gì không trong quá trình segment đi qua network.

Ở Transport Layer, ta thu được tập hợp segments.

#### 2.4.1. UDP Protocol.
Transport Layer với UDP Protocol sẽ thực hiện những việc sau:

- **Segmentation**: message cũng đưa chia nhỏ thành nhiều chunks, mỗi một chunk được add thêm UDP header, header có kích thước 8 bytes, các thành phần trong UDP header được mô tả trong hình vẽ dưới đây: ![](/img/network2/udp-header.png)
- **Error Control**: checksum tương tự TCP.

Chú ý:
    - Khi sử dụng UDP protocol, không cần tạo connection trước khi thực hiện exchange data như TCP protocol.
    - Nếu segment bị mất trong quá trình exchange, receiver sẽ không gửi acknowledgement cho sender gửi lại segment, nên dẫn đến data loss. 
    - UDP protocol sẽ không kiểm tra thứ tự của segment, vì vậy thứ tự của segments tại receiver sẽ có thể không đúng.
**Kết luận**: UDP có performance tốt hơn TCP nhưng về reliability sẽ không bằng TCP.

Một số services sử dụng UDP như: streaming video, song, DNS, ...

### 2.5. Network Layer.
Network layer nhận segments từ Transport layer, sẽ thực hiện những việc sau:

- **Logical Addressing**: IPv4 và IPv6 gọi là logical addresses, mỗi một segment sẽ được add thêm IP header vào để tạo thành một packet. IP header gồm các thông tin được mô tả trong hình vẽ sau: ![](/img/network2/ip-header.png)

    - **Source Address**: lúc này vẫn là private IP của máy tính, khi đi đến router của mạng nội bộ chúng ta, IP này sẽ bị thay bởi public IP của router *(NAT giúp ta làm điều này). Từ đó cho đến khi chạm destination, source IP sẽ không bị thay đổi.

    - **Destination Address**: Địa chỉ IP của destination, ở đây chỉ public IP của server.

- **Routing**: Dựa vào `Destination Address` packets sẽ được route đến `hop` kế tiếp, data sẽ được forward từ `hop` này sang `hop` khác cho đến khi đến destination.

- **Path Determination**: Sử dụng các thuật toán như: OSPF, BGP, IS-IS để tìm ra đường đi tốt nhất để route packet (phần này mình cũng mơ hồ, chưa tìm hiểu kỹ, khi tìm hiểu rồi sẽ update.)


### 2.6. Data Link Layer.
Data Link Layer sẽ nhận packets từ Network Layer, add thêm data link header và trailer vào mỗi packet để tạo thành một frame.

![](/img/network2/data-link.png)

Khi đi qua 1 `hop`, header và trailer được add từ hop trước sẽ được bóc ra và gắn vào header và trailer mới tương ứng với `hop` đó. Header và trailer sẽ bao gồm các thành phần mô tả trong hình sau:

![](/img/network2/data-link-header.jpeg)

### 2.7. Physical Layer.
Physical Layer nhận frames từ Data Link Layer và chuyển từ binary data sang những physical signal như electrical signal, light signal, radio signal (xem lại bài viết viết về network component tại đây: [Link](https://nhatthanh020996.github.io/socket-programing-p1/))

![](/img/network2/physical-layer.png)

Những tín hiệu vật lý sẽ được truyền đi từ `hop` này sang `hop` khác bởi network media như LAN cable, optical fiber hoặc air. Việc convert từ binary data sang physical signal sẽ được thực hiện bởi NICs, và chuyển sang dạng physical signal nào thì sẽ phụ thuộc vào network media network mà thiết bị đang sử dụng là gì.

## 3. Reference.
- https://www.youtube.com/watch?v=KZfvO1DVpjw&ab_channel=TechTerms
- https://www.youtube.com/watch?v=jRU_ReDUaMY&ab_channel=TechTerms
- https://www.youtube.com/watch?v=iYdW0B1olLE&t=156s&ab_channel=TechTerms
- https://www.geeksforgeeks.org/difference-between-segments-packets-and-frames/
- https://www.javatpoint.com/udp-protocol#:~:text=UDP%20Header%20Format,would%20be%2065%2C535%20minus%2020
- https://en.wikipedia.org/wiki/OSI_model
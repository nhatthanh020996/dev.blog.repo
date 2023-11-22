---
layout:     post
title:      "Computer Network (P1) - Network components"
description: "Internet là tập hợp nhiều thiết bị tính toán được kết nối lại với nhau cho phép chúng ta có thể gửi tin nhắn, voice calls, video calls, online shoping ... Trong bài viết này mình sẽ nói về những network components chính của Internet"
date:    2023-11-04
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - TCP/IP
    - Socket
    - Networking
URL: "/socket-programing-p1/"
categories: [ Tech ]
---

### 1. End Points
End points bao là những thành phần như: máy tính, máy tính bảng, điện thoại, server, máy in, ... là những thành phần đầu và cuối trong một kết nối internet.

### 2. Network Interface Card - NIC.
NIC bao gồm những thiết bị chúng ta thường gặp như wifi card, card mạng, ... Là thành phần giúp covert những loại tín hiệu như: electrical signal, light signal, radio signal về 1001 - tức data mà máy tính có thể hiểu. Ngược lại NIC cũng giúp convert từ binary về các loại tín hiệu trên. Mỗi một end point sẽ có một thiết bị NIC nhất định, ngoài ra những thành phần khác cũng sở hữu một NIC ví dụ switch, router. Hình vẽ sau đây biểu thị một NIC - card dây lan của một máy tính.

![](/img/network1/nic.png)

### 3. Network media.
Network media là component giúp truyền các tín hiệu electrical signal, light signal, radio signal từ NIC này sang NIC khác trong mạng máy tính.
![](/img/network1/network-media.png)
- LAN Cable sẽ được dùng để truyền Eletrical Signal.
- Optical Fiber sẽ được dùng để truyền Light Signal.
- Air sẽ được dùng để truyền Radio Signal.

### 4. Connector.
Connector là component để kết nối network media devices và NICs với nhau, vì vậy mà được coi như là một `connection point`. Thiết bị sau (phần màu vàng) là RJ45 là thiết bị kết nối LAN cable với một NIC của một chiếc laptop.

![](/img/network1/connector.png)

### 5. Switch.
Switch là thiết bị để kể nối tất cả các End points lại với nhau để tạo thành một local network. Ví dụ được mô tả trong hình dưới đây.

![](/img/network1/switch.png)

Một switch có nhiều ports và một port sẽ được kết nối với một end point khác nhau thông qua. Ngoài kết nối với end point, một switch có thể kết nối trực tiếp với một switch khác hoặc một router.

### 6. Router.
Sau khi có được một mạng nội bằng cách kết nối nhiều end points tới một switch như trên, để kết nối local network này với mạng internet ở bên ngoài, ta cần phải kết nối switch trên với một thiết bị là router.

![](/img/network1/router.png)

Router sẽ làm 2 nhiệm vụ:
- Kết nối các mạng nội bộ lại với nhau.
- Tìm đường đi tốt nhất để forward data tới destination. Vấn đề này ta sẽ tìm hiểu rõ hơn ở những phần sau.

Router này chính là đại diện của một local network với bên ngoài, tức là các thiết bị ở bên ngoài một mạng nội bộ thì chỉ biết địa chỉ IP của Router mà thôi, vì vậy địa chỉ IP của router chính là public address IP.

### 7. Network Components.
![](/img/network1/router.png)

Hình trên đã mô tả đẩy đủ cách mạng internet được hình thành từ các components trên. Hiểu được cơ bản mỗi component đảm nhận vai trò gì trong internet là tiền đề để ta có thể hiểu rõ hơn khi đi về chi tiết của mạng máy tính trong những bài viết sau. Cảm ơn các bạn đã theo dõi bài viết, hẹn gặp lại ở những bài viết sau.
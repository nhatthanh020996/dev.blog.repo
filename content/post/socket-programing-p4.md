---
layout:     post
title:      "Computer Network (P3) - Berkeley Socket API"
description: "Bài viết này mình sẽ cùng tìm hiểu về Berkeley Socket API hay còn gọi ngắn gọn hơn là socket - một trong những khái niệm quan trọng nhất trong computer network."
date:    2023-11-10
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - TCP/IP
    - Socket
    - Networking
URL: "/socket-programming-p4/"
categories: [ Tech ]
---
## 1. Introduction.
Socket là một trong những kiến thức low-level nhất và quan trọng nhất của computer networking. Nắm vững kiến thức về socket là tiền đề quan trọng để tiếp cận những kiến thức high-level hơn về computer networking như, TCP/IP protocol, HTTP, Websocket, SSH ... Bài viết này sẽ cung cấp cho bạn những kiến thức cơ bản sau:

- Lịch sử ra đời của ARPANET và TCP/IP.

- Tìm hiểu về Berkeley Socket API.

- Viết một chương trình đơn giản để mô tả flow hoạt động của Berkeley Socket API.

Bây giờ chúng ta cùng bắt đầu bài viết nhé.

## 2. Socket History.
Làm thế nào để 2 process đang chạy trên 2 máy tính khác nhau có thể trao đổi data được với nhau? đó là lý do thúc đẩy sự ra đời của ARPANET vào đầu thập nên 70 của thế ký trước. Cũng trong những năm đầu của thập niên 70 đó, TCP/IP protocol ra đời lúc đó vẫn đang được gọi với cái tên Transmission Control Program. Sau đó vào năm 1983, TCP/IP đã được APARNET sử dụng làm protocol chính cho toàn bộ mạng máy tính của quân đội Mỹ. ARPANET và TCP/IP chính là foudation của mạng INTERNET sau này.

Tại sao mình lại nhắc đến ARPANET và TCP/IP ở đây? vì mình muốn nói rằng bản thân TCP/IP không phải là một tập con của socket vì TCP/IP có trước cơ mà. Bản thân socket nó không phải là một protocol hay cái gì hết, nó đơn giản chỉ là một interface giúp người dùng máy tính có thể dễ dàng hơn để tạo ra một protocol - một việc mà trước khi có socket API, ta phải can thiệp trực tiếp vào OS Kernel, và chỉ có những nhà khoa học máy tinh mới làm được.

## 3. Berkeley Socket API.
### 3.1. History.
Trước khi Berkeley Socket API ra đời, để tạo được một network connection trên một OS nhất định, lập trình viên phải can thiệp vào OS kernel, mỗi một loại OS khác nhau lại phải can thiệp một cách khác nhau. Năm 1983, Hệ điều hành 4.2BSD Unix ra đời từ phòng nghiên cứu của đại học Berkeley, Hệ điều hành này cung cấp cho developers bộ APIs cho phép khởi tạo và sử dụng một network connection một cách dễ dàng hơn, lúc này developers không cần phải can thiệp trực tiếp vào OS Kernel để tạo ra một network connection nữa.

Sau sự ra đời của Berkeley Socket API, nhiều OS khác đương thời đã implement API này, biến nó trở thành một standard interface. Hiện nay, hầu hết các hệ điều hành hiện đại cũng đang implement Berkeley Socket API. Từ khi Berkeley Socket API được implement bời hầu hết các hệ điều hành, developers đã có thể dễ dàng tạo ra và sử dụng một network connection mà không cần quan tâm đến hệ điều hành mà họ đang sử dụng là gì (vì hầu hết OS đều implement API này).

Berkeley Socket API tiếp tục phát triển với một vài thay đổi nhỏ và cuối cùng trở thành một component trong POSIX specification - Portable Operating System Interface cũng là một interface khác, nhưng là một standard system interface, giúp các biến thể UNIX-like OS có thể cùng follow một standard nào đó, dẫn đến một sự thống nhất cần thiết. Ví dụ lệnh `ls` thì ở macOS, debian hay ubuntu đều mang một ý nghĩa như nhau.

### 3.2. Socket API functions.

The Berkeley socket API cung cấp những functions sau:

- `socket()`
	```C
	#include <sys/socket.h>

	int socket(int domain, int type, int protocol);
	```
	Function này tạo ra một socket - một endpoint communication, ví dụ: process A muốn trao đổi data với process B, thì cả A và B đều phải tạo ra 2 endpoint và trao đổi data giữa A và B sẽ được diễn ra thông qua 2 endpoint này. Function này sẽ trả về một file descriptor (kiểu integer) - là một indentifier của một file trong UNIX-like OS, ở đây socket là một special file type. Trong UNIX-like OS, mọi thứ đều là file, file ở đây không được hiểu là file thông thường được lưu trữ trong disk, mà một entity trong UNIX-like OS.

	`domain`: để chỉ communication domain, ví dụ: AF_UNIX - được dùng khi 2 processes chạy trên cùng một host. AF_INET - được dùng khi 2 processes chạy trên 2 host khác nhau, và indentifiers trong mạng máy tính của 2 host này được biểu thị bằng IPv4 Internet protocols.
	`type`: socket type - xác định protocol sẽ được sử dụng ở tầng transport. Ví dụ: SOCK_STREAM - khi muốn áp dụng TCP ở tầng transport, SOCK_DGRAM - khi muốn áp dụng UDP ở tầng transport.


- `bind()`
	```C
    #include <sys/socket.h>

    int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
	```
	Sau khi socket entity được create bới hàm socket(), bind() function sẽ tiến hành assign một địa chỉ cho socket đó. Ví dụ: giả sự socket được tạo ra với SOCK_STREAM type, ta có thể assign cho nó address là '127.0.0.1' và port: 8080. Trong đó với giá trị của port, được biểu thị bằng 16bits => tổng số socket tối ta có thể tạo ra trong một máy tính chạy linux kernel là 2^16 - 1 = 65535. Về mặt lý thuyết, đây cũng chính là số connection tối đa mà một máy tính chạy linux kernel có thể đạt được, tất nhiên nó còn phụ thuộc vào resource về phần cứng của máy tính và nhiều yếu tố khác, nên gần như chắc chắn sẽ không đạt được con số này trên bất kỳ máy tính nào.

- `listen()`
	```
	#include <sys/socket.h>

	int listen(int sockfd, int backlog);
	```
	Function này chỉ sử dụng với server socket, nó đánh dấu rằng socket với file descriptor là `sockfd` bắt đầu lắng nghe yêu cầu tạo connection từ client sockets.

	`sockfd`: file descriptor của server socket.
	`backlog`: số connection tối đa mà accept queue có thể chấp nhận.

- `connect()`
	```C
	#include <sys/socket.h>

	int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
	```
	Hàm này chỉ thực hiện khi socket type là SOCK_STREAM, với socket type là SOCK_DGRAM thì không dùng. Client socket dùng hàm này để yêu cầu tạo connection với server socket.

	`sockfd`: file descriptor của client socket.
	`sockaddr`: address và port của server socket.

	Sau khi server socket nhận lời yêu cầu tạo connection từ client socket, sẽ tiến thành three-handshake procedure, sau khi thành công three-handshake, yêu cầu sẽ được chuyển vào accept queue. Server socket sẽ lấy pop các phần tử trong accept queue để tạo connection.


- `accept()`
	```C
	#include <sys/socket.h>

    int accept(int sockfd, struct sockaddr *_Nullable restrict addr, socklen_t *_Nullable restrict addrlen);
	```
	Hàm này chỉ thực hiện khi socket type là SOCK_STREAM, với socket type là SOCK_DGRAM thì không dùng.

	accept() được thự hiện bởi socket server, sau khi accept incoming attempt, một socket mới sẽ được tạo ra, ta gọi socket này là: conn socket - đại diện cho connection giữa client và server socket. Từ đó về sau, conn socket sẽ giữa vai trò trao đổi data với client socket.

	`sockfd`: file descriptor của server socket.
	`sockaddr`: địa chỉ + port của client socket.



- `send()`, `recv()`, `sendto()`, and `recvfrom()`
	```C
	#include <sys/socket.h>

	ssize_t send(int sockfd, const void buf[.len], size_t len, int flags);

	ssize_t sendto(int sockfd, const void buf[.len], size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);


	ssize_t recv(int sockfd, void buf[.len], size_t len, int flags);

	ssize_t recvfrom(int sockfd, void buf[restrict .len], size_t len,
	                 int flags,
	                 struct sockaddr *_Nullable restrict src_addr,
	                 socklen_t *_Nullable restrict addrlen);
	```
	Sau khi connection được tạo ra, conn socket và client socket sẽ tiến hành trao đổi data với nhau. 
	`send()`: là hàm gửi data đi tới socket khác.
	`recv()`: là hàm nhận data từ socket khác.

- `close()`
	```C
	#include <unistd.h>

	int close(int fd);
	```
	Sau khi trao đổi data xong, cả client socket và conn socket đều có thể chủ động close connection. Sau khi gọi hàm close, socket sẽ được giải phóng khỏi bộ nhớ.

### 3.3. Sequence of function calls.
![](/img/socket/sockets-tcp-flow.webp)
Hình ảnh trên biểu thị thứ tự các functions được call từ khi một server socket và client socket được khởi tạo, cho đến khi client socket gửi request tạo connection cho server socket, rồi quá trình trao đổi data, cuối cùng là kết thúc bằng close connection.

## 4. Reference.
- https://csperkins.org/teaching/2007-2008/networked-systems/lecture04.pdf
- https://en.wikipedia.org/wiki/ARPANET#cite_note-59
- https://en.wikipedia.org/wiki/Berkeley_sockets
- https://en.wikipedia.org/wiki/Unix_domain_socket
- https://www.geeksforgeeks.org/packet-loss-detection-algorithms/
---
layout:     post
title:      "Computer Network (P4) - How To Create A TCP Client - Server in Python"
description: "Ở bài viết trước, mình nói về Berkeley Socket API - một interface giúp developer có thể dễ dàng để tạo ra một network connection mà không cần quan tâm đến OS đang sử dụng là gì, điều mà rất khó để làm được trước sự ra đời của socket. Trong bài viết này mình sẽ viết một Client - Server bằng Python để mô tả Berkeley Socket API"
date:    2023-11-14
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - TCP/IP
    - Socket
    - Networking
    - Python
URL: "/socket-programming-p4/"
categories: [ Tech ]
---

## 1. Introduction.
Trong bài viết này, mình sẽ sử dụng `socket` module - một standard module trong python để tạo socket. `socket` module thực ra là một wrapper của socket interface in C, vì vậy nó cũng sẽ có đẩy đủ những function như: `bind()`, `listen()`, `connect()`, `accept()`, `recv()`, `send()`, `close()`, `shutdown()`, ... `socket` module được sử dụng để tạo underlying webserver của hầu hết các webframework trong python như: Django, FastAPI, Flask

Sau khi kết thúc bài viết này chúng ta sẽ nắm được những kiến thức sau:
- Tạo được client - server đơn giản, client và server giao tiếp với nhau bằng TCP protocol.
- Kết hợp với bài viết trước về [OSI model](https://nhatthanh020996.github.io/socket-programing-p3/), ta có thể mapping xem phần code nào sẽ tương ứng với layer nào trong OSI model.

## 2. Echo Client and Server
### 2.1. Socket Server.
socket server:
```python
# echo-server.py
import socket

HOST = "127.0.0.1"
PORT = 65432 # Port to listen on (non-privileged ports are > 1023)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server_socket:
    server_socket.bind((HOST, PORT))
    server_socket.listen()
    conn, addr = server_socket.accept()
    with conn:
        print(f"Connected by {addr}")
        while True:
            data = conn.recv(1024)
            if not data:
                break
            conn.sendall(data)
```

### 2.2. Socket Client.
Code để tạo một socket client:
```python
# echo-client.py
import socket

SERVER_HOST = "127.0.0.1"
SERVER_PORT = 65432

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client_socket:
    client_socket.connect((SERVER_HOST, SERVER_PORT))
    client_socket.sendall(b"Hello")
    data = client_socket.recv(1024)

print(f"Received {data!r}")
```


## 3. Handling Multiple Connections.

### 3.1. Simple.
Đây là cách dễ nhất để tạo ra một server, mình sẽ tạo ra một infinite loop để look up `Accept Queue`, Nếu `Accept Queue` chứa attempt connection từ clients, attempt connections này sẽ được pop ra để handle. Với cách làm này, những attempt connections này sẽ được xử lý tuần tự, tức sẽ xẩy ra hiện tượng blocking, đây là điểm yếu của các làm này.

**Chú ý**: `Accept Queue` là một queue, phần tử trong queue này là danh sách những request tạo connection từ clients, những request này đã pass three-way handshake và sẵn sàng tạo connection.
```python
# echo-server.py
import socket

HOST = "127.0.0.1"
PORT = 65432 # Port to listen on (non-privileged ports are > 1023)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server_socket:
    server_socket.bind((HOST, PORT))
    server_socket.listen()
    while True:
        conn, addr = server_socket.accept()
        with conn:
            print(f"Connected by {addr}")
            while True:
                data = conn.recv(1024)
                if not data:
                    break
                conn.sendall(data)
```

### 3.2. Using multithreads.
Ở cách làm này, mỗi một connection sẽ được handle trong một thread riêng dành cho nó, vì vậy ta có thể handle nhiều connections trong một thời điểm. Điểm yếu của cách làm này đó là danh sách attempt connections vẫn sẽ được accept lần lượt, nên vẫn có thể tạo blocking. Mặc dù vậy cách làm này đã tốt hơn cách trước rất nhiều.
```python
# echo-server.py
import socket
import threading

HOST = "127.0.0.1"
PORT = 65432

def process_request(self, conn, client_address):
	with conn:
		while True:
			data = conn.recv(1024)
			if not data:
				break
			conn.sendall(data)

def serve_forever():
	with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server_socket:
		server_socket.bind((HOST, PORT))
		server_socket.listen()
		while True:
			conn, addr = server_socket.accept()
			t = threading.Thread(target=process_request, args=(conn, addr, ))
			t.start()

if __name__ == '__main__':
	serve_forever()
```

### 3.3. Using select().

## 4. Reference.
- https://realpython.com/python-sockets/
- https://docs.python.org/3/howto/sockets.html#

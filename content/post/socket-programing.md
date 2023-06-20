---
layout:     post
title:      "Socket Programming"
description: "Làm thế nào để 2 processes trong một máy tính hoặc nằm trên hai máy tính khác nhau 
có thể giao tiếp được với nhau? Trong bài viết này mình xin nói về socket programming."
date:    2023-06-11
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - TCP/IP
    - Socket
URL: "/socket-programming/"
categories: [ Tech ]
---
## Mở đầu.
A socket is a fundamental component provided by operating system that enables communication between processes, whether they reside on the same
machine or different machines. To facilitate effective communication, processes must adhere to specific protocols, similar to humnans utilize a 
common language to communicate, TCP/IP stands out as the prevailing choice in modern times. This article will primarily focus on TCP/IP due to
its widespread adoption. To establish a clear understanding, let's define a couple of key terms:
1. Socket: It represents the means by which two processes can exchange signals each other.
2. TCP/IP: This protocol builds upon the concept of socket. Signals transmitted by processes must adhere to predefined set of rules to ensure
mutual comprehension between two communicating processes.

## 
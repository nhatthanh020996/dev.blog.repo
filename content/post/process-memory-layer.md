---
layout:     post
title:      "Operating System (P2) - Memory Layers Of A Process"
description: "Ở trong bài viết trước về Process, mình đã có nói đến một memory representation của một process sẽ gồm những segment nào? Trong bài viết này mình sẽ nói kỹ hơn về các phần của một memory representation sẽ thay đổi ra sao trong quá trình thực thi process."
date:    2023-10-20
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - Operating system
URL: "/process-memory-layer/"
categories: [ Tech ]
---

## 2. Memory layers.
Memory layers của một process sẽ 4 segment chính như hình vẽ dưới đây.
![](/img/memory-layers/layers.jpeg)

Hình ảnh trên biểu thị memory layout của một chương trình C, ở đây data segment được chia ra bao gồm uninitialized data và initialized data, nhưng để cho đơn giản ta sẽ gọi 2 phần này là data segment.

Sau khi process được OS cung cấp memory, sẽ có data segment và text segment là những vùng nhớ cố định, không thay đổi về cả kích thước lẫn nội dung của 2 vùng nhớ này trong suốt quá trình process được thực thi.

Heap segment và stack segment là những vùng nhớ thể thay đổi trong suốt quá trình thực thi process, mình nói là "có thể" vì sẽ có nhưng chương trình đơn giản không sử dụng đến hai vùng nhớ này.

## 3. Text segment and data segment.
```assembly
.data
    msg: .asciiz "hello world \n"

.text
    li $v0, 4
    la $a0, msg
    syscall
```
## 4. Stack segment.

## 5. Conclusion.



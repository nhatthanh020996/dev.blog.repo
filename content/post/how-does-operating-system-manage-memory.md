---
layout:     post
title:      "Operating System (P3) - Memory Management"
description: "Khi tồn tại nhiều processes trong hệ thống, làm thế nào để Operating System có thể cấp phát bộ nhớ cho những processes này một cách hợp lý nhất. Memory Management là một trong những nhiệm vụ quan trọng nhất của OS, và trong bài viết này mình sẽ nói về điều này."
date:    2023-10-27
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - Operating system
URL: "/how-does-operating-system-manage-memory/"
categories: [ Tech ]
---

## 1. Logical vs Physical Address Space.
### 1.1. Logical address.
Logical address sẽ được sinh ra ở lúc biên dịch chương trình, 
Ví dụ ta có một chương trình đơn giản helloworld như sau:

```assembly
.data
    msg: .asciiz "Hello world! \n"

.text
    li $v0, 4
    la $a0, msg
    syscall
```
Sau khi biên dịch, ta sẽ thấy được logical address của mỗi instruction như sau:

![](/img/memory-management/logical-address.png)

Các giá trị ở cột `Address` chính là logical address của mỗi instruction tương ứng.


### 1.2. Physical address.

Là địa chỉ thật sự mà instruction, data, stack,... trong memory, sau khi process được OS cấp phát memory. Với những hệ điều hành hiện đại ngày này, bộ nhớ của một process trong memory sẽ không phải là một giải liên tục. Cách phân bổ chúng như thế nào sẽ được nói rõ hơn ở phần sau.


## 2. Paging.
### 2.1. Paging concept.

Chia physical memory ra thành các block có kích thước bằng nhau và là luỹ thừa của 2 (tại sao phải là luỹ thừa của 2 thì sẽ được giải thích ở phần sau). Block này gọi là `frames`.

Chia logical memory ra thành các block có kích thước bằng với một frame, mỗi một block như vậy gọi là một `page`

### 2.2. Allocate memory to process.
Để load một chương trình vào bộ nhớ trong thì phải theo các bước sau:
- Phải keep tracking tất cả những free frames in RAM.
- Để run một chương trình có kích thước n pages, cần phải tìm n free frames và load chương trình vào trong bộ nhớ.

- Setup một page table để translate logical addresses to physical addresses.
- Khi một frame được giải phóng sau khi được thực hiện trong tiến trình, frame này sẽ được cập nhật vào danh sách những frames đang nhàn rỗi trong bộ nhớ, mà có thể được sử dụng bởi những tiến trình khác.
- Mỗi một process có một page table riêng.
- Mỗi một page table của một process sẽ được lưu trong process control block của nó.

**Note**: Ở phương pháp này vẫn tồn tại phân mảnh, thường là ở frame cuối cùng, khi page cuối cùng có kích thước nhỏ hơn frame.

## 3. Address Translation scheme.
### 3.1. Page number, page offset
Logical address sẽ được xác định thông qua 2 thành phần:
    - Page number - p: số hiệu trang.
    - Page offset - d: địa chỉ offset ở trong trang đó.

Khi một logical address được nạp vào 1 thanh ghi PC, ta có thể tìm ra được physical address của nó bằng cách trực quan như sau:
![](/img/memory-management/page_offset.png)

Trong thực tế, ta không dùng cách trên để tìm ra physical address vì ta có thể thấy là nó rất tốn kém chi phí để tính toán. Thay vào đó người ta dùng phương pháp sau đây:
- Giả sử ta có một thanh ghi 6bits để chứa địa chỉ, như vậy ô nhớ địa chỉ logic là 9 trong trường hợp trên sẽ được biểu diễn dưới dạng nhị phân là 001010
- 2 bits cuối sẽ được dùng để chứa để chứa offset, còn những bits còn lại sẽ được sử dụng để chứa page index. Như vậy ta có 001010 ⇒ 0010|10, ta suy ra được ngay với logical address là 9 thì sẽ nằm ở page thứ 0010=2 và offset 10=1. Đó là lý do vì sao frame_size phải là một luỹ thừa của 2, vì chỉ có vậy ta mới sử dụng được phép dịch bit và tìm ra physical address một cách nhanh nhất.

### 3.2. Cache.
Như vậy nếu CPU đọc được virtual address từ Memory Address Register, để biết được physical memory address và load instruction vào thanh ghi CPU thì CPU phải access và RAM 2 lần. Giả sử virtual address là pd:
- Đầu tiên CPU phải truy cập vào page table của process để tìm kiếm frame tương ứng với p. Tính toán để tìm ra physical address.
- Sau khi tìm được physical address, CPU sẽ phải access vào địa chỉ physical address này ở RAM lần thứ 2 để đọc instruction.
Truy cập vào RAM 2 lần như vậy sẽ làm chậm quá trình xử lý process của CPU, vì vậy người ta nghĩ ra việc lưu page table vào cache.

# 4. How does page table look like?
Trong quá trình tải 1 process vào bộ nhớ trong, có thể vì một lý do nào đấy, giả sử số pages để chứa process đó lớn hơn tổng số free frames còn lại trong bộ nhớ trong chẳng hạn. Thì hệ điều hành sẽ chỉ cấp một số lượng nhất định frames cho cho một số pages nhất định của process, những pages này sẽ được đánh dấu là `valid` trong page table, những page chưa có sẵn trong memory sẽ đánh dấu là `invalid`. Ta có hình ảnh minh hoạ như sau:

![](/img/memory-management/valid-invalid-tag.png)

Lỗi Page Fault sẽ xuất hiện lúc truy cập vào page được đánh dấu invalid.

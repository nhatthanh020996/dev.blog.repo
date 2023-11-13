---
layout:     post
title:      "Operating System (P1) - Process."
description: "Mình sẽ mở đầu series về Operating System với Process Management - một trong những phần quan trọng nhất trong nguyên lý hệ điều hành."
date:    2023-10-20
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - Operating system
URL: "/what-is-process/"
categories: [ Tech ]
---

## 1. Process concept.
### 1.1. The Process.
Một chương trình đang được thực thi bởi máy tính gọi là Process, ví dụ:
Bạn có một chương trình helloworld được viết bằng C trong file helloworld.c như sau:
```C
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```
Bạn tiến hành biên dịch chương trình trên bằng câu lệnh sau trên terminal:
```bash
gcc helloworld.c -o helloworld
```
Bạn tiếp tục chạy câu lệnh sau trên terminal:
```bash
./helloworld
```
Lúc này một process đã được khởi tạo.

Mình muốn nhấn mạnh rằng một program bản thân nó không phải là một process. Một program là một `passive entity`, ví dụ: là một file chứa nhiều câu lệnh và được lưu ở trong disk, ta gọi nó là executable file, trong ví dụ trên là `helloworld`. Ngược lại, process là một `active entity`, và một program trở thành một process khi `executable file` được load vào trong memory. Memory representation của một process sẽ bao gồm những thành phần sau:
1. **Text segment**: chứa instructions của của process, khi process được thực thi bởi CPU, những instruction này sẽ được fetched vào CPU để thực thi.
2. **data**: chứa những hằng số được sử dụng ở trong chương trình của bạn. Ví dụ: ở chương trình helloworld ở trên, thì data segment sẽ phải có một vùng nhớ để chứa "Hello, World!\n", khi CPU tiến hành fetch instructions ở text segment để thực thi, CPU sẽ tiến hành load "Hello, World!\n" từ vùng vùng nhớ - nơi chứa string này ở memory vào thanh ghi CPU. CPU sau đó sẽ thực hiện instruction in đoạn text này ra I/O device - trong trường hợp ở ví dụ trên là monitor.
3. **Heap**: là vùng nhớ dùng để chứa những biến được cấp phát động trong quá trình chương trình được thực thi.
4. **Stack**: Trong quá trình thực thi chương trình, khi xuât hiện lời gọi hàm, sẽ tạo ra một stack frame ở trong vùng nhớ stack, vùng nhớ này sẽ có thể ghi lại tham số truyền vào của hàm, địa chỉ trả về - là địa chỉ của instruction tiếp theo sau khi lời gọi hàm kết thúc, biến local được sử dụng trong hàm.

Trong khuôn khổ bài viết này mình sẽ không viết quá chi tiết về một memory representation của một process? tất cả điều này mình sẽ viết ra trong một bài viết khác.

### 1.2. Process State.
Sau khi một process được tạo ra, nó sẽ phải trải qua những trạng thái (state) như sau:
- **New**: Là trạng thái lúc process mới được khởi tạo, trước khi được hệ điều hành allocate tài nguyên memory cho nó.
- **Ready**: Lúc này process đã được OS cung cấp đủ tài nguyên memory, sẵn sàng để được OS allocate CPU và execute. Lúc này process sẽ được xếp vào `ready queue`, gồm nhiều process khác, tất cả đều chờ để được CPU thực thi.
- **Running**: Ở trạng thái này process được CPU fetches instructions trong text segment của nó ở memory, thực thi các instructions này.
- **Waiting**: Trong quá trình thực thi, sẽ có những lúc process chờ những event khác để được thực thi tiếp, ví dụ: chờ input được nhập vào từ bàn phím, chờ data được lấy từ internet device, ... Trong lúc chờ event, process sẽ move khỏi `ready queue` và add vào `waiting queue`, CPU lúc này sẽ tiến hành thực thi những processes khác ở ready queue. Sau khi có kết quả từ event, processes sẽ bị remove ra khỏi `waiting queue` và được add trở lại vào `ready queue` để tiếp tục chờ CPU xử lý.
- **Terminated**: Process kết thúc chu trình fetches instruction -> execute instruction.

![](/img/what-is-process/states.png)

### 1.3 Process Control Block.
Ở trong phần 1.2 ở trên, mình có nhắc đến những queue như `ready queue` hay `waiting queue`, mỗi phần tử trong những queue này chính là representation của một process, vậy representation này là gì?

`ready queue` sẽ chứa con trỏ chứa địa chỉ của những `process control block` gọi tắt là `PCB` (có một số hệ điều hành sẽ gọi là `task control block`). Một PCB là representation của một process, chúng sẽ chứa những thông tin chính sau:

- **Process state**: là trạng thái hiện tại của process, một trong những trạng thái đã được nhắc đến ở phần 1.2.

- **Program counter**: Là địa chỉ của instruction tiếp theo sẽ được thực hiện bởi CPU. Các bạn có thể xem lại bài viết về kiến trúc máy tính để biết thêm về thanh ghi PC.

- **CPU registers**: Khi có một interupt xuất hiện (ví dụ: process phải chờ một event nào đó từ I/O device để tiếp tục thực hiện), trạng thái của các registers sẽ được lưu lại ở trong PCB. Khi CPU thực hiện process trở lại, nó sẽ tiến hành load những giá trị này từ PCB vào CPU để tiếp tục thực thi proceess.

- **CPU-scheduling information**: mức độ ưu tiên của process, sẽ được nói rõ hơn ở phần sau.

- **Memory-management information**: Chứa các thông tin chính như sau:
    - `base register` và `limit register`: thanh ghi cơ sở và giới hạn của memory representation of process, 2 thanh ghi này giúp ngăn các processes access vào vùng nhớ lẫn nhau.
    - `page tables`: Memory của một process sẽ được OS chia thành các page (mỗi page gồm nhiều bytes, và có kích một page sẽ là luỹ thừa của 2), thanh ghi PC chỉ chứa `logical address` của process, nên CPU sẽ phải tìm cách đổi địa chỉ này qua `physical address` để có thể access vào memory và fetch instruction. Quá trình đổi địa chỉ này sẽ cần sử dụng `page tables`, chi tiết hơn về quá trình này sẽ được mình giải thích trong những phần sau về memory management của OS.

![](/img/what-is-process/PCB.png)

### 1.4. Threads.
Khái niệm process được mình sử dụng cho đến lúc này của bài viết là những process với một thread duy nhất trong đó, hay còn gọi là single thread. Ở máy tính hiện đại ngày nay, hầu hết các process có nhiều threads trong nó, điều này giúp tận dụng sức mạnh của những CPU có nhiều cores. Với những processes chứa multithreads, memory representation sẽ có sự thay đổi. Các threads trong một process sẽ dùng chung các vùng nhớ: text segment, data segment, và header segment, nhưng mỗi một thread sẽ có một stack riêng. 

Với những hệ điều hành hỗ trợ multithreads, thì đơn vị nhỏ nhất để OS schedule sẽ là thread, chứ không phải là process. Tuy nhiên, về cách OS schedule thực hiện, sẽ không có sự khác biệt nhiều giữa process và thread.

Thread là một phần rộng, nên phần kiến thức chi tiết hơn sẽ được mình đề cập trong một bài viết khác.

## 2. Process scheduling.
Mục đích của các hệ thống multiprogramming là cho phép nhiều process có thể được thực hiện cùng lúc, "cùng lúc" ở đây có thể là concurrency hoặc parallelism. Với single-processor system, mỗi một thời điểm sẽ chỉ có một process được CPU thực thi, khi process này rơi vào trạng thái `waiting`, sẽ có một process khác được CPU thực hiện, vì vậy sẽ có nhiều processes được CPU luân phiên thực thi, hình thức thực thi như vậy gọi là concurrency.

Với multi-processor system, tại một thời điểm có thể có nhiều processes được nhiều processors thực thi cùng một lúc, hình thức thực thi như vậy gọi là parallelism. Trong phần này, mình sẽ tập trung nói về process scheduling, nên để đơn giản hoá mình sẽ nói về process scheduling trong single-processor system.

### 3.2. Scheduling Queues.
Khi những processes được tạo ra trong hệ thống và được OS allocate memory, những processes này sẽ được chuyển sang trạng thái `ready` và được add vào `ready queue`. Queue này nhìn chung là một linked list, header chứa pointer của PCB đầu tiên, tail sẽ pointer của PCB cuối cùng. 

Hệ thống cũng tồn tại nhiều queues khác. Khi một process được allocated CPU, nó sẽ có thể được thực thi cho đến khi kết thúc instruction cuối cùng, hoặc phải nhường CPU cho process khác và bị đưa ra `waiting queue`, hoặc đẩy ra `waiting queue` vì phải chờ một event nhất định ví dụ: chờ để hoàn thành việc tạo một file ở disk, lúc này có nhiều process khác cũng interact with disk, vì vậy những processes này cũng phải được xếp vào một queue khác để được chờ được disk xử lý, queue này gọi là `device queue`, mỗi một thiết bị sẽ có `device queue` riêng. Tiến trình này sẽ được mô tả rõ hơn ở sơ đồ dưới đây.

![](/img/what-is-process/queue-scheduling.png)

### 2.3. Schedulers.
Một process sẽ được migrate qua nhiều queues khác nhau trong suốt vòng đời của nó. OS sẽ phải lập lịch cho những processes trong những queues này. Việc lập lịch này sẽ được thực hiện bởi scheduler.

Scheduler sẽ được chia thành 3 loại chính bao gồm: 
**long-term scheduler**: Hầu như chỉ xuất hiện trong các hệ thống batch system, và không được sử dụng ở các hệ thống time-sharing như UNIX hay Microsoft Windows.
**medium-term scheduler**: ý tưởng của kiểu scheduler này là những process nếu như cần thiết có thể bị remove ra khỏi memory và được lưu trữ tạm ở bộ nhớ thứ cấp (có thể là disk, cpu, cd ...), sau đó có thể nạp lại memory để chờ thực hiện bởi CPU, cơ chế này gọi là swapping. Kiểu scheduler này xuất hiện phổ biến ở các OS hiện đại.
**short-term scheduler**: Kiểu scheduler này phải chọn process cho CPU thực thi một cách rất thường xuyên, một process có thể chỉ được thực thi vài milliseconds trước khi phải chờ vì một I/O request. Thông thường, một `short-term scheduler` phải thực hiện công việc chọn process để CPU thực thi sau mỗi 100 milliseconds. Do quá trình chọn process này diễn ra liên tục, cho nên nó cũng gây ra nhiều wasted time. Điều đó dẫn đến nếu có quá nhiều process trong hệ thống, bạn có thể cảm nhận được máy tính của bạn không còn xử lý mượt mà nữa, đó là vì CPU phải switch qua lại giữa các processes quá nhiều.


## 3. Conclusion.
Trong khuôn khổ bài viết này, mình đã đi qua một số những kiến thức cơ bản về process. Do kiến thức hạn hẹp của bản thân, nên trong bài viết sẽ không thể tránh khỏi những sai sót. Mình hi vọng sẽ được ai đó đóng góp ý kiến, và mình luôn sẵn sàng đón nhận.

Cảm ơn tất cả mọi người đã kiên nhẫn đọc hết bài viết này của mình. Chúc mọi người có một buổi tối cuối tuần vui vẻ, hẹn gặp lại mọi người trong những bài viết sau.
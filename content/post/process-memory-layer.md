---
layout:     post
title:      "Operating System (P2) - Memory Layers Of A Process"
description: "Ở trong bài viết trước về Process, mình đã có nói đến một memory representation của một process sẽ gồm những segment nào? Trong bài viết này mình sẽ nói kỹ hơn về các phần của một memory representation sẽ thay đổi ra sao trong quá trình thực thi process."
date:    2023-10-23
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

Trên đây là một chương trình helloworld đơn giản được viết bằng assembly với kiến trúc tập lệnh MIPS.

Sau khi mình chạy chương trình trên bằng MARS simulator - MIPS Assembler and Runtime Simulator, chương trình trên sẽ được đưa vào memory simulator với 2 vùng nhớ như sau:

**Text Segment**
![](/img/memory-layers/text-segment.png)
text segment là space memory chứa instructions của process.

**Data Segment**
![](/img/memory-layers/data-segment.png)
data segment là space memory chứa static data của process, trong chương trình trên chính là dòng text "hello world \n"

Khi process được allocated CPU để thực thi instruction, thanh ghi PC sẽ chứa địa chỉ của instruction đầu tiên của process, trong chương trình này là: `0x00400000`. CPU sẽ fetches instruction được lưu ở địa chỉ trên thanh ghi PC, instruction sẽ được lưu ở thanh ghi IR (instruction register), giá trị trong thanh ghi PC sẽ được cộng lên đúng bằng độ dài của instruction, trong trường hợp này là 4 bytes. Sau đó, Control Unit sẽ điều phối CPU để thực hiện instruction, cách CPU thực thi một instruction như thế nào mình sẽ giải thích trong bài viết về kiến trúc máy tính. Chu trình fetch-execute instruction sẽ diễn ra cho đến instruction cuối cùng của process.

## 4. Stack segment.

```assembly
.data

msg: .asciiz "Enter a number"
answer: .asciiz "\nFactorial is: "

.text

# welcome message
li $v0, 4
la $a0, msg
syscall

# read integer
li $v0, 5
syscall

# print the integer
move $a0, $v0
li $v0, 1
syscall

jal calculate_factorial
move $a1, $v0

li $v0, 4
la $a0, answer
syscall

move $a0, $a1
li $v0, 1
syscall

li $v0, 10
syscall

calculate_factorial:
    addi $sp, $sp, -4
    sw $ra, ($sp)

    li $v0, 1
    multiply:
        beq $a0, $zero, return
        mul $v0, $v0, $a0
        addi $a0, $a0, -1
        j multiply

    return:
    lw $ra, ($sp)
    addi $sp, $sp, 4  # Deallocate the stack space
    jr $ra

```
Đây là chương trình tính giai thừa của một số bất kỳ được nhập vào từ bàn phím, trong chương trình này ta thực hiện lệnh gọi hàm `calculate_factorial` bằng câu lệnh `jal calculate_factorial`. Ở hàm `calculate_factorial`, ta thấy có 2 câu lệnh:

```
    addi $sp, $sp, -4
    sw $ra, ($sp)
```
Ở câu lệnh đầu tiên, stack pointer sẽ giảm xuống 4bytes, space memory từ địa chỉ ban đầu $sp đến 4 bytes trở xuống sẽ dùng để lưu các giá trị như: biến local, params truyền vào, địa chỉ trả về sau khi kết thúc lời gọi hàm, trong chương trình này địa chỉ trả về sẽ được lưu lại bằng instruction: `sw $ra, ($sp)`. Space memory như thê này sẽ được gọi là `stack frame`. Mỗi một lần gọi hàm như vậy, nếu cần một `stack frame`, thì giá trị của thanh ghi $sp lại được giảm xuống đúng bằng độ dài của `stack frame`.

Để kết thúc lời gọi hàm, `stack frame` sẽ được giải phóng với instruction:
```
addi $sp, $sp, 4 
```
chương trình sẽ tiếp tục thực hiện instruction mà địa chỉ được lưu trong thanh ghi $ra bằng lệnh:
```
jr $ra
```
$ra lúc này là địa chỉ của instruction `move $a1, $v0`, sau khi thực thi `jr $ra`, giá trị của thanh ghi $pc sẽ được update lại bằng giá trị của $ra, lại tiếp tục thực hiện fetch rồi execute instruction.

Chú ý rằng giới hạn vùng nhớ dành cho stack segment của một process sẽ tuỳ thuộc vào từng hệ điều hành, ví dụ với MACOS mà mình đang sử dụng thì giá trị này là 8MB. Từ đây ta có thể suy ra được nguyên nhân gây ra lỗi stack overflow, đó là do stack frame được tạo quá nhiều dẫn đến vượt quá không gian vùng nhớ dành cho stack segment được cho phép.

## 5. Conclusion.
Trong bài viết này mình đã chỉ ra một memory representation của một process sẽ trông như thế nào? các memory layers trong đó sẽ trông ra sao trong quá trình process được thực thi? Từ những kiến thức này có thể sẽ giúp ích cho chúng ta viết được một chương trình giúp tiết kiệm memory, hay có thể hiểu để giải thích một số vấn đề, bugs mà ta gặp phải trong quá trình làm việc.

Bài viết chắc chắn sẽ còn nhiều thiếu sót, hi vọng sẽ được đóng góp ý kiến với đọc giả. Cảm ơn các bạn vì đã đọc bài viết lần này, hẹn các bạn trong những bài viết sau.

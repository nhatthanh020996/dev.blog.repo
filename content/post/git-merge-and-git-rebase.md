---
layout:     post
title:      "Git rebase và git merge"
description: ""
date:    2023-06-12
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - Git
URL: "/git-merge-and-git-rebase/"
categories: [ Tech ]
---
## Git merge thực sự là gì?
Giả sử chúng ta đang ở branch A và muốn merge branch B vào branch A với câu lệnh:
```
git merge B
```
Sẽ có 2 trường hợp xuất hiện:
- B đang trỏ vào commit là commit cha của commit mà A đang trỏ vào. Ở đây Git sẽ tuân thủ `fast forward strategy` để merge B vào A. Kết quả sẽ xấy ra như ví dụ dưới đây:
    ```bash
    # A: C1 -> C2 -> C3
    # B: C1 -> C2 -> C3 -> C4 -> C5
    git merge B

    # result
    # A: C1 -> C2 -> C3 -> C4 -> C5
    # B: C1 -> C2 -> C3 -> C4 -> C5
    ```
- B đang trỏ vào commit `không` phải là cha của commit mà A đang trỏ vào. Ở đây Git sẽ thường tuân thủ `Recursive strategy` để merge từ B vào A. Kết quả sẽ xấy ra như ví dụ dưới đây:
    ```bash
    # A: C1 -> C2 -> C3
    # B: C1 -> C2 -> C4 -> C5
    git merge B

    # result
    # A: C1 -> C2 -> C3 ------> C35
                | -> C5 --/
    # B: C1 -> C2 -> C3 -> C4 -> C5
    ```
    bạn sẽ sinh thêm một commit mới C35, và khi nhìn vào cây commit chúng ta sẽ không hiểu commit này đang nói lên điều gì, và đây chính là nguyên nhân gây nên commit history không được clean. Trong những trường hợp này để tránh xuất hiện commit C35, `rebase` sẽ là một cách hiệu quả, cụ thể thế nào mình sẽ nói rõ ở phần sau của bài viết.


## Remote Tracking Branch là gì?

## Git Rebase là gì? Lúc nào dùng Rebase thay cho Merge.

## Kết bài.

## Tài liệu tham khảo
- https://git-scm.com/docs/git-merge
- https://git-scm.com/docs/git-rebase
---
layout:     post
title:      "Git rebase và git merge"
description: ""
date:    2023-10-12
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
Cho đến phần này của bài viết, mọi thứ mình đang làm trên repository để được thực hiện local, mọi thay đổi chỉ diễn ra trên máy tính cá nhân của mình. Tất nhiên Git không sinh ra chỉ để quản lý version của source code - cái chỉ mỗi mình contribute vào. Git được sinh ra để quan lý version của mã nguồn - được contribute bởi nhiều thành viên khác nhau. Để thấy được những thay đổi tạo ra bởi những thành viên khác nhau trong team, các thành viên sau khi tạo commit ở `local repository` phải đưa những thay đổi đó lên server (thông qua câu lệnh `git push` như hầu hết chúng ta đã quen thuộc). Repository ở server sẽ được gọi là `remote repository`.

Ở `local reposity`, chúng ta có chứa references của `remote repository`, những references này sẽ cập nhật state của `remote repository` thông qua câu lệnh `git fetch`, những references này chính là cái mà chúng ta gọi là `Remote Tracking Branch`. Ở trong thư mục `.git` vị trí của những `remote tracking branhes` này sẽ được nhìn thấy ở thư mục .git/refs/remotes như ví dụ dưới đây:

```bash
tree .git/refs/
├── remotes
│   └── origin
│       ├── HEAD
│       ├── staging
│       ├── master
│       ├── develop
│       ├── hotfix
│       │   ├── allow_testing_emails_blank
│       │   ├── disble_verifying_ssl_billing_event
│       │   ├── partner_api
│       │   └── set_conn_max_age
```

## Git Rebase là gì? Lúc nào dùng Rebase thay cho Merge.
Ở phần đầu tiên của bài viết, khi nói về `git merge` mình có nói đến việc khi `git merge` không tuân thủ `fast forward stradegy` nó sẽ tạo ra thêm những commit thừa, đồng thời làm cho lịch sử commit không còn tuyến tính nữa. Để giải quyết vấn đề này ta có thể sử dụng đến `git rebase`, mình xin được nhắc lại bài toán ở phần trước như sau:
- ở thời điểm t1, bạn fetch code từ nhánh develop của server về local, lúc này state của develop đang là:
    ```bash
    # develop
    C1 -> C2
    ```

- Bạn tạo ra một nhánh mới là feature1 ở local của bạn từ develop với câu lệnh `git branch feature1:origin/develop`, lúc này state của nhánh feature1 sẽ giống với develop:
    ```bash
    # feature1
    C1 -> C2
    ```
- Ở thời điểm t2, đồng nghiệp A của bạn cũng fetch code từ develop, và tạo branch feature2 trên máy của A. A tạo thêm 2 commit C3, C4 và push lên develop, lúc này state của develop sẽ như sau:
    ```bash
    # develop
    C1 -> C2 -> C3 -> C4
    ```

- Ở nhánh feature1 ở local của mình, mình cũng tạo thêm commit C5:
    ```bash
    # feature1
    C1 -> C2 -> C5
    ```

- Lúc này mình muốn đẩy code lên branch develop của server, mình sử dụng câu lệnh:
    ```bash
    git push origin feature1:develop
    ```
   Đáng tiếc là khả năng cao Git sẽ không cho phép bạn push code và raise lên lỗi `Git push non-fast-forward updates were rejected`, vì commit C5 mà feature1 đang trỏ vào không phải là cha của commit C4 mà develop server đang trỏ vào.

- Lúc này mình sẽ sử dụng `git rebase` để giải quyết vấn đề, đầu tiên mình sẽ fetch remote develop branch về local của mình, origin/develop branch trên local của mình sẽ được cập nhập trạng thái mới nhất của remote develop branch.
    ```bash
    git fetch origin develop
    ```
    Sử dụng `git rebase`:
    ```bash
    git rebase origin/develop
    ```
    Lúc này trạng mới của feature branch sẽ như sau:
    ```bash
    C1 -> C2 -> C3 -> C4 -> C5'
    ```
    Commit C5' được tạo ra và C5 đã biến mất, C5' sẽ mang theo mọi thay đổi từ C5, nhưng sẽ là cha của C4, điều đó cho phép ta có thể update code từ feature1 lên `remote develop branch` mà không làm mât đi tính tuyến tính của commit history.

Thông qua ví dụ trên, chúng ta cũng đã phần nào trả lời được câu hỏi: Lúc nào thì nên dùng rebase để thay thế cho merge. Thực tế thì rebase là một công cụ mạnh mẽ, cho phép chúng ta giải quyết được rất nhiều vấn đề, giúp ta tạo ra được một lịch sử commit rõ ràng và mạch lạc. Trong khuôn khổ bài viết này mình xin không đề cập thêm những ứng dụng của rebase nữa, vì nó quá dài để kể.


## Kết bài.
Thông qua 2 bài viết về Git Internal, mình đã đưa ra đến nhiều khái niệm quan trọng trong Git như: BLOB, tree, commit, giải thích git push, git rebase, remote tracking branch. Việc hiểu những khái niệm này là cần thiết đối với một software engineer - những người phải làm việc gới Git thường xuyên, nó sẽ mang lại sự tự tin lớn cho bạn khi làm việc với Git hơn rất nhiều.

Cảm ơn các bạn đã theo dõi bài viết, chắc chắn sẽ không thể tránh khỏi những thiếu sót, mình rất mong nhận được sự đóng góp của bạn đọc, và sẵn sàng sửa chữa.

## Tài liệu tham khảo
- https://git-scm.com/docs/git-merge
- https://git-scm.com/docs/git-rebase
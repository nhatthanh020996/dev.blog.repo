---
layout:     post
title:      "Git Internal (P1)"
description: "Git là một trong những công cụ quen thuộc nhất đối với bất kì một developer nào, chúng ta làm việc với nó gần như mỗi ngày, nhưng có một sự thật là có rất nhiều lập trình viên vẫn chưa đủ tự tin khi làm việc với Git. Trong bài viết này, mình xin chia sẻ những gì
xẩy ra phía sau những câu lệnh Git của chúng ta, tất nhiên ở một độ sâu nào đó (không đến mức phải đọc từng dòng code C, vì mình chưa đủ
năng lực để làm việc đó). Từ đây hi vọng chúng ta có thể hiểu đươc phần nào đó về cách Git hoạt động, cũng như tự tin hơn khi sử dụng nó."
date:    2023-10-07
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - Git
URL: "/git-internal/"
categories: [ Tech ]
---

## Mở đầu.
Chắc là chúng ta sẽ không mất nhiều thời gian để nói về Git nữa vì chúng quá phổ biến và gần gũi đúng không nào. Mình chỉ chia sẻ một chi tiết khá thú vị về nguồn gốc ra đời Git, Git được sáng tạo ra bởi Linus Torvalds - một cái tên quá nổi tiếng trong thế giới phần mềm rồi đúng không. Theo như những gì chia sẻ trong một cuộc phỏng vấn ở Ted Talk của tác giả, Git được ra đời vì Torvalds muốn tạo ra một công cụ nào đó giúp ông quản lý source code của Linux được hiệu quả hơn. Vâng nó chỉ ra đời vì mục đích cá nhân của Torvalds, nhưng sau đó đã trở
thành một trong những mã nguồn mở có ảnh hưởng nhất đối với lịch sử phát triển phần mềm cho đến ngày nay. 

## Git Internal.

### `.git` folder.
Ta sẽ nói về những khái niệm này thông qua ví dụ cụ thể sau đây.
Đầu tiên ta sẽ tạo một thư mục `example` và khởi tạo Git trong thư mục với câu lệnh sau:
```bash
git init example
```
Đứng ở thư mục `example` gõ câu lệnh:
```bash
ls -al
```
Ta sẽ thấy xuất hiện thư mục ẩn `.git` (trong hệ điều hành based-Unix 1 file ẩn sẽ được bắt đầu bằng dấu chấm, và nó sẽ không xuất hiện nếu
ta chỉ gõ câu lệnh `ls` thông thường), những gì diễn ra trong `.git` sẽ phản ánh cách mà Git đang vận hành. Git repository chỉ là một folder trong máy tính của chúng ta, trong ví dụ này là thư mục `example`, và `.git` là thư mục chứa metadata của repository đó. Có nghĩa là nếu chúng ta có bất kì một sự thay đổi nào trong repository, thì sự thay đổi đó sẽ được lưu lại dưới dạng những files trong `.git`. Hãy xem những gì xuất hiện trong thư mục `.git` với câu lệnh:
```bash
tree .git
```
Kết quả là:
```bash
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```
Trong những subdirectory này chúng sẽ tạm chỉ quan tâm đến 2 thư mục quan trọng nhất `objects` và `refs`.

### Working, Staging và Repository areas trong Git.
Trước khi quay trở lại với `.git` folder, chúng ta sẽ nói qua khái niệm về những areas trong git: Working, Staging và Repository. 
Mục đích của Git là tạo ra một version control system cho phép bạn theo dõi những sự thay đổi trong project của bạn qua thời gian. 
- Working directory - đơn giản chỉ là một directory trong hệ điều hành, directory này chứa project của bạn. Bạn mở 1 file trong project bằng IDE, bạn bắt đầu có những thay đổi trong file đó, lúc này bạn đang thao tác với Working directory.
- Staging area - để dễ hiều thì sau khi bạn thay đổi 1 file nào đó, hoặc fix một bug nào đó ... bạn gõ lệnh `git add .`, tất cả những files vừa thay đổi sẽ được add vào staging area.
- Repository area - Sau khi add vào staging area, bạn gõ lệnh `git commit -m "commit name"`, tất cả những files vừa thay đổi sẽ được lưu lại permanently trong Git, cái mà chúng ta vẫn gọi đó là commit.

Bây giờ, chúng ta hãy cùng theo dõi khi chúng ta gõ những câu lệnh `git add` và `git commit`, folder `.git` sẽ thay đổi như thế nào?

### Objects in Git.
Tạo 1 file `Readme.md` với câu lệnh `touch`
```bash
touch README.md
```

Dùng câu lệnh `git add README.md` để copy file `README.md` từ `working` area đến `staging` area.
```bash
git add README.md
```

Bây giờ, kiểm tra lại thư mục `.git` xem có gì thay đổi không, ta thu được kết quả:
```bash
├── objects
│   ├── e6
│   │   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```
Ta thấy rằng thư mục `objects` xuất hiện file `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391`, file này được gọi là một objects. Trong Git, có 3 loại
objects chính mà chúng ta cần quan tâm là `blob`, `tree`, và `commit` (ngoài ra còn một loại nữa là `tag`, nhưng trong bài viết này chúng ta không đề cập đến nó).
#### 1. BLOB.
Viết tắt của `binary large object`, mình sẽ hiểu blob đại khái là: khi ta gõ câu lệnh `git add README.md`, Git sẽ copy nội dung của 
file `README.md`, chú ý ở đây là chỉ nội dung của file mà không chứa những thông tin khác như tên file, người tạo, quyền ... sau đó đưa nội dung này vào một file trong thư mục `objects`, nội dung này sẽ được hash bằng thuật toán SHA1 trả ra một chuỗi gồm 40 ký tự hexa, lấy chuỗi ký tự này làm file name - Trong trường hợp trên là `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391`. Cá nhân mình không rõ thuật toán cụ thể phía trong Git là như thế nào, nhưng chúng ta chỉ cần quan tâm rằng `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391` là một `blob objects` - đại diện cho nội dung chứa trong file `README.md`. Do đó nếu ta có thêm 1 file `README1.md` có nội dung tương tự như `README.md`, sau khi thực hiện câu lệnh `git add README1.md`, trong thư mục objects sẽ không xuất hiện thêm một blob nào mới.

#### 2. Tree.
Sau khi thực hiện câu lệnh `git commit`, Git sẽ tạo ra một snapshot cho toàn bộ project tại thời điểm thực hiện commit, và `tree` là một object - chứa thông tin về cây thư mục của snapshot đó. Ta sẽ thấy rõ hơn với ví dụ sau:
```bash
git commit -m "Create empty readme"
```
Kiểm tra lại thư mục `.git` xem có gì thay đổi không, kết quả là:
```bash
├── objects
│   ├── 2d
│   │   └── 161ef3370a25a8c81123313d2b0a1f66800031
│   ├── e6
│   │   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
│   ├── f9
│   │   └── 3e3a1a1525fb5b91020da86e44810c87a2d7bc
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   └── master
    └── tags
```
`objects` xuất hiện thêm 2 files mới, vậy đâu là tree object trong 2 objects mới này, ta sử dụng câu lệnh `git cat-file  -t` để kiểm tra kiểu của từng object:
```
git cat-file  -t f93e3a
```
Kết quả trả về cho ta thấy object `f93e3a` (ta không cần phải copy hết 40 ký tự của mã SHA) - là một tree object. Ta sẽ kiểm tra nội dung trong file `f93e3a`, về cơ bản đây là 1 binary file nên ta không dùng những câu lệnh thông thường như `cat` để xem được mà sẽ sử dụng câu lệnh sau:
```bash
git cat-file -p f93e3a
```
Kết quả trả về:
```bash
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	README.md
```
Ta thấy tree object có references đến các blob objects, điều này nói lên rằng tree object sẽ chứa danh sách file + subtree trong project. Một file sẽ bao gồm metadata của file + blob object tương ứng của file đó.

Vì tree object bao gồm các thông tin trên, cho nên khi có bất kỳ một sự thay đổi nhỏ nào và tạo commit mới, ví dụ: thay đổi quyền truy cập của 1 file, thay đổi tên file... Thì ngay lập
tức một tree object mới sẽ được tạo ra và mã SHA sẽ khác hoàn toàn với mã SHA của object trước đó.

#### 3. Commit.
Commit object sẽ được sinh ra sau khi tạo ra commit, về cơ bản thì đây chính là đại diện cho một snapshot của project tại thời điểm thực hiện commit. Thông thường, trong 3 loại object chúng ta chỉ cần quan tâm đến loại object này. Bây giờ chúng ta sẽ xem trong một commit object sẽ chứa những nội dung gì nhé.
```bash
git cat-file -p 2d161e
```
Kết quả trả về:
```bash
tree f93e3a1a1525fb5b91020da86e44810c87a2d7bc
author thanhtn0209 <thanhtn1@galaxy.com.vn> 1689395943 +0700
committer thanhtn0209 <thanhtn1@galaxy.com.vn> 1689395943 +0700

Create empty readme
```
Nhìn vào kết quả trên, ta thấy rằng commit object sẽ references đến tree object tương ứng của nó, ngoài ra nó cũng bao gồm những thông tin về người tạo commit, mà message của commit. Ngoài ra, nó còn bao gồm 1 thông tin nữa là parent commit - mã SHA của commit trước khi thực hiện commit này. Trong ví dụ này ta không thấy xuất hiện parent commit vì đơn giản đây là commit đầu tiên nên nó không có parent commit.

Từ nội dung của commit ta có thể suy luận ra rằng, để có thể gen ra nội dung của project tại môt thời điểm nào đó, Git sẽ tìm commit object ứng với thời điểm đó, qua đó lấy được tree object, rồi từ tree object sẽ tiến hành lấy ra hết nội dung file + thư mục con, cuối cùng sẽ lấy ra hết những gì có trong project. Tất nhiên đó chỉ là suy đoán của cá nhân mình, còn để thực sự hiểu được Git thực hiện thế nào, ta cần có những tìm hiểu xa hơn.

## Branch là gì?
Git là một version control system, tức là chúng ta muốn tracking lại từng snapshot của project theo thời gian, và khi mà chúng ta muốn tìm lại một snapshot nào đó, ta sẽ phải references đến commit tương ứng với snapshot đó. Để làm điều đó ta sử dụng câu lệnh sau:
```bash
git checkout [commit's SHA]
```
Trong đó [commit's SHA] mà mã SHA của commit như đã đề cập ở phần trước.
Nhưng có một điều là mã SHA này có 40 ký tự hexa nên rất khó nhớ, vì vậy chung ta sẽ cần một cái tên để thay thế cho mã SHA này (tư tưởng gần giống như đặt tên miền). Branch ra đời vì mục đích đó, branch là một lightweight movable pointer (xin lỗi mình k biết gọi cái này là gì nên xin phép copy trực tiếp từ trang chủ của Git), khi được tạo ra nó sẽ trỏ vào commit mà bạn đang đứng hiện tại. Khi tạo ra commit mới, `branch` sẽ chủ động trỏ vào commit mới.

Ta tạo một `branch` từ một commit bằng câu lệnh sau:
```bash
git branch [Tên branch mới] [Tên của commit/hoặc tên của branch đã tồn tại]
```

Trong ví dụ ở phần trước, sau ghi tạo commit, trong thư mục `.git/refs/heads` ta thấy xuất hiện file master, ta sẽ thử xem có gì trong file master này:
```bash
cat ./git/refs/heads/master
```
Kết quả thu được là `2d161ef3370a25a8c81123313d2b0a1f66800031` ứng với mã SHA của commit đã được tạo ra ở phía trên.

## Git Push thực sự là gì?
khi bạn thực hiện lệnh `git push`, có bao giờ bạn nghĩ rằng sẽ push những files ở local lên remote repository? Không, thay vì push raw files này lên remote repository, Git sẽ chỉ push objects lên mà thôi. Khi ta gõ câu lệnh:
```bash
git push origin master
```
Đầu tiên Git sẽ kiểm tra xem, commit mà local branch đang trỏ tới có phải là con của commit mà remote branch đang trỏ tới hay không. Nếu không, mặc định thì bạn sẽ không được phép push lên, nếu vẫn muốn push lên thì bắt buộc chúng ta phải thêm option --force hoăc --force-with-lease, ý nghĩa của những options này mình sẽ nói rõ ở phần sau.

Khi được phép push code lên, thì git sẽ tìm những objects xuất hiện trong branch master ở local mà không xuất hiện ở trong branch master ở remote repository để push lên.

## Có thể mất code vì gõ sai những câu lệnh Git không?
Khi mới bắt đầu làm việc với Git, do chưa hiểu được bản chất nên có nhiều khi mình sợ rằng nếu không cẩn thận mình sẽ làm mất code của người khác hoặc code của chính mình. Điều đó làm cho mình chỉ giám sử dụng những câu lệnh cơ bản như: clone, pull, merge, push. Cho dù vậy thỉnh thoảng khi có sự cố xẩy ra, mình vẫn cảm thấy sợ và ngay lập tức tìm sự giúp đỡ.

Sau khi tìm hiểu về Git Internal, mình nhìn thấy rằng, những đơn vị tạo nên Git là objects, và chúng ta chỉ có thể tạo những objects mới, chứ không được sửa những objects đang tồn tại. Vì vậy mà sẽ không có chuyện mất code, đơn giản vì objects vẫn còn ở đấy.

## Kết bài.
Hi vọng sau khi đọc xong bài viết này chúng ta đã có thể nắm được về cơ bản Git đang hoạt động như thế nào, dó là tiền đề quan trọng để ta có thể hoàn toàn làm chủ được Git. Vì hạn hẹp về kiến thức nên có thể đâu đó trong bài viết có những kiến thức mình hiểu chưa đúng, mong ai đó đọc được sẽ để lại comments giúp mình. Cảm ơn vì đã theo dõi!

## Nguồn tham khảo.
- https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
- https://www.youtube.com/watch?v=lG90LZotrpo&t=420s&ab_channel=CS50




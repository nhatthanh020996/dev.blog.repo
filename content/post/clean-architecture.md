---
layout:     post
title:      "Clean Architecture"
description: "Mỗi lập trình viên luôn muốn tạo ra những phần mềm mà nó dễ dàng thay đổi lúc business logic thay đổi, hoặc yêu cầu kỹ thuật có sự thay đổi. 
Họ muốn phần mềm của họ dễ dàng mở rộng, thêm những tính năng mới mà không ảnh hưởng đến những tính năng đang chạy. Để làm được điều đó chúng ta cần tích luỹ 
nhiều kinh nghiệm, kiến thức. Vậy có một mô hình chung nào, hoặc một tư tưởng thiết kế nào giúp ta làm điều đó dễ dàng hơn. Trong bài viết này mình xin giới 
thiệu một kiến trúc như thế, nó gọi là Clean Architecture."
date:    2023-06-25
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - clean-architecture
    - python
    - django
URL: "/clean/architecture/"
categories: [ Tech ]
---
## Mở đầu.
Đầu tiên chúng ta phải thống nhất lại rằng Clean Architecture là 1 tên gọi được Robert Martin - một lập trình viên nổi 
tiếng người Mỹ đặt tên trong cuốn sách cùng tên nổi tiếng của ông, nó cũng giống như A architecture hay B architecture 
nào đó. Vì vậy Clean Architecture không nên hiểu là một kiến trúc tốt trong mọi hoàn cảnh. Dựa trên nhiều yếu tố như 
thời gian, độ phức tạp, tính thay đổi của dự án người lập trình viên sẽ chọn ra chọn đâu là kiến trúc code phù hợp nhất 
và nó không nhất thiết phải là Clean Architecture. Nhưng tất nhiên Clean Architecture sẽ có thể phần nào đó đảm bảo cho
chúng ta có được một kiến trúc dễ thay đổi, dễ mở rộng và nâng cấp.

Trong bài viết này mình muốn tập trung nói về cách để triển khai một project theo kiến trúc này thay vì nói nhiều về lý
thuyết - những thứ các bạn có thể tìm đọc trong cuốn sách của Robert Martin. Mình dùng Python + Django để thể hiện ý tưởng
của mình, tất nhiên các bạn có thể làm điều đó tương tự với những ngôn ngữ khác.

Khi đọc bài viết này, nếu bạn đọc có thể chỗ nào chưa được chính xác và hợp lý, xin hãy gửi ý kiến đóng góp đó cho mình 
qua email cá nhân của mình. Mình luôn sẵn sàng lắng nghe và thay đổi sai sót của bản thân, còn bây giờ hãy bắt đầu nào.

## Đặt vấn đề.
Giả sử chúng ta được nhận source code của một dự án nào đó đã được chạy nhiều năm, code hiện tại đang sử dụng Django (một 
web framework nổi tiếng của Python cho bạn nào chưa biết), team lead muốn chúng ta sử dụng FastAPI thay thế cho Django để 
tận dụng asynchronization giúp cải thiện hiệu năng của service. Vấn đề là business logic không có sự thay đổi nào, và bạn 
phải làm sao để có thể tái sử dụng lại được phần này trong service mới?

Một ví dụ thứ hai, lần này không phải là thay đổi framework hiện tại mà Tech Lead muốn bạn thay đổi database, thay vì dùng 
PostgreSQL ổng muốn bạn dùng MongoDB vì service này đòi hỏi phải ghi nhiều vào database cũng như muốn tận dụng những điểm 
mạnh khác của MongoDB. Vấn đề lúc này Django lại không có hỗ trợ MongoDB, vậy làm thế nào để bạn thực hiện ý tưởng đó với 
chi phí ít nhất?

## Ý Tưởng của Clean Architecture.
Clean Architecture vận dụng ý tưởng chia project ra thành nhiều lớp, các lớp này giao tiếp với nhau bằng một cách nào đó
mà việc thay đổi môt lớp không làm ảnh hưởng đến những lớp còn lại, cách phân lớp này có thể đươc minh hoá như hình bên 
dưới.
![](/img/clean-architecture/layers.png)

### Entities.
Trong mô hình trên Entities là những đối tượng sẽ xuất hiện trong trong project. Ví dụ: project của bạn là thiết kế một 
blog cá nhân, entities ở đây sẽ là author, post, comment, ... Có thể nhiều bạn sẽ bảo Entities ở đây chính là Models. 
Không entity không phải là model, chúng ta nhớ lại khái niệm về model trong những web framework mà chúng ta thường gặp, 
model là 1 thực thể được map từ database và lập trình viên sẽ thông qua những model này để giao tiếp với database được
dễ dàng hơn. Entity không phải như vậy, Entity là khái niệm được xây dựng đứng trên quan điểm sản phẩm, không có dính 
dáng gì đến kỹ thuật. Chỉ đơn giản là nếu như bạn nói về một blog cá nhân thì bạn sẽ nghĩ rằng trên blog đó authors, 
có các bài posts, có những lời comments, và blog này hoạt động chính là lúc những thực thể này đang tương tác với nhau. 
Đừng lo nếu bạn đọc cảm thấy khó hiểu ở chỗ này, vì trong ví dụ cụ thể ở phần tiếp theo, bạn sẽ thấy mọi thứ được rõ ràng hơn.

### Usecases.
Usecases layer chứa những yêu cầu về mặt business logic của project và không liên quan gì đến bất kỳ một ý tưởng 
kỹ thuật nào cả. Ví dụ khi xây dựng một blog cá nhân chúng ta sẽ có những yêu cầu:
- Hiện thị tất cả các bài viết liên quan đến một từ khoá nào đó.
- Độc giả có thể comment bằng tài khoản google.
- Độc giả có thể vote sao để đánh giá chất lượng của bài viết.

Những phần code bao gồm Entities và Usecases chúng ta có thể gọi đó là Internal System. Những phần này không có dính dáng 
gì đến bất kỳ với 1 yếu tố kỹ thuật nào cả. Chính vì cách chia như vậy, chúng ta có thể giải quyết được vấn đề thứ nhất 
trong phần đặt vấn đề.

### External System.
Trước khi nói đến lớp thứ 3 của mô hình, mình xin được nói lớp ngoài cùng trước. Lớp ngoài tập hợp 
rất nhiều thành phần bao gồm: database, web framework, UI, platform, cache, ... Chúng ta gọi lớp 
này là External System để phân biệt rõ với 2 lớp trước (Internal System = Entities + Usecases).
Điều đang chú ý là ở lớp này, hầu hết mọi thứ đều là những vấn đề liên quan đến các kỹ thuật, công
cụ trong lập trình và là nơi cần thay đổi để để cải thiện hiệu suất của service, như những yêu cầu của tech lead mình từng đề cập ở phần đặt vấn đề.

### Adapters.
Đây là layer thứ ba của mô hình, layer giúp layer phía trong có thể giao tiếp với layer phía ngoài. Ví dụ với những yều cầu về business logic như vậy, chúng ta có thể chọn tech stack của layer
phía ngoài là (Django + PostgreSQL + Redis) hoặc (FastAPI + MongoDB + Redis), ... Để làm rõ ý tưởng
của layer này, các bạn hãy đọc phần triển khai với ví du thực tế trong phần sau của bài viết. 

### Nguyên tắc vàng khi các layer giao tiếp với nhau trong Clean Architecture.
![golden rule](/img/clean-architecture/goden-rule.png)


Nếu bạn implement code trong một layer nào đó, nếu bạn muốn sử dụng những đối tượng thuộc các layers phía bên trong so với layer đang đứng hiện
tại, bạn có thể trực tiếp khởi tạo những đối tượng đó. ví dụ: giả sử bạn đang implement code ở external layer, bạn có thể sử dụng những đối tượng thuộc Usecase layer bằng cách khởi tạo trực tiếp

Nếu bạn implement code trong một layer nào đó, nếu bạn muốn sử dụng những đối tượng thuộc các layer ở phía bên ngoài so với layer đang đứng hiện
tại, bạn chỉ được sử dụng các đối tượng thông qua interface.

## Ý tưởng thiết kế hiện tại của Django.
Chúng ta sẽ cùng nhìn lại về Django - python web framework phổ biến nhất hiện tại xem tư tưởng thiết kế của Django là như thế nào. Mình sẽ tóm gọn lại tư tưởng của Django với mô hình dưới đây:

![](/img/2018-03-29-what-is-service-mesh-and-istio/microservice.PNG)

Từ mô hình trên bạn có thể thấy rằng, trong Django layer Entities và External được gắn liền với nhau, cụ thể Entities và Database gắn chặt với nhau và tạo thành cái mà chúng ta gọi là Model. Có nhiều bạn sẽ phản đối mình và nói rằng không, model là một interface và nó không gắn chặt
với một database cụ thể nào cả. Nhưng không, nếu các bạn đọc source code của Django, bạn sẽ thấy được phần này của Django chỉ được phát triển 
cho các cơ sở dữ liệu dạng SQL. 

Trong Django, nếu bạn dùng ORM của nó để viết business logic thì bạn 
đang mix cả business logic - UseCases layer với database = External System.

Nhiều bạn đọc đến đây sẽ nghĩ rằng, tại sao một framework phổ biến, nổi tiếng như Django 
lại không theo cái nguyên tắc Clean Architecture? Chắc Clean Architecture là thứ tào lao rồi :(

Như mình đã từ nói ở phần đầu của bài viết, Clean Architecture không chắc là tốt, và những phần mềm không
tuân thủ nó cũng không chắc là không tốt. Điều quan trọng là kiến trúc code của chúng ta có giúp ta thực
hiện được mục tiêu của chúng ta hay không. Ở Django, những lập trình viên đầu tiên của Django đã phát 
triển nó để đạt được mục tiêu giúp người dùng - những developers có thể nhanh chóng xây dựng được một
website với đầy đủ tính năng, dễ dùng và dễ hiểu. Mục tiêu của Django không phải là tạo ra một framework
với nguyên lý giúp lập trình viên dễ dàng thay đổi, mở rộng lúc business logic thay đổi và mở rộng.

Thực tế có rất nhiều web framework xây dựng kiến trúc giống với tư tưởng của Clean Architecture, ví dụ: 
Kratos, go-micro của Go. Bạn đọc có thể kiểm chứng lúc đọc qua kiến trúc của những framework này, còn
bây giờ mình sẽ đến với phần tiếp theo - xây dựng một project vừa tận dụng một số thành phần của Django
vừa theo tư tưởng của Clean Architecture.

## Triển khai Clean Architecture với Django.
Trong phần này chúng ta sẽ đi xây dưng một blog cá nhân. Với một blog cá nhân thông thường, nó sẽ có 
những tính năng cơ bản như sau:
- Tác giả post bài.
- Tác giả thêm, xoá, và chỉnh sửa bài viết.
- Đôc giả đọc bài.
- Độc giả tìm kiếm list những bài viết liên quan đến một tag name nào đó.
- ...
Ở phần này mình chỉ đưa ra các bước để xây dựng code implement một tính năng đơn giản đó là list ra danh sách bài 
viết có trong blog. Những tính năng còn lại có thể implement bằng cách thức gần như tương tự.

### External System.
Web framework nằm ở External System chính là nơi đầu tiên tiếp nhận HTTP request, và ở đây nhiệm vụ của web framework
chỉ là tiếp nhận HTTP request. Phần xử lý logic sẽ được gọi xuống UseCase layer để xử lý, sau đó web framework nhận lại
kết quả trả ra từ UseCase và tạo HTTP response.

![](/img/clean-architecture/web-usecase.png)

Ở đây ban đọc có thể thấy 1 instance `post_repository` và `list_posts_usecase` được khởi tạo trực tiếp từ External System,
nó tuân theo goden rule mà mình đã nói đến ở phần trước.
```python
from internal.usecases import ListPostsUsecase


class Post(View):
    def get(self, request):
        limit = request.query_params.get('limit')
        page = request.query_params.get('page')

        # initialize an instance of ListPostsUsecase
        post_repository = DjangoPostRepositoryImpl()
        list_posts_usecase = ListPostsUsecase(post_repository)
        list_posts_usecase.with_params(page, limit)
        usecase_result = list_posts_usecase.execute()

        response = Serializer(usecase_result).data
        return response
```

### Usecases.
UseCase chịu trách nhiệm xử lý business logic, UseCase gọi vào database lấy dữ liệu để phục vụ cho việc xử lý business logic.
![](/img/clean-architecture/usecase-database.png)

Do database nằm ở External System layer nên UseCase phải giao tiếp với database thông qua 1 interface.
![](/img/clean-architecture/usecase-adapter.png)

Trong đoạn code dưới đây, bạn thấy rằng `IPostRepository` là một interface giúp UseCase có thể giao tiếp được với database. Việc
sử dụng một interface như thế giúp decoupling Usecase và database, kỹ thuật này được sử dụng phổ biến trong lập trình hướng đối
tượng, và được biết đến với tên gọi là Dependency Inversion Principle.
```python
class ListPostsUsecase:
    def __init__(self, post_repository: IPostRepository):
        self._post_repository = post_repository

    def with_params(self, page, limit):
        self._page = page
        self._limit = limit
        return self

    def execute(self):
        return self._post_repository.posts(page=self._page, limit=self._limit)
```
Usecase sẽ tương tác với database thông qua một interface như `post_repository` trong đoạn code trên. Kết quả trả ra từ những function được gọi
qua interface này sẽ được biểu diễn bằng Entity hoặc các kiểu dữ liệu, cấu trúc dữ liệu cơ bản khác trong Python như: `int`, `string`, `list`,
`dict`. Bây giờ, chúng ta sẽ tiếp tục xây dựng IPostRepository.

### Adapter Interface.
Chúng ta sẽ tiếp tục xây dựng `IPostRepository` và những class implement interface này. Ở đây mình implement class `DjangoPostRepositoryImpl`
class này tận dụng ORM của Django để lấy dữ liệu trực tiếp từ database.
![](/img/clean-architecture/usecase-adapter-database.png)
```python
import abc
from entities import PostEntity
from ..models import Post as ORMPost


class IPostRepository(abc.ABC):
    @abstractmethod
    def _to_entity(self, post: Post):
        raise NotImplementedError()

    @abstractmethod
    def _to_entities(self, posts):
        raise NotImplementedError()

    @abstractmethod
    def posts(self, **kwargs):
        raise NotImplementedError()


class DjangoPostRepositoryImpl(IPostRepository):
    def _to_entity(self, post: Post):
        return PostEntity(id=post.id, title=post.title, content=post.content,
                          created_at=post.created_at,
                          updated_at=post.updated_at)

    def _to_entities(self, posts):
        return [self._to_entity(post) for post in posts]

    def posts(self, **kwargs):
        page = kwargs.get('page') if kwargs.get('page') is not None else 0
        limit = kwargs.get('limit') if kwargs.get('limit') is not None else DEFAULT_LIMIT
        start_item = page * limit
        end_item = page * limit + limit
        posts = ORMPost.objects.all()[start_item:end_item]
        return self._to_entities(posts)
```
### Entities.
```python
import attr
from datetime import datetime


@attr.s(frozen=True)
class PostEntity:
    id: int = attr.ib()
    title: str = attr.ib()
    content: str = attr.ib()
    created_at: datetime = attr.ib(default=datetime.now())
    update_at: datetime = attr.ib(default=datetime.now())
```

## Phân tích ưu và nhược điểm của Clean Architecture.
### Ưu điểm.
- Nếu để ý thì ta sẽ thấy Clean Architecture sử dụng 5 nguyên tắc SOLID để thiết kế một mã nguồn giúp ta dễ dàng chỉnh sửa, mở rộng tính năng.
ví dụ: usecase giao tiếp với database thông qua interface chính là áp dụng nguyên tắc thứ 5 - Dependency Inverson Priciple. Mà việc sử dung
Dependency Inversion cũng đã giúp code của bạn thoả mãn các quy tắc khác của SOLID.

- Sau khi project đã trải qua thời gian ban đầu và bước vào quá trình ổn định, việc có được một thiết kế tốt sẽ giúp chúng ta giảm được rất nhiều
chi phí khi cần chỉnh sửa tính năng cũ cũng như thêm vào những tính năng mới.

- Clean Architecture giúp bạn cực kỳ thuận lợi để follow Test Driven Development. Bạn dễ dàng test từng layer một mà chưa cần phải implement những layer còn lại.
### Nhược điểm.
- Tốn kém chi phí ban đầu (phải suy nghĩ để có một thiết kế hợp lý, thời gian implement lâu vì số lượng code lớn) vì vậy với những project đòi hỏi
triển khai nhanh thì Clean Architecture không thể làm được.
- Thiết kế theo Clean Architecture có thể gây khó khăn cho những lập trình viên mới vào ngành, vì nó đòi hỏi phải có kiến thức về OOP khá vững.

## Kết luận.
Clean Architecture là một đề tài rộng, và bản thân mình rõ ràng chưa thể nắm được hết cách triển khai của nó, vì vậy trong bài viết này
thực sự có nhiều đoạn mình không thể diễn đạt ra được thật mạch lạc bằng ngôn từ của mình. Nếu bạn đọc đến đây vẫn cảm thấy khó hiểu
thì hãy tham khảo thêm những nguồn tài liều mà mình trích dẫn ở sau đây nhé.

## Tài liệu tham khảo.
- https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164
- https://www.youtube.com/watch?v=C7MRkqP5NRI&pp=ygUZZGphbmdvIGNsZWFuIGFyY2hpdGVjdHVyZQ%3D%3D
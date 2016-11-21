 ###  I. Giới thiệu

Là một lập trình viên Ruby, chắc hẳn đã không ít lần bạn gặp lỗi `NoMethodError: undefined method "abc" for nil:NilClass`. Đây là một exception khá phổ biến ở trong Ruby mà mình chắc hẳn chúng ta ai cũng đã từng gặp phải.
Mỗi lần phải đối mặt với exception này, bạn đã giải quyết thế nào? Trước đây, mình đã nghĩ ngay đến việc dùng `try` để giải quyết vấn đề này, cho đến khi lỗi này lại lặp lại ở 1 chỗ khác, với 1 đối tượng khác. Sử dụng những method như `try` chỉ là các giải pháp tạm thời, không thể giải quyết được tận gốc của vấn đề. Đến đây, có lẽ ta nên đặt câu hỏi: Tại sao ta lại gặp phải lỗi này? Nguyên nhân chính ở đây là gì? Liệu có cách nào loại bỏ nó 1 cách triệt để không? ...

Trong bài viết này, chúng ta sẽ cùng nhau tìm hiểu về `nil` exception và các cách xử lý khi gặp các trường hợp như vậy.


 ### II. Các vấn đề thường gặp với Nil và cách giải quyết.

Chúng ta cùng xem qua một đoạn code ví dụ gây ra exception này.

```Ruby
# Controller
class SomeController
  def show
   @view_object = SomeViewObject.new params
  end
end

# View object
class SomeViewObject
  attr_reader :email

  def initialize params
    @email = params[:emal]
  end

  def user
    @user ||= User.find_by email: email
  end

  def user_name
    user.name
  end
end

# Views
<%= @view_object.user_name %>

```

Nếu chạy đoạn code trên, khả năng chúng ta gặp lỗi `NoMethodError: undefined method 'name' for nil:NilClass` ở ngoài view là khá cao. Khi gặp lỗi này, câu hỏi đầu tiên mà chúng ta nghĩ tới đó là tại sao `user` lại `nil`? Dò lại đoạn code 1 chút, chúng ta sẽ đoán vấn đề nằm tại hàm `user`, nơi chứa method `find_by`. Exception này bị raise lên có thể do `email` không đúng nên không thể tìm ra user này trong bảng User, hoặc do chính `email` đang `nil`(điều này rất có thể xảy ra vì chúng ta đang lấy `email` từ hash `params`). Sau 1 chút thời gian debug, chúng ta có thể tìm ra vấn đề, nhưng ta nên giải quyết thế nào? Có nên bắt ngoại lệ tại đúng nơi xảy ra lỗi này không? Chúng ta cùng xem qua 1 vài giải pháp:

#### **Use the right set of methods**

Nếu để ý một chút, bạn sẽ phát hiện ra: phần lớn lỗi `nil` trong Rails application đến từ việc truy vấn phần tử không tồn tại trong 1 hash hoặc ActiveRecord finders không tìm thấy record phù hợp.

* Đối với Hash, ta có thể fix lỗi này khá đơn giản bằng cách sử dụng `Hash#fetch` thay cho `Hash#[]`. Method `fetch` hoạt động giống như `[]` nhưng thay vì trả ra `nil` nếu cặp key-value không tồn tại, nó sẽ raise lên lỗi `KeyError`. Bạn cũng có thể trả về 1 giá trị mặc định, hoặc raise lên 1 custom error, chỉ cần truyền nó vào hàm như 1 tham số :

```Ruby
params.fetch(:email, "default@email.com")
params.fetch(:email) {raise EmailNotFoundError}
```

* Đối với ActiveRecord, ta có thể raise lỗi với sự hỗ trợ từ Rails thông qua `ActiveRecord::RecordNotFound` nếu như không tìm thấy record thay vì trả về `nil`. Thay vì dùng `find_by`, hãy dùng `find_by!`

```Ruby
User.find_by! email: email
```

Sử dụng các phuơng thức 1 cách đúng đắn sẽ giúp bạn tiết kiệm được rất nhiều thời gian trong việc tìm ra vấn đề gây ra `nil` exception. Tuy nhiên, trong trường hợp `user` có thể `nil`, bạn có thể giải quyết vấn đề này ở View:

```Ruby
<% if @view_object.user %>
  <%= @view_object.user_name %>
<% else %>
  Anonymous user
<% end %>
```
Tới đây thì bạn đã có thể giải quyết được vấn đề với `nil` nhưng liệu đây có phải là một giải phấp thực sự tốt?

#### **The Null Object Pattern**

Null Object là một object đã được implements như một interface với các phương thức để giải quyết một số trường hợp đặc biệt. Ví dụ dưới đây sẽ giúp bạn dễ hiểu hơn:

```Ruby
# View object
def user
  @user ||= User.find_by(email: email) || NullUser.new
end

# Null Object
class NullUser
  def name
    "Anonymous user"
  end
end

# View
<%= @view_object.user.name %>

```
Không cần phải check điều kiện gì, không bị gặp phải lỗi `nil` và code nhìn sẽ khá gọn và đẹp.

Bây giờ, chúng ta sẽ xét thêm một vài trường hợp khác. Nhiệm vụ đầu tiên là show ra tất cả các comments của user đó.
```Ruby
class NullUser
  ...

  def comments
    []
  end
end
```
Việc add thêm method comments như trên có vẻ giải quyết được vấn đề, tuy nhiên nếu chúng ta muốn sử dụng các phuơng thức như `where`, `order` hay 1 scope nào đó để lấy ra 10 comments mới nhất thì sao? Cách đơn giản là có thể thêm 1 hàm `ten_last_comments` vào class User và NullUser. Tuy nhiên cách này khá mất công vì nếu ta cần dùng các scope khác, sẽ lại phải add thêm các hàm khác vào trong class `NullUser`. Thật may là Rails 4 cho phép chúng ta giải quyết vân đề này bằng `NullRelation`:
```Ruby
class NullUser
  ...
  def comments
    Comment.none
  end
end
```

Bây giờ thì bạn đã có thể gọi bất cứ một relation methods nào trên 1 null user's comments. Nhờ có `NullRelation`, chúng ta có thể cover được khá nhiều trường hợp.
Mở rộng ra một chút, giả sử ta có làm 1 blog, người dùng có thể gửi comments. Bỗng nhiên 1 ngày họ muốn xóa bỏ tài khoản, tuy nhiên, nếu chúng ta vẫn muốn giữ lại tất cả các comments liên quan tới người dùng đó, chúng ta sẽ phải tìm được cách để implement một cách hợp lý.

```Ruby
<%= @comment.author.name %>
<%= @comment.author.email %>
```
Giả sử ở ngoài View chúng ta đang in ra thông tin của người dùng như trên. Khi người đó xóa tài khoản, `Nil` error sẽ xuất hiện. Chúng ta có thể add thêm điều kiện check ở ngoài view, hoặc sử dụng `try`.  Tuy nhiên có 1 phương pháp tốt hơn là dùng `delegate`.

```Ruby
# Comment.rb
class Comment < ActiveRecord::Base
  delegate :email, to: author, prefix: true, allow_nil: true
end

# View
<%= @comment.author_email %>
```
Bằng cách sử dụng `allow_nil: true`, hàm `author_email` sẽ trả về nil nếu `author` không tồn tại. Như thế chúng ta sẽ tự bảo vệ mình từ lỗi `NoMethodError`.

Null Object đã giúp ta giải quyết khá nhiều vấn đề, Tuy vậy, trong 1 vài trường hợp, khi bạn muốn tạo 1 object mà có thẻ nhận vào 1 null object và muốn assign nó tới 1 model khác như: `Comment.new(author: NullUser.new`, bạn có thể sẽ gặp lỗi: `ActiveRecord::AssociationTypeMismatch`. Để có thể linh hoạt hơn trong việc giao tiếp với ActiveRecord, ta nên viết 1 module riêng để xử lý:
```Ruby
module NullObjectPersistable
  extend ActiveSupport::Concern

  included do
    def self.mimics_persistence_from(real_model_class)
      @real_model_class = real_model_class
    end

    def self.real_model_class
      @real_model_class
    end

    def self.table_name
      @real_model_class.to_s.tableize
    end

    def self.primary_key
      "id"
    end
  end

  def real_model_class
    self.class.real_model_class
  end

  def id
  end

  def [](*)
  end

  def is_a?(klass)
    if klass == real_model_class
      true
    else
      super
    end
  end

  def destroyed?
    false
  end

  def new_record?
    false
  end

  def persisted?
    false
  end
end

# Null object
class NullUser
  include NullObjectPersistable

  mimics_persistence_from User
end
```
Như vậy, ta đã add đầy đủ các methods để ActiveRecord có thể raise ra các lỗi thích hợp. Null Object bây giờ đã khá hoàn thiện, bạn có thể đem vào sử dụng để tránh các lỗi liên quan đến `nil class`. :)

 ### III. Kết luận

`Nil` là 1 lỗi khá phổ biến trong Rail application. Để giải quyết vấn đề này có nhiều phuơng pháp. Tùy vào mức độ ảnh hưởng của lỗi này mà bạn nên lựa chọn cách giải quyết phù hợp. Bài viết trên tập trung giới thiệu về  việc xây dựng và sử dụng Null Object pattern thông qua 1 vài ví dụ. Hy vọng bài viết sẽ cho bạn 1 hướng giải quyết mới trong việc xử lý `nil` error.

Cám ơn các bạn đã đọc bài viết.
![59659170.jpg](https://viblo.asia/uploads/images/15710dd85247a62e3aa5a11089b38835729597e9/e5c9fbd6124b823d0e21996a661d70edb7ab9a01.jpg)

Nguồn: http://blog.ragnarson.com/2015/05/06/problems-with-nil.html

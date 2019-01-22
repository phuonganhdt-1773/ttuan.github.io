---
layout: post
title: 3 ActiveRecord Mistakes that slow down Rails App
tags:
- Rails
- Ruby
---

## 1. Mở đầu

Mình đã code Ruby on Rails khá lâu, tuy nhiên có nhiều điều mình vẫn chưa thực sự hiểu, ví dụ như ActiveRecord execute SQL query như thế nào? Và mình tin rằng cũng còn khá nhiều lập trình viên khác cũng không để ý tới điều này. Trong bài viết dưới dây, chúng ta sẽ cùng tìm hiểu thêm về ActiveRecord qua một số trường hợp hay gặp: Sử dụng sai method `count`, sử dụng `where` để lấy subsets, và sử dụng `present?`. Bạn có thể tạo ra lỗi N+1s queries nếu sử dụng sai 3 methods này đấy ;)

ActiveRecord thực sự rất tốt. Tuy nhiên, nó khá trừu tượng, nó tách chúng ta ra khỏi SQL queries. Do đó, nếu bạn không thực sự hiểu cách ActiveRecord hoạt động, có thể nó sẽ dẫn đến việc chạy 1 số câu SQL không cần thiết.

Một trong những lỗi hay gặp nhất là việc sử dụng những câu SQL không cần thiết (đã lấy ra records 1 lần rồi nhưng lần sau lại gọi lại để lấy tiếp). Thông thường thì các câu SQL này sẽ xuất hiện nhiều trong controller và trong các partial view.

Hôm nay, chúng ta sẽ cùng tìm hiểu về cách implement, cách sử dụng của 3 methods - những methods có thể dẫn đến gọi các câu query không cần thiết trong Rails app: `count`, `where` và `present?`

## 2. 3 Mistakes

### 2.1 Rule of thumb
Vậy, làm sao để chúng ta biết 1 câu query là không cần thiết?

Mình thường dựa vào 1 rule để cân nhắc xem query đó có cần thiết hay không. Ý tưởng là: **1 action trong  Rails controller chỉ NÊN execute 1 SQL query cho mỗi table**. Nếu bạn thấy nhiều hơn 1 SQL query/ table, bạn có thể tìm cách để giảm 1,2 queries. Số lượng các queries trên mỗi bảng được show rất rõ ràng trên NewRelic, ví dụ:

![](https://www.speedshop.co/assets/posts/img/nplusoneposts-7a096c5fa29cd74436d17d40baf35963b98bb92a47f9af7ad56c650ed8dfce68.png)

Một rule khác đó là: **Các câu queries NÊN được execute trong nửa đầu của controller action's response, không nên gọi queries trong partial view**. Các câu query được gọi ở trong view rất dễ dẫn đến N + 1s. Nếu trên development logs, chúng ta có thể dễ dàng bắt gặp những đoạn log như:

```
User Load (0.6ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = $1 LIMIT 1  [["id", 2]]
Rendered posts/_post.html.erb (23.2ms)
User Load (0.3ms)  SELECT  "users".* FROM "users" WHERE "users"."id" = $1 LIMIT 1  [["id", 3]]
Rendered posts/_post.html.erb (15.1ms)
```

Hãy chú ý preload data trước khi in ra view. Đó là 1 số rule chúng ta nên chú ý. Còn bây giờ, hãy xem thử 3 methods: `count`, `where` và `present?` để xem lý do tại sao chúng hay dẫn tới các câu SQL k cần thiết nhé.

### 2.2 `count`
`.count` sẽ luôn execute lệnh `COUNT` trong SQL mỗi khi bạn gọi.

Điều này có lẽ cũng có khá nhiều người biết, vì mình thấy có nhiều article nói tới điều này khi so sánh `count` với `length` và `size`. Hãy luôn nhớ 1 điều là: **Chỉ sử dụng `count` nếu bạn muốn execute SQL COUNT ngay lúc đó**.

Thông thường, chúng ta dùng `count` 1 association, sau đó sẽ sử dụng lại ở trong view. Điều này sẽ dẫn tới việc thừa 1 lần count. Ví dụ:

```
# _messages.html.erb
# Assume @messages = user.messages.unread, or something like that

<h2>Unread Messages: <%= @messages.count %></h2>

<% @messages.each do |message| %>
blah blah blah
<% end %>
```

Ở đây sẽ execute 2 commands, `COUNT` và `SELECT`. `COUNT` được execute bởi lệnh `@messages.count`, còn `@messages.each` sẽ execute SELECT để load tất cả message. Nếu chúng ta thay đổi thứ tự code trong partial và thay đổi `count` thành `size` thì sẽ loại bỏ được `COUNT` query và giữ lại `SELECT`:

```
<% @messages.each do |message| %>
blah blah blah
<% end %>

<h2>Unread Messages: <%= @messages.size %></h2>
```

Có điều này là do method `size` được định nghĩa như sau: [source](https://github.com/rails/rails/blob/94b5cd3a20edadd6f6b8cf0bdf1a4d4919df86cb/activerecord/lib/active_record/relation.rb#L210)

```ruby
# File activerecord/lib/active_record/relation.rb, line 210
def size
  loaded? ? @records.length : count(:all)
end
```

Nếu relation đã được load, chúng ta sẽ gọi `length` (1 method của Ruby để lấy ra size của 1 mảng), còn nếu ActiveRecord::Relation chưa được load, chúng ta sẽ gọi lệnh `COUNT`.

Đây là cách [count implemented](https://github.com/rails/rails/blob/94b5cd3a20edadd6f6b8cf0bdf1a4d4919df86cb/activerecord/lib/active_record/relation/calculations.rb#L41) :

```ruby
def count(column_name = nil)
  if block_given?
    # ...
    return super()
  end

  calculate(:count, column_name)
end
```

và đây là cách [calculate implementation](https://github.com/rails/rails/blob/94b5cd3a20edadd6f6b8cf0bdf1a4d4919df86cb/activerecord/lib/active_record/relation/calculations.rb#L131). Nó không được cache hay lưu trữ gì, chỉ đơn giản là execute SQL calculation mỗi khi được gọi. Tuy nhiên, như ở trong ví dụ trên kia, chúng ta move phần tính `@messages.size` xuống bên dưới đoạn chạy `each`, như thế khi chạy vòng `each`, `@messages` đã được load, và chúng ta có thể chạy lệnh `size` để lấy ra. Tuy nhiên, nếu chúng ta vẫn muốn in ra size trước khi chạy each thì sao? Hãy dùng `load` method.

```
<h2>Unread Messages: <%= @messages.load.size %></h2>

<% @messages.each do |message| %>
blah blah blah
<% end %>
```
`load` sẽ thực hiện load luôn messages, thay vì load lazy. Nó zẽ trả về 1 ActiveRecord::Relation. Do đó, khi `.size` được gọi, `@messages` đã được load sẵn, k cần 1 query để tính size nữa.

Vậy, nếu chúng ta sử dụng `messages.load.count` thì câu lệnh `.count` có bị tính ra 1 câu query khác không? Câu trả lời là có. Vậy khi nào thì câu query `.count` mới bị ignore? Chỉ khi result được cached bởi `ActiveRecord::QueryCache`, khi có 2 câu SQL giống hệt nhau được gọi:

```
<h2>Unread Messages: <%= @messages.count %></h2>

... lots of other view code, then later:

<h2>Unread Messages: <%= @messages.count %></h2>
```

Theo quan điểm của mình, tốt nhất hãy dùng `.size` thay thế cho `.count` ở mọi chỗ. Bạn chỉ nên dùng `count` khi chúng ta không thực sự load hết tất cả các record của 1 associatión mà ta đang tính `count`. Ví dụ:

![](https://www.speedshop.co/assets/posts/img/rspecview-9229ad8cdcaaf7aad42c0403811ce5eee825eb3fd49fc4536ce992b4fff7c6cd.png)

Trong "version" list, view đã in ra số `count` để lấy tổng số releases của 1 gem.

```
<% if show_all_versions_link?(@rubygem) %>
  <%= link_to t('.show_all_versions', :count => @rubygem.versions.count), rubygem_versions_url(@rubygem), :class => "gem__see-all-versions t-link--gray t-link--has-arrow" %>
<% end %>
```
Trong view này, nó không load tất cả versions của 1 `@rubygem`, nó chỉ lấy ra 5 hoặc versions gần nhất. Nên nếu ta gọi `count` ở đây sẽ là hợp lý nhất (mặc dù dùng `.size` cũng sẽ execute điều tương tự).

### 2.3 `where`
Giả sử chúng ta có 1 đoạn code như sau, trong file `_post.html.ẻb`

```
<% @posts.each do |post| %>
  <%= post.content %>
  <%= render partial: :comment, collection: post.active_comments %>
<% end %>
```
và trong file `post.rb`

```ruby
class Post < ActiveRecord::Base
  def active_comments
    comments.where(soft_deleted: false)
  end
end
```
Nếu bạn nói: Việc này sẽ dẫn tới SQL query bị execute mỗi khi render ra post partial. Bạn đã đúng. `where` luôn luôn dẫn tới việc gọi query. Vậy nếu chúng ta gọi `includes` hoặc sử dụng preloading methods khác trong controller thì sao? bạn cũng k thể loại bỏ được việc `where` execute a query.

Điều này cũng xảy ra khi bạn gọi scopes, vd trong model Comment, ta viết:

```
class Comment < ActiveRecord::Base
  belongs_to :post

  scope :active, -> { where(soft_deleted: false) }
end
```
Do đó, chúng ta có 2 rules: **Không gọi scopes trong associations khi chúng ta render collections** và **Không dùng query method (vd `where`) trong instance methods của 1 ActiveRecord::Base class**

Gọi scopes trong 1 associations khiến chúng ta không thể preload được result. Trong ví dụ bên trên, chúng ta có thể preload comments của 1 post, nhưng chúng ta khôgn thể preload `active` comments của 1 post được. Do đó, chúng ta phải quay lại db, execute new queries cho mỗi phần tử của collection.

Vậy làm thế nào để fix N+1 cho render collection? Cách tốt nhất mà mình tìm được là ở trong [bài viết](https://www.justinweiss.com/articles/how-to-preload-rails-scopes/) này. ý tưởng là chúng ta sẽ tạo ra 1 association mới, nơi chúng ta có thể preload.

```
class Post
  has_many :comments
  has_many :active_comments, -> { active }, class_name: "Comment"
end

class Comment
  belongs_to :post
  scope :active, -> { where(soft_deleted: false) }
end

class PostsController
  def index
    @posts = Post.includes(:active_comments)
  end
end
```
View ở đây không cần thay đổi gì, nhưng bây giờ nó sẽ chỉ execute 2 SQL queries, 1 query load trong bảng Post và 1 trong bảng Comment.

```
<% @posts.each do |post| %>
  <%= post.content %>
  <%= render partial: :comment, collection: post.active_comments %>
<% end %>
```
Chúng ta nên tránh sử dụng những methods nào trong ActiveRecord model instance methods? Có khá nhiều: [QueryMethods](https://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html), [FinderMethods](https://api.rubyonrails.org/classes/ActiveRecord/FinderMethods.html), [Calculations](https://api.rubyonrails.org/classes/ActiveRecord/Calculations.html). Nếu chúng ta sử dụng 1 trong những method này ở trong instance class, chúng sẽ luôn gọi SQL query, ngay cả khi chúng ta đã preloading. Vì thế, hãy luôn cẩn thận, chú ý development log để phát hiện sớm các trường hợp N + 1 này.

### 2.4 `any?`, `exist?` and `present?`
Chúng ta hãy cùng xem qua 1 ví dụ sau:

```
class DocComment < ActiveRecord::Base
  belongs_to :doc_method, counter_cache: true

  # ... things removed for clarity...

  def doc_method?
    doc_method_id.present?
  end
end
```
Tại sao ta dùng `present?` ở đây? Mục đích chỉ là để check xem DocComment object có thuộc 1 doc_method nào không. Nó sẽ chuyển giá trị của `doc_method_id` từ `nil` hoặc 1 Integer thành `true` hoặc `false`.

```
class Object
  def present?
    !blank?
  end
end
```
`blank?` tương đương với câu hỏi: Liệu object này là truthy hay falsey? Mảng và hash rỗng là truthy nhưng nó blank, 1 string rỗng cũng là blank. Trong bí dụ bên trên, `doc_method_id` sẽ chỉ có 2 loại giá trí là `nil` và `Integer`, điều đó có nghĩa là: `present?` ở đây tương đương với `!!`.

```ruby
def doc_method?
  !!doc_method_id
  # same as doc_method_id.present?
end
```

Ví dụ bạn muốn biết nếu 1 ActiveRecord::Relation có records nào hay không, bạn có thể sử dụng `any?/present?/exits?` hoặc dùng: `none?/blank?/emtpy?`. Liệu bạn có chắc là k xảy ra vấn đề gì hay không?

Giả sử chúng ta có đoạn code này:
```
- if @comments.any?
  h2 Comments on this Post
  - @comments.each do |comment|
```
Có bao nhiêu SQL query được execute trong đoạn code trên? => 2. Một câu sẽ được gọi bởi `@comments.any?` (SELECT 1 AS one FROM ... LIMIT 1), và 1 câu `@comments.each` sẽ load toàn bộ relation comments (SELECT "comments".* FROM "comments" WHERE ...).

Vậy nếu ta viết thế này thì sao:

```
- unless @comments.empty?
  h2 Comments on this Post
  - @comments.each do |comment|
```
Nếu viết thế này, sẽ chỉ có 1 câu query được load: `@comment.empty?` sẽ load cả relation bằng câu: `SELECT "comments".* FROM "comments" WHERE ....`. Vậy nếu viết thế này:

```
- if @comments.exists?
  This post has
  = @comments.size
  comments
- if @comments.exists?
  h2 Comments on this Post
  - @comments.each do |comment|
```
Sẽ có 4 queries. `exits?` không có tính năng cache và nó k load relation. `exists?` ở đây sẽ triggers câu: `SELECT 1 ...`, `.size` sẽ triggers câu lệnh `COUNT` vì relation không được cache. và lệnh `exits?` sẽ trigger 1 câu lệnh SELECT khác: `SELECT 1 ...`. Cuối cùng, `@comments` load hết relation để chạy cho vòng each. Trong khi đó, chúng ta hoàn toàn có thể chỉ cần dùng 1 query cho đoạn code này:

```
- if @comments.load.any?
  This post has
  = @comments.size
  comments
- if @comments.any?
  h2 Comments on this Post
  - @comments.each do |comment|
```
Chúng ta hãy check qua 1 số actions, tùy thuộc vào version của Rails:

+ Trong Rails 5.1:

method|SQL generated|memoized?|implementation|Runs query if loaded?|
----|---|----|---|---|
present?|SELECT “users”.* FROM “users”|yes (load)|Object (!blank?)|no|
blank?	|SELECT “users”.* FROM “users”|	yes (load)	|load; blank?|	no|
any?|	SELECT,1 AS one FROM “users” LIMIT 1	|no unless loaded|	!empty?|	no|
empty?|	SELECT,1 AS one FROM “users” LIMIT 1	|no unless loaded	|exists? if !loaded?|	no|
none?|	SELECT,1 AS one FROM “users” LIMIT 1	|no unless loaded	|empty?|	no|
exists?|	SELECT,1 AS one FROM “users” LIMIT 1	|no|	ActiveRecord::Calculations|	yes|

+ Trong Rails 5.0:

method|SQL generated|memoized?|implementation|Runs query if loaded?|
----|---|----|----|-----|
present?|	SELECT “users”.* FROM “users”	|yes (load)	|Object (!blank?)	|no|
blank?|	SELECT “users”.* FROM “users”	|yes (load)	|load; blank?	|no
any?|	SELECT COUNT(*) FROM “users”	|no unless loaded	|!empty?	|no
empty?|	SELECT COUNT(*) FROM “users”	|no unless loaded|	count(:all) > 0|	no
none?|	SELECT COUNT(*) FROM “users”	|no unless loaded	|empty?	|no
exists?|	SELECT,1 AS one FROM “users” LIMIT 1	|no|	ActiveRecord::Calculations|	yes

+ Trong Rails 4.2:

method|SQL generated|memoized?|implementation|Runs query if loaded?|
----|---|----|----|-----|
present?|	SELECT “users”.* FROM “users”	|yes	|Object (!blank?)	|no
blank?|	SELECT “users”.* FROM “users”	|yes|	to_a.blank?|	no
any?|	SELECT COUNT(*) FROM “users”	|no|unless loaded	!empty?|	no
empty? |	SELECT COUNT(*) FROM “users”|	no unless loaded|	count(:all) > 0|	no
none?|	SELECT “users”.* FROM “users”	|yes (load called)|	Array|	no
exists?|	SELECT,1 AS one FROM “users” LIMIT 1|	no	|ActiveRecord::Calculations	|yes

`any?`, `empty?`, `none?` làm chúng ta nhớ tới việc implementation của `size` - Nếu records đã được `loaded?`, ta sẽ sử dụng các method basic của Array, nếu chúng chưa được load, hãy sử dụng SQL query. `exists?` không có caching hay memoization, nó chỉ là `ActiveRecord::Calculatión`, nên sử dụng nó cũng sẽ tùy vào hoàn cảnh, nó sẽ tệ hơn việc dùng `present?` trong 1 vài case.

Có 6 predicate methods, chúng được implement khác nhau, có performace khác nhau, tùy thuộc vào bản Rails chúng ta sử dụng. Nên hay cân nhắc:

+ Không nên sử dung `present?` và `blank?` nếu `ActiveRecord::Relation` không được sử dụng toàn bộ sau khi chúng ta gọi 1 trong 2 methods này. Ví dụ: `@my_relation.present?`, `@my_relation.first(3).each`.
+ `any?`, `none?`, `empty?` nên được thay thế cho `present?`, `blank?` trừ khi bạn chỉ lấy 1 phần của ActiveRecord::Relation sử dụng: `first` hoặc `last`. Chúng sẽ sinh ra 1 câu query SQL check nếu bạn dùng entire relation. Hãy thay: `@users.any?; @users.each` thành `@users.present?; @users.each` hoặc `@users.load.any?; @users.each`, hoặc `@users.any?; @users.first(3).each` is fine.
+ `exists?` giống như `count`, nó không được memorized, và luoon chạy 1 câu SQL query. Không nên sử dụng nó, chúng ta nên thay bằng `present?` hoặc `blank?`

### 2.5 App Checklist
Sau khi đã đọc bài phân tích bên trên, chúng ta hãy cùng tổng hợp lại 1 vài lưu ý:
+ Xem lại những chỗ đang dùng `present?`, `none?`, `any?`, `blank?`, `empty?` cho ActiveRecord::Relations. Khi relations trả về records, bạn muốn in ra thông tin của những records đó sau này, hãy add thêm method `load` khi gọi, vd: `@my_relation.load.any?`
+ Hãy cẩn thận khi dùng `exist?`, nó luôn luôn execute SQL query. Chỉ dùng khi thích hợp (vd chỉ muốn check có hay k, và không sử dụng kết quả của query). thay vào đó, hãy cân nhắc dùng `present?` hoặc các methods có dùng `empty?`
+ Cẩn thận với việc sử dụng `where` trong instance methods đối với ActiveRecord objects. Chúng có thể break preloading và thường dẫn đến N+1s khi render collections.
+ `count` luôn execute SQL query, hãy cân nhắc sử dụng `size` để thay thế.

## Kết luận
Khi Rails app được mở rộng và phức tạp hơn hoặc số lượng người dùng tăng lên, các câu SQL không cần thiết sẽ trở thành "gánh nặng" khi render/ load 1 trang, ảnh hưởng trực tiếp đên app performance. Hãy để ý tới từng chi tiết nhỏ nhất để cải thiện kĩ năng lập trình của mình.

Hy vọng bài viết sẽ hữu ích cho bạn khi bạn refactor/ code Rails app của mình.

### Reference: [3 ActiveRecord Mistakes That Slow Down Rails Apps: Count, Where and Present](https://www.speedshop.co/2019/01/10/three-activerecord-mistakes.html)

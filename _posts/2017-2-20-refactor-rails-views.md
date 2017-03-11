---
layout: post
comments: true
title: 5 ways to refactor Rails views
tags:
- Become a Rubyist
---


# Mở đầu
Khi bạn phát triển một ứng dụng web, việc code nở ra rất nhiều theo thời gian là điều khó tránh khỏi. Thêm vào đó, khi mở rộng code, spec thay đổi hoặc trong giai đoạn fix bug, bạn sẽ gặp rất nhiều khó khăn nếu không quản lý tốt những dòng code của mình từ đầu. Refactor rails views code cũng là một kĩ năng quan trọng và cần thiết cho bạn. Trong bài viết này, mình sẽ đưa ra một vài cách để bạn có thể refactor lại views code của mình.

# Cách để refactor views code
## 1. Partials
Đây là một phương pháp cơ bản và phổ biến nhất mà chúng ta có thể nghĩ tới. Partials được xây dựng trong Rails để có thể tái sử dụng những đoạn code view hoặc chia nhỏ từng phần của file view tổng thể, khiến cho chúng ta dễ theo dõi được cấu trúc của 1 file view.
Một partial thông thường sẽ được gọi thế này
```
# app/views/employees/new.html.erb
<h1>New Employee</h1>
<%= render 'form' %>
<%= link_to 'Back', employees_path %>
```
Nó sẽ render ra file `_form.html.erb` trong cùng thư mục với file `new.html.erb`. Trong trang index employee, bạn có thể render ra 1 list các @employees bằng cách
```
<%= render @employees %>
```
Rails sẽ tìm kiếm partial tên `_employee` và sử dụng nó để render mỗi employee trong list @employees collection.
Ngoài ra nhiều lúc, bạn cũng sẽ cần đến `render partial: "shared/employee', collection: @employees` để render ra một file partial nằm hoàn toàn ở 1 thư mục khác :D

### Resources
* [Layouts and Rendering in Rails (Official Docs)](http://guides.rubyonrails.org/layouts_and_rendering.html)
* [Action View Partials (Official Docs)](http://api.rubyonrails.org/classes/ActionView/PartialRenderer.html)
* [Partials in Ruby on Rails](https://richonrails.com/articles/partials-in-ruby-on-rails)

## 2. Decorators
Decorator pattern được sử dụng để đóng gói các logic hiển thị mà không thuộc về model, nhưng cũng không thuộc về views :D
Ví dụ như bạn có một blogging application, hiển thị ngày public của một article theo một format (vd Feb 28th, 2017) hoặc là "Draft" nếu như nó chưa được publish. Khi đó, đoạn code của bạn sẽ là:
```
<article>
  <span class="publication-status">
    <% if @article.published? %>
      Published at: <%= @article.published_at.strfitme("%B #{@article.published_at.day.ordinalize}, %Y")
    <% else %>
      Draft
    <% end %>
  </span>
...
</article>
```
Nếu bạn để ý, khi có những logic hiển thị nằm trong view file, chúng ta sẽ rất khó để nhìn ra cấu trúc tổng thể của toàn bộ file view.
Bạn có thể nghĩ đến việc dùng View Helpers của Rails như một giải pháp thay thế việc đặt logic ở trong view. Tuy nhiên, sử dụng Helper có một vài hạn chế:
* Helpers được include trong tất cả các views, tức là bạn sẽ phải cẩn trọng trong việc đặt tên hàm vì nó sẽ rất dễ bị conflict. Thêm vào đó, 1 view có thể gọi những method mà không có tác dụng gì cho nó thì cũng không hợp lý. :D
* Helpers là các modules, do đó bạn sẽ không thể truy cập từ 1 object. Điều đó có nghĩa là bạn sẽ phải truyền 1 object vào phương thức giống như thế này: `article_published_at_date(article)`

Draper là một gem rất phổ biến cung cấp cho banj định nghĩa các decorator pattern để mở rộng một Active Record object với logic hiển thị mà không cần đặt hết chúng vào trong model.
```
class ArticleDecorator < Draper::Decorator
  delegates_all

  def publication_status
    if is_published?
      "Published at: #{published_at}"
    else
      "Draft"
    end
  end

  def published_at
    object.published_at.strfitme("%B #{published_day}, %Y")
  end

  private

  def published_day
    object.published_at.day.ordinalize
  end
end
```
Để object của chúng ta gọi được decorator,  ta chỉ việc thêm nó vào trong controller như sau:
```
class ArticlesController < ApplicationController
  def show
    @article.find(params[:id]).decorate
  end
end
```
Khi đó, trong view file sẽ chỉ còn lại là:
```
<article>
  <span class="publication-status">
    <%= @article.publication_status %>
  </span>
...
</article>
```
Sẽ không còn những logic không cần thiết trong view. Chúng ta có thể dễ dàng nhìn, đọc và hiểu được điều gì đang xảy ra trong view và cấu trúc tổng thể của file view là như thế nào :D

### Resources
* [Draper Gem](https://github.com/drapergem/draper)
* [Experimenting with Draper](http://tutorials.jumpstartlab.com/topics/decorators.html)
* [Decorators on Rails](http://johnotander.com/rails/2014/03/07/decorators-on-rails/)
* [Rails Presenters](http://nithinbekal.com/posts/rails-presenters/)

## 3. Null Objects
Sử dụng Null Object pattern để tranh việc gặp phải các logic phân nhánh có điều kiện.
Giả sử chúng ta có một ứng dụng mà người dùng có thể là admins, customers hoặc guest nếu họ không đang nhập. Tùy thuộc vào role của từng loại user, chúng ta sẽ phải show ra từng loại header navigation khác nhau.
```
<% if user_signed_in? %>
  <% if current_user.role == 'admin' %>
    <%= render 'admin_nav' %>
  <% elsif current_user.role == 'customer' %>
    <%= render 'customer_nav' %>
  <% end %>
<% else %>
  <%= render 'guest_nav' %>
<% end %>
```
Chúng ta đang sử dụng partials, trông code có vẻ khá tốt. Tuy nhiên, đoạn code ở đây chưa được linh hoạt cho lắm. Mỗi khi có 1 role mới được thêm vào, chúng ta lại sẽ phải tìm những chỗ nào tương ứng vs từng loại user mà sửa lại code, thêm 1 đoạn elsif nữa vào. Nếu phải làm như thế, chúng ta sẽ phải sửa ở rất nhiều nơi, mà k có gì đảm bảo là ta đã sửa hết cả :v
Sử dụng một chút refactoring, chúng ta có thể linh động trong việc render partial dựa trên user's role như sau:
```
<% if user_signed_in? %>
  <%= render "#{current_user.role}_nav" %>
<% else %>
  <%= render 'guest_nav' %>
<% end %>
```
Great! Tuy nhiên chúng ta vẫn bỏ sót trường hợp null, đó là khi current_user bị nil. Khi đó current_user sẽ trả về nil -> Không render ra được partial tương ứng. Đó là lúc Null Object tỏ ra hữu ích.
Null Object là một pattern rất đơn giản, nhưng nó mang tới rất nhiều ứng dụng. Một Null Object là một  Plain old Ruby object mà chứa những method giống như non-null object, chỉ khác ở giá trị trả về. Kết hợp với Ruby's duck typing, chúng ta sẽ có thể coi Null Object giống như những normal objects đang được gọi.
```
class NullUser
  def role
    'guest'
  end
end
```
Bây giờ, trong ApplicationCOntroller, nơi current_user được định nghĩa, chúng ta sẽ trả về NullUser thay cho việc tra về nil nếu người dùng chưa đăng nhập.
```
class ApplicationController
  def current_user
    super || NullUser.new
  end
end
```
Điều đó có nghĩa là trong views code  của chúng ta giờ chỉ còn lại 1 đoạn rất ngắn gọn:
```
<%= render "#{current_user.role}_nav" %>
```

### Resources
* [Rails refactoring Example: Introduce Null Object](https://robots.thoughtbot.com/rails-refactoring-example-introduce-null-object)
* [Naught Gem](https://github.com/avdi/naught/)

## 4. Form Objects
Form objects là một công cụ hữu ích khi bạn có một multi-modle forms hoặc các form logic phức tạp. Thay vì việc sử dụng `accepts_nested_attributes_for` và handing với việc validations ở nhiều model khác nhau, form object cho phép bạn nhóm các thuộc tính của form và validation vào 1 object duy nhất.

Ví dụ bạn có một model `Survey`. Nếu bạn muốn tạo một survey, bạn cần tạo rất nhiều `Question`, bạn nên xem xét có nên sử dụng Form Object để tập trung xử lý logic của việc tạo Survey vào 1 chỗ không. ( Đôi khi phần xử lý logic này diễn ra trong controller, nên sử dụng Form object cũng là 1 cách hữu ích để banj có thể làm thin controller của mình :D )

Để dễ hiểu hơn, mình sẽ sử dụng gem Virtus để tạo ra 1 object mới (trông nó sẽ khá giống với 1 ActiveRecord model ) nhưng bên trong nó sẽ bao gồm các options cần thiết để tạo ra Form Objects
```
class CreateSurvey
  include Virtus

  extend ActiveModel::Naming
  include ActiveModel::Conversion
  include ActiveModel::Validations

  attribute :title, String
  attribute :questions, Array[String]

  validates :title, presence: true

  def save
    if valid?
      persist!
      true
    else
      false
    end
  end

  private

  def persist!
    transaction do
      @survey = Survey.create!(title: title)
      @questions = questions.map{|question_text| Question.create(text: question_text)
    end
  end
end
```
Trong controller, chúng ta sẽ tạo 1 form object với params được gửi lên từ form trong view, sử dụng CreateSurvey object ở trên.
```Ruby
class SurveysController < ApplicationController
  def create
    @survey = CreateSurvey.new(params[:survey])

    if @survey.save
      # logic if successful
    else
      # logic if unsuccessful
    end
  end
end
```

### Resources
* [Reform Gem](https://github.com/apotonick/reform)
* [Form Object Validations in Rails 4](http://crypt.codemancers.com/posts/2013-12-18-form-objects-validations/)
* [Form objects with Virtus](http://hawkins.io/2014/01/form_objects_with_virtus/)
* [Form object (Rails Cast)](http://railscasts.com/episodes/416-form-objects)
* [7 Patterns to refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)

## 5. Alternate Templating Languages
Erb là một template language tốt, đủ dùng và rất phổ biến. Tuy nhiên, một vài ngôn ngữ *nâng cao*  khác như Haml và Slim sẽ giúp cho bạn tối giản hết mức những đoạn code HTML của mình, khiến cho những dòng code trở nên ngắn ngọn hơn rất nhiều.
**ERB**
```
<section class=”container”>
   <h1><%= post.title %></h1>
   <h2><%= post.subtitle %></h2>
   <div class=”content”>
     <%= post.content %>
   </div>
</section>
```
**Haml**
```
%section.container
   %h1= post.title
   %h2= post.subtitle
   .content
     = post.content
```
**Slim**
```
section.container
   h1= post.title
   h2= post.subtitle
      .content= post.content
 ```

Chọn một template language thích hợp là một việc mang tính cá nhân. Nếu bạn cảm tháy thoải mái với HTML tags, ERB sẽ là một lựa chọn tốt và hỗ trợ rất nhiều cho bạn. Nếu bạn muốn ngắn gọn, nhìn code sạch, dễ nhìn hơn, hãy thử sử dụng Haml hoặc Slim và trải nghiệm :D

### Resource
* [Haml](http://haml.info/)
* [Slim](http://slim-lang.com/)
# Kết luận
Nếu bạn đọc tới đây thì chắc bạn cũng nhận ra rằng:
> Refactor code view KHÔNG làm cho code tổng thể của project giảm đi, nó chỉ bê code từ chỗ này cho sang chỗ khác =]]

Tuy nhiên, mọi thứ sẽ trở về đúng vị trí của nó, khiến bạn sẽ dễ dàng hơn rất nhiều trong việc mở rộng và quản lý code view của mình.
Hy vọng bài viết này có ích cho bạn! :)

Nguồn: https://allenan.com/refactoring-rails-views/:

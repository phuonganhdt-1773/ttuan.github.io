---
layout: post
title: Create your own generator
tags:
- Ruby
---

## 1. Tản mạn
Nếu bạn đã từng code Rails, dù master hay mới chỉ beginer, chắc hẳn bạn đã có lần dùng qua lệnh `rails generator model $model_name`, hoặc `rails g controller $controller_name`. Generators là 1 tool của Rails, giúp chúng ta có thể sinh code theo các templates có sẵn một cách dễ dàng, tiện lợi. Bạn có thể đọc về code của tính năng generator của Rails trên github: [base-generator](https://github.com/rails/rails/blob/b2eb1d1c55/railties/lib/rails/generators/base.rb).

Mọi chuyện sẽ không có gì đáng nói nếu dự án không "to" dần lên, khiến chúng ta nghĩ tới chuyện áp dụng các design patterns vào Rails project: Service Object, Value Object, Form Object, View Object, Null Object, ... Đi kèm với đó là các files sinh ra khi áp dụng theo 1 patterns (ít nhất là 1 file trong thư mục `app/$pattern_name` và 1 file trong thư mục spec). Việc tạo file và gõ đi gõ lại khá nhàm chán, nên ban đầu mình dùng cơm - sử dụng `text snippet`. Hầu hết các editor thông dụng đều hỗ trợ custom snippet, như với Vim thì mình có tạo snippet ở [đây](https://github.com/ttuan/my-vim-snippets/blob/master/snippets/ruby.snippets#L5). Tuy nhiên, cách này khá thủ công, và chỉ dùng được cho 1 file, vẫn phải gõ khá nhiều =))

Và Rails Generator là 1 giải pháp khác, xịn xò hơn. Trong bài viết này mình sẽ tự tạo 1 generator cho service object. (Các loại object khác cũng có thể làm tương tự ;)

## 2. Triển
Trong trường hợp bạn chưa dùng service object :yaoming: thì Service Object là 1 ruby class, có nhiệm vụ thực thi 1 action nào đó. Nó được sinh ra nhằm giảm tải được logic cho controller hoặc model. Khi tách phần logic đó ra bên ngoài, chúng ta sẽ dễ đọc code hơn, dễ tái sử dụng hơn, dễ test hơn và dễ mở rộng hơn, code sẽ trông DRY hơn ;)

Example:

```ruby
class TestService
  def initialize
  end

  def execute
  end

  private

  def method1
  end

  # .
  # .
  # .

  def methodN
  end
end
```

Có 2 cách để tạo ra 1 rails generator: làm bằng tay hoặc dùng luôn generator =)) Tức là trong Rails đã hỗ trợ sẵn cách để generate ra 1 generator =))

Để tạo ra 1 generator, bạn chỉ cần mở terminal và chạy lệnh:

```bash
$ bin/rails generate generator service
      create lib/generators/service
      create lib/generators/service/service_generator.rb
      create lib/generators/service/USAGE
      create lib/generators/service/templates
      invoke test_unit
      create test/lib/generators/service_generator_test.rb
```
Command trên sẽ sinh ra thư mục generator trong thư mục `lib` của Rails app. Nếu bạn muốn tách code của Rails generator ra chỗ khác, k để trong lib thì hãy custom + thêm extra autoload paths vào application.rb.

Cùng xem qua file được rails generator tạo ra:

```ruby
class ServiceGenerator < Rails::Generators::NamedBase
     source_root File.expand_path('templates', __dir__)
end

```
Trong file này, generator của chúng ta đang thừa kế từ `Rails::Generators::NamedBase`, tức là nó cần ít nhất 1 argument vào câu lệnh tạo. Bạn có thể đọc thêm nhiều options của Rails generator tại [rails-guide](https://guides.rubyonrails.org/generators.html).

Mục tiêu của chúng ta là sẽ tạo ra 1 generator, mà khi ta gọi lệnh sinh ra files, nó sẽ nhận vào 1 list các tham số: tên service, tên các method, namespace của class, ... Câu lệnh này sẽ sử dụng template được chúng ta quy định sẵn.

#### 1. Create service generator
Sau khi đã sinh ra thư mục `lib/generators/service` bằng câu lệnh bên trên, chúng ta sẽ sửa code, phần lớn code sẽ nằm ở file `service_generator.rb`

```ruby
class ServiceGenerator < Rails::Generators::NamedBase
	source_root File.expand_path('../templates', __FILE__)

	argument :methods, type: :array, default: [], banner: "method method"
	class_option :module, type: :string

	def create_service_file
		@module_name = options[:module]

		service_dir_path = "app/services"
		generator_dir_path = service_dir_path + ("/#{@module_name.underscore}" if @module_name.present?).to_s
		generator_path = generator_dir_path + "/#{file_name}.rb"

		Dir.mkdir(service_dir_path) unless File.exist?(service_dir_path)
		Dir.mkdir(generator_dir_path) unless File.exist?(generator_dir_path)

		template "service.erb", generator_path
	end

end
```
Dòng `	source_root File.expand_path('../templates', __FILE__)` sẽ trỏ tới thư mục templates, nơi chúng ta tạo các template mặc định cho service.

```ruby
argument :methods, type: :array, default: [], banner: "method method"
class_option :module, type: :string
```
Đoạn code trên định nghĩa các tham số sẽ truyền vào. Method `argument` sẽ parse các tham số trên command line thành các themdos, đưa chúng thành `attr_accessor`, còn `class_option` sẽ parse command line options và lưu chúng vào trong biến `options`.

Phần code bên dưới khá đơn giản, chúng ta sẽ phải tạo 1 method `create_service_file`, trong đó sẽ chứa toàn bộ logic. Do có sử dụng method `class_option` bên trên, nên option `module` của chúng ta đã được lưu vào biến `options`, do đó, ta có thể gọi ra module_name từ `options[:module].

Bên dưới, chúng ta chỉ định nơi sẽ lưu file service mới. Thông thường là trong thư mục `app/services`, nếu có module_name thì sẽ là `app/services/module_name/`. Giá trị `file_name` sẽ được định nghĩa trong `NameBase` mà ta kế thừa. Và cuối cùng, chúng ta dùng method `template` để lấy template cần dùng. Nếu bạn muốn generate nhiều file hơn, làm tương tự: định nghĩa file_path, sau đó gọi method `template` để gen ra file.

#### 2. Create template file
```ruby
<%# service.erb %>

<% if @module_name.present? %> module <%= @module_name.camelize %>  <% end %>
class <%= class_name %>

def initialize
end

def call
end

<% if methods.present? %> private <% end %>
<% for method in methods %>
  def <%= method %>
  end
<% end %>
end
<% if @module_name.present? %> end <% end %>
```
Template của chúng ta viết khá đơn giản, dùng code nhúng erb để viết.

#### 3. Fill out the USAGE file
Cuối cùng, chúng ta sẽ điền vào file USAGE để cho mọi người dễ dùng hơn, vì file USAGE này như 1 tờ hướng dẫn sử dụng vậy :v  Thông tin sẽ được show ra khi người dùng sử dụng options `--help` hoặc `-h` kèm với command.

![](https://arsfutura-production.s3.amazonaws.com/magazine/2019/02/diy_rails_generator/description.png)

Nếu bạn gõ lệnh `rails g service -h`, kết quả sẽ được show ra:

![](https://arsfutura-production.s3.amazonaws.com/magazine/2019/02/diy_rails_generator/description-output.png)

#### 4. Use
Bạn có thể sử dụng các câu lệnh dưới đây để sinh ra file service:

* `rails g service test_service`

	![](https://arsfutura-production.s3.amazonaws.com/magazine/2019/02/diy_rails_generator/basic-service.png)
	```ruby
	class TestService
	  def initialize
	  end

	  def call
	  end
	end
	```

* `rails g service test_service test_method1 test_method2`

	```ruby
	class TestService
	  def initialize
	  end

	  def call
	  end

	  private

	  def test_method1
	  end

	  def test_method2
	  end
	end
	```
* `rails g service test_service --module test_module`

	![](https://arsfutura-production.s3.amazonaws.com/magazine/2019/02/diy_rails_generator/service-with-module.png)

	```ruby
	module TestModule
	  class TestService
	    def initialize
	    end

	    def call
	    end
	  end
	end

	```

Nếu bạn không muốn sinh thêm những đoạn code generator ở trong thư mục code của dự án, bạn cũng có thể sử dụng [gem](https://github.com/zenjara/matas_service_generator). Gem này được build từ những đoạn code chúng ta vừa xem ở bên trên. ;)

## 3. Kết
Chỉ bằng vài đoạn code đơn giản, bạn đã có thể tự tạo cho mình 1 generator mới. Có thể nó không giúp bạn tiết kiệm nhiều thời gian, nhưng chí ít, nó cũng giúp bạn hiểu được phần nào về Rails generator.

Hy vọng sau bài viết này, bạn có thể tự tạo thêm các generator khác để áp dụng cho các design pattern còn lại. ;)

Bài viết được tham khảo từ: [create your own rails generator](https://arsfutura.co/magazine/diy-create-your-own-rails-generator/).































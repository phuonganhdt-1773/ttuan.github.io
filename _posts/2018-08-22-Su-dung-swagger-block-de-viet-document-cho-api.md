---
layout: post
title: Sử dụng gem swagger-block để tạo document cho Rails API app
tags:
- Rails
---

![swagger](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSXgM-tYqDndnMDn5GCkgu8kSQjlAaLY6rwlObqN4gqoMW9rVAJ)

# I. Tản mạn
Gần đây mình có làm 1 dự án API, và như bao dự án API khác, việc đầu tiên mình nghĩ tới sau khi setup xong base code là Document. Document chính là phương tiện chính để bên mobile có thể nhìn vào đó mà sử dụng API của mình. 

Mình có google một số gem dùng để viết API cho Rails app. Gem được rate nhiều stars trên github nhất là [apipie-rails](https://github.com/Apipie/apipie-rails) với 1850 stars :v Trong dự án trước mình đã sử dụng thằng này rồi, nhưng gặp khá nhiều vấn đề với nó (và mình cũng không hiểu sao nó lại được nhiều stars vậy :v). Đó là: 

* Cú pháp viết dài dòng, không mạch lạc. Chủ yếu nội dung hiển thị ra là nằm trong phần `description`, phải viết bằng tay.
* Khó để tái sử dụng lại code.
* Khó để mở rộng: Khi có thay đổi, ví dụ khi ta muốn thêm 1 trường vào bảng user, ta sẽ phải chỉnh sửa hết các document có chứa thông tin user. 

Do đã làm việc trước với `apipie-rails` rồi nên mình định sẽ đi tìm 1 giải pháp khác, thì có được gợi ý sử dụng [Swagger UI](https://swagger.io/tools/swagger-ui/). Đây là 1 nền tảng cho phép các team viết API có thể viết document cho api, bất kể là ngôn ngữ gì, chỉ cần bạn cung cấp cho nó 1 file `.json` là ok. Google thử khi áp dụng vào Rails project thì có 2 gems được đánh giá cao là: [swagger-docs](https://github.com/richhollis/swagger-docs) và gem [swagger-block](https://github.com/fotinakis/swagger-blocks). Tuy nhiên, trong bài viết này mình sử dụng swagger-block vì nó support cho v2.0 của swagger specification.

# II. Triển
Điều đầu tiên bạn cần phải chú ý đó là gem `swagger-block` nó không tự tạo doc cho bạn, nó chỉ có tác dụng tạo ra 1 file `.json` phù hợp với format của `swagger-ui`. Tức là chúng ta sẽ phải tạo ra 1 file json, sau đó dùng nó là đầu vào cho `swagger-ui` -> trang document.

## 1. Setup API server

Trước hết, chúng ta cần tạo 1 project mới: 

```bash
$ rails new api_sample_app --api
$ cd api_sample_app
$ bin/rails g scaffold User name:string email:string
$ bin/rails db:migrate
```

Sau đó hãy thêm config cho CORS: (flow this [link](https://github.com/cyu/rack-cors#rails-configuration))

```ruby
# config/application.rb
config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'
    resource '*', :headers => :any, :methods => [:get, :post, :options]
  end
end
```

```ruby
# Gemfile
gem "swagger-blocks"
```

## 2. Setup Swagger Block

Để cho dễ theo dõi, mình sẽ đi theo các bước như trong [hướng dẫn](https://github.com/fotinakis/swagger-blocks#petscontroller) của gem swagger-block, sau đó sẽ đi dần tới các đoạn refactor.

### 2.1 Request in Swagger-Block

Code mỗi doc sẽ đi liền với 1 action/ controller.

```
class UsersController < ApplicationController
  include Swagger::Blocks

  swagger_path '/users/{id}' do

    # GET /users/:id
    operation :get do
      key :description, 'Find a user by ID'
      key :operationId, :find_user_by_id

      parameter name: :id do
        key :in, :path
        key :description, 'User ID'
        key :required, true
        key :type, :integer
        key :format, :int64 
      end

      response 200 do
        key :description, 'User'
        schema do
          key :required, [:id, :name]
          property :id do
            key :type, :integer
            key :format, :int64
          end
          property :name do
            key :type, :string
          end
        end
      end
    end
  end

  def  show 
    # ... 
  end

  # ...
end
```

Swagger Block cung cấp 1 method là: `Swagger::Blocks.build_root_json` có nhiệm vụ nhận vào các class chưa thông tin về API. ví dụ như:

```
Swagger::Blocks.build_root_json([User, UserController])
```

Do đó, chúng ta có thể config thành:

```ruby
# routes.rb

namespace :api, format: "json" do
	namespace :v1 do
		get  'api-docs' ,  to: 'api_docs#index'  unless Rails.env.production?
	end
end
```

```ruby
# app/controllers/api_docs_controller.rb
class ApiDocsController < ApplicationController
  include Swagger::Blocks

  swagger_root  do 
    # define meta information such as API name and version 
  end

  def index
    render json: Swagger::Blocks.build_root_json([User, UsersController])
  end
end
```

Tuy nhiên, tới đây chắc chắn bạn sẽ nhìn thấy 1 vấn đề. Đó là controller sẽ phình to rất nhanh nếu như đặt doc ở đó. Thêm nữa, nhiệm vụ của controller cũng không phải là nơi xử lý doc này. Cách tốt nhất là cứ nên tách nó ra thành các module đúng không. 

Trước hết, bạn có thể tạo 1 thư mục `app/controllers/concerns/swagger/` để lưu trữ các file liên quan tới `request` của swagger.

```ruby
# app/controllers/concerns/swagger/users_api.rb

module  Swagger :: UsersApi 
  extend  ActiveSupport :: Concern 
  include  Swagger :: Blocks

  included do
    swagger_path '/users/{id}' do
      parameter :user_id do
        key :name, :id
      end

      operation :get do
        key :description, 'Finds the specified user'
        key :operationId, :find_user_by_id

        response 200 do
          key :description, 'User specified by its ID'
          schema do
            key :'$ref', :UserOutput
          end
        end
      end
      # ...

    end 
  end 
end
```

Sau đó `include` nó vào trong controller:

```ruby
# app/controllers/users_controller.rb

class UsersController < ApplicationControllers 
  include  Swagger::UsersApi

  # ...
end
```

### 2.2 Response and Parameters 

Response và parameters sẽ rất hay bị trùng trong các API. Ví dụ: API login xong, bạn sẽ phải trả về thông tin `User`. Trong API show/edit thông tin user, bạn cũng sẽ phải trả về thông tin `User`, ... Chúng ta sẽ phải tìm cách để sao cho có thể tái sử dụng code.

Trong SwaggerUI, chúng ta có thể sử dụng `$ref` để gọi tới 1 schema đã được định nghĩa sẵn. Hình dung nó như 1 cái link để trỏ tới 1 Schema JSON khác. 

```ruby
# app/models/concerns/swagger/user_schema.rb

module Swagger::UserSchema
  extend ActiveSupport::Concern
  include Swagger::Blocks

  included do
    swagger_schema :User do
      key :required, [:name, :email]
      property :name do
        key :type, :string
      end
      property :email do
        key :type, :string
      end
    end
  end
end
```

Ở đây mình đã định nghĩa ra 1 `user_schema`, bao gồm 2 key là `name` và `email`. Schema `User` này sẽ được dùng ở những API yêu cầu trả về thông tin của User.

Sau đó, bạn cần `include` nó vào trong model `user.rb`

```ruby
# app/models/user.rb

class User < ApplicationRecord 
  include Swagger::UserSchema 
end
```

1 schema cũng rất hay cần dùng chung đó là `ErrorSchema`, sẽ được dùng khi API trả về thông tin lỗi. (Do không authenticate được user/ hoặc không tạo được record.)

```ruby
# app/models/concerns/swagger/error_schema.rb

module Swagger::ErrorSchema
  extend ActiveSupport::Concern
  include Swagger::Blocks

  included do
    swagger_schema :ErrorOutput do
      key :required, [:errors]
      property :errors do
        key :type, :array
        items do
          key :type, :string
        end
      end
    end
  end
end
```

Sau khi đã có các schema này rồi, chúng ta có thể tạo 1 file `common_response` với nội dung:

```ruby
module Swagger::ErrorResponses
  module NotFoundError
    def self.extended(base)
      base.response 404 do
        key :description, 'Resource not found'
        schema do
          key :'$ref', :ErrorOutput 
        end
      end
    end
  end

  # ...
end
```

Tới đây là xong phần response. Chúng ta sẽ chuyển qua phần tổ chức code cho parameter.

Sẽ có nhiều API có chung parameter. Ví dụ như phần lớn các API đều cần sử dụng 1 param `token` đểu authenticate người dùng. Hoặc API update user và API lấy thông tin user sẽ có chung param `user_id`. Chúng ta cần phải định nghĩa riêng 1 chỗ cho các parameter này.

```ruby
# app/controllers/concerns/swagger/parameters.rb

module Swagger::Parameters
  def self.extended(base)
    base.parameter :user_id do
      key :in, :path
      key :description, 'User ID'
      key :required, true
      key :type, :integer
      key :format, :int64
    end
  end
end
```

sau đó `extend` file này vào trong swagger root:

```
# app/controllers/concerns/swagger/sample_app_root.rb

module Swagger::SampleAppRoot
  extend ActiveSupport::Concern
  include Swagger::Blocks

  included do
    swagger_root do
      # ...

      extend Swagger::Parameters
    end
  end
end
```

Tới đây, phần lớn những gì có thể tách ra để dùng chung của chúng ta đã xong. Việc còn lại chỉ là chỉnh sửa lại file `users_api.rb` sao cho sạch đẹp thôi :v 

```ruby
# app/controllers/concerns/swagger/users_api.rb

module  Swagger :: UsersApi 
  extend  ActiveSupport :: Concern 
  include  Swagger :: Blocks

  included do
    include Swagger::ErrorSchema

    swagger_path  '/ users / {id}'  do 
      operation :get do
        key :description, 'Returns the specified user'
        key :operationId, :find_user_by_id
        
        parameters  : user_id  # Include parameters defined by root

        response 200 do
          key :description, 'User specified by its ID'
          schema do
            key :'$ref', :User
          end
        end

        # Captures the error response which is separately defined 
        Extend  Swagger :: ErrorResponses :: NotFoundError 
      End 
    End 
  End 
End
```

Vậy là mọi việc đã xong, cấu trúc của swagger của chúng ta sẽ có dạng:

```
app/controllers
├── api_docs_controller.rb
├── application_controller.rb
├── concerns
│   └── swagger
│       ├── api_docs.rb
│       ├── error_responses.rb
│       ├── parameters.rb
│       └── users_api.rb
└── users_controller.rb

app/models
├── application_record.rb
├── concerns
│   └── swagger
│       ├── error_schema.rb
│       └── user_schema.rb
└── user.rb
```

Bây giờ thưởng thức thành quả thôi:

```
$ bin/rails s
$ curl http://localhost:3000/api-docs.json | jq .
```

Response trả về sẽ là 1 thông tin dạng json, như ở [đây](https://gist.github.com/ttuan/2a5b456ec526501fd90e6b1bf0a0f310).

## 3. Setup Swagger UI

Như vậy là chúng ta đã có 1 file json chứa đầy đủ thông tin rồi. Nhiệm vụ bây giờ chỉ còn là làm thế nào để show nó ra cho bên mobile có thể xem một cách dễ dàng được. 

Google 1 hồi thì mình có thấy gem [swagger_ui_engine](https://github.com/zuzannast/swagger_ui_engine). Gem này nhận vào 1 thông tin json (có thể là file, cũng có thể là ở dạng raw response)

Cách config cũng khá đơn giản: 

```
gem 'swagger_ui_engine'
```

Sau khi `bundle install` thì bạn mount nó trong file `config/routes.rb`

```
mount SwaggerUiEngine::Engine, at: "/api_docs"
```

Bạn có thể tham khảo trong README của gem này, ở đây mình chỉ config tới đường dẫn sẽ render ra thông tin dạng json như sau:

```ruby
# config/initialize/swagger_ui_engine.rb

if Rails.env.development? || Rails.env.staging?
  SwaggerUiEngine.configure do |config|
    config.swagger_url = {
      v1: "/api/v1/api_doc"
    }

    config.validator_enabled = true
    config.json_editor = true
    config.request_headers = true
  end
end
```

Done. Tới đây bạn có thể restart lại server, truy cập vào đường link: `/api_docs`, bạn sẽ thấy được thành quả của mình:

![sample app](https://camo.qiitausercontent.com/a519fed26c22936e5200faf39978f6bc8d9a8164/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f36323039372f62616434353363622d376131352d396137302d653131652d3135643234373637633434612e706e67)


# Kết luận
`swagger-block` là một công cụ khá hay để bạn có thể dễ dàng tạo ra 1 trang doc ok trong một khoảng thời gian ngắn. Riêng cá nhân mình đánh giá:

#### Ưu điểm

* Dễ tái sử dụng code, các code được tách ra thành các phần riêng biệt.
* Giao diện khá bắt mắt, có thể thấy luôn được input, output
* Bên phía client có thể gọi request ở ngay trên trang doc của mình, không cần dùng qua 1 restclient nào cả(vd Postman/ curl, ...)

#### Nhược điểm
* Debug khá khó. Mỗi lần debug sẽ khá khó để dò ra lỗi, log cũng không có thông tin gì nhiều -> phải dựa vào kinh nghiệm debug của từng người :v 

Hy vọng qua bài viết này, các bạn có thể tạo một trang doc đẹp mắt khi làm dự án API. Good luck ;)
























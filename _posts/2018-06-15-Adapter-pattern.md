---
layout: post
title: Rails Design Pattern - Adapter Pattern
tags:
- Rails
- design-pattern
---

![](https://i1.wp.com/www.sihui.io/wp-content/uploads/2017/09/img_59cb2a2d0738c.png?w=1265)

## Mở đầu
Nếu bạn là một web developer với khoảng 2 năm kinh nghiệm, chắc hẳn các bạn đã không ít lần đọc qua về các Design patterns hay cách áp dụng chúng để làm cho code trở nên hướng đối tượng hơn, dễ đọc, dễ hiểu, dễ maintain, dễ mở rộng, ... Các design patterns được áp dụng khá nhiều trong các Rails projects, mà phổ biến nhất là Service Object, Decorators, Form Object, Query Objects, Policies, Null Objects, ... Những design patterns này có lẽ đã khá quen thuộc với các bạn nên hôm nay mình sẽ giới thiệu 1 design pattern khác (không mới nhưng vẫn rất hiệu quả) đó là **Adapter Pattern**

## Adapter Pattern

> Adapters là các objects được sinh ra với mục đích đóng gói các công việc gọi API của bên thứ 3.

Adapters có thể coi là 1 abstraction layer bên trên mỗi khi chúng ta gọi API ra bên ngoài. Tất cả các công việc như tạo input, call api, xử lý dữ liệu trả về, ... sẽ được nhóm chung vào 1 chỗ để dễ quản lý. Đồng thời nếu trong tương lai, chúng ta muốn thay đổi gem đã dùng để gọi API, chúng ta có thể dễ dàng chuyển đổi hơn, chỉ cần tìm và thay thế 1 chỗ chứ k phải sửa trong toàn project.

**Note:** Adapter objects được đặt ở `/app/adapters/folder`

#### Problem
Chúng ta có 1 bài toán đơn giản như sau: Gọi API của Instagram với 1 access token có sẵn, sau đó lấy về feed của tài khoản đó.

Khá đơn giản phải không =))

### 1. Bad Solution
Bạn có thể làm việc này rất đơn giản, chỉ cần viết API call trong controller hoặc trong Service Object: Tạo Instagram client trong controller bằng token có sẵn, sau đó gọi API và lấy kết quả trả về. Ez =))


```ruby
class PhotosController < ApplicationController
  Rails.application.secrets.instagram_access_token

  def index
    instagram_client = Instagram.client(access_token: INSTAGRAM_ACCESS_TOKEN)
    feed = instagram_client.user_recent_media

    render json: feed
  end
end
```

Có 1 nhược điểm mà bạn có thể nhận thấy, đó là những đoạn code của chúng ta có thể bị trùng lặp, và nằm tản mác ở nhiều nơi. Nếu sau này muốn thay thế gem khác (hoặc có thể code của gem thay đổi) thì sẽ gặp khó khăn trong việc search in project && sửa lại code.

### 2. Better Solution

Chúng ta sẽ tách phần gọi API của Instagram ra 1 chỗ, InstagramAdapter. Các dữ liệu cần thiết để khởi tạo client cũng sẽ được move vào trong Adapter class. Bằng cách này, chúng ta sẽ dễ swap gem với 1 gem khác. Bên cạnh đó, ta cũng đảm bảo được Single Responsibility Prrinciple, do InstagramAdapter class chỉ có nhiệm vụ là handle việc gọi API tới Instagram.

```ruby
class PhotosController < ApplicationController
  def index
    render json: instagram_client.recent_media_with_location
  end

  def instagram_client
    InstagramAdapter.new
  end
end
```

```ruby
# app/adapters/instagram/instagram_adapter.rb
class InstagramAdapter
  INSTAGRAM_ACCESS_TOKEN = "9999"

  def initialize
    @client = Instagram.client(access_token: INSTAGRAM_ACCESS_TOKEN)
  end

  def recent_media
    @recent_media ||= @client.user_recent_media(count: 50)
  end

  def recent_media_with_location
    recent_media.reject { |item| item.location.nil? }
  end
end
```

Như vậy, việc gọi API tới Instagram đã được tách hẳn ra bên ngoài. Nếu như gọi tới nhiều API của 1 bên thứ 3 khác chúng ta cũng có thể tạo nhiều Adapter. Ví dụ chúng ta cần gọi dữ liệu liên quan tới Commit và Repo trên Github, ta có thể chia thành 2 adapters: `app/adapters/github/commits_adapter.rb` và `app/adapters/github/repos_adapter.rb` thừa kế từ `app/adapters/github/base_adapter.rb`. Khá dễ dàng để quản lý và mở rộng đúng không :D

Tuy nhiên, tới đây chúng ta lại gặp phải 1 vấn đề: Nếu như chúng ta muốn xây dựng 1 wrapper đầy đủ để tương tác với API của bên thứ 3 (bao gôm việc handle requests/response khi gọi API đó) thì việc chỉ sử dụng adapters là không đủ.

Ví dụ như khi gọi API, dữ liệu trả về ở dạng XML nhưng chúng ta lại muốn sử dụng dạng JSON chẳng hạn. Nếu như gem chúng ta dùng không đáp ứng đầy đủ được, chúng ta sẽ phải tìm 1 giải pháp hợp lý khác. (Thay vì việc viết thêm hàm vào trong Adapter để xử lý response trả về ^^)

Để giải quyết vấn đề này, chúng ta sẽ xây dựng Serializers và Deserializers Objects. Trong đó: Serializers sử dụng để xử lý đầu vào trước khi nó được gửi lên server của bên thứ 3. Deserializers sử dụng để parse responses trả về từ API.

### Deserializers Objects
Trong thực tế, chúng ta rất dễ gặp những trường hợp như format của response trả về từ API không phù hợp với việc chúng ta đang cần làm.

Deserializers được lưu ở `/app/adapters/{api_service}/deserializers/`. Trong đó `api_service` có thể là `facebook`, `github`, ...

Example:

```ruby
module Instagram
  module Deserializers
    class RecentMedia
      attr_reader :response_body

      def initialize(response_body)
        @response_body = response_body
      end

      def success?
        stripped_response_body[:response_code] == "0"
      end

      def failed?
        !success?
      end

      def status_code
        stripped_response_body[:response_code]
      end

      def status_message
        stripped_response_body[:response_code_description]
      end

      private

      def stripped_response_body
        @stripped_response_body ||=
          @response_body[:hosted_page_authorize_response][:hosted_page_authorize_result]
      end
    end
  end
end
```

Khi đã có Deserializer này rồi, chúng ta sẽ dễ dàng xử lý response trả về trong Adapter:

```ruby
module Adapters
  class InstagramAdapter
    INSTAGRAM_ACCESS_TOKEN = "9999"

    def initialize
      @client = Instagram.client(access_token: INSTAGRAM_ACCESS_TOKEN)
    end

    def recent_media
      Instagram::Deserializer::RecentMedia.new(raw_recent_media)
    end

    def recent_media_with_location
      recent_media.reject { |item| item.location.nil? }
    end

    private

    def raw_recent_media
      client.user_recent_media(count: 50)
    end
  end
end
```

Ngoài cách làm trên, bạn cũng có thể deserialize 1 collections items

```ruby
module Instagram
  class RecentMedia

    def initialize(instagram_user_id)
      @instagram_user_id = instagram_user_id
    end

    def instagram_recent_media
      Instagram.user_recent_media(instagram_user_id, count: 50)
    end

    def feed
      @feed ||= instagram_recent_media.map { item| Deserializer::MediaItem.new(item) }
    end

    def items_with_location_present
      feed.reject { |item| item.restaurant_name.nil? }
    end

    def items_with_location_present_in_json
      items_with_location_present.map(&:to_json)
    end
  end
end

class InstagramAdapter
  INSTAGRAM_ACCESS_TOKEN = "9999"

  def initialize
    @client = Instagram.client(access_token: INSTAGRAM_ACCESS_TOKEN)
  end

  def user_recent_media
    itemize(@client.user_recent_media)
  end

  private

  def itemize(user_recent_items)
    user_recent_items.map { |item| MediaItem.new(item) }
  end
end
```

```
module Instagram
  class MediaItem
    attr_reader :item

    def initialize(item)
      @item = item
    end

    def restaurant_name
      item.location.name if item.location?
    end

    def image_url
      item.images.standard_resolution.url
    end
  end
end
```

### 4. Serializer Objects
Serializers Objects được dùng để chuẩn bị dữ liệu trước khi gửi nó lên trên server. Nó đặc biệt hữu dụng khi chúng ta dùng để tạo xml request.

```
module Dolcela
  class RecipeSerializer
    def initialize(id)
      @id = id
    end

    def method
      "post"
    end

    def request_body
      @xml_request_body ||= "<?xml version='1.0'?>" + Gyoku.xml(
        method_call: {
          method_name: "recipes.get",
          params: {
            param: {
              value: {
                struct: {
                  member: [
                    {
                      name: "content_id",
                      value: { int: @id }
                    },
                    {
                      name: "omit_author_data",
                      value: { int: 1 }
                    },
                    {
                      name: "get_basic_data",
                      value: { boolean: 1 }
                    },
                    {
                      name: "include_preparation_steps",
                      value: { boolean: 1 }
                    },
                    {
                      name: "get_action_shots",
                      value: { boolean: "1" }
                    },
                    {
                      name: "include_ingredients",
                      value: { boolean: 1 }
                    }
                  ]
                }
              }
            }
          }
        }
      )
    end
  end
end
```

```ruby
module Coolinarika
  module Adapter
    class RecipeAdapter < BaseAdapter
      def recipe(id)
        @id = id
        RecipeDeserializer.new(fetch_recipe)
      end

      private

      def fetch_recipe
        execute_request RecipeSerializer.new(@id)
      end
    end
  end
end
```

trong BaseAdapter, chúng ta sẽ viết 1 hàm excute_request chung cho tất cả các adapters để các adapter có thể gọi tới API được.

```
module Coolinarika
  module Adapter
    class BaseAdapter

      API_BASE_URL = "http://www.coolinarika.com/api/"

      def self.execute_request(request)
        RestClient::Request.execute(request.method,
                                    url: API_BASE_URL,
                                    payload: request.body,
                                    headers: { content_type: "application/xml" }
                                   )
      end
    end
  end
end
```

## Kết luận
Như vậy, chúng ta có thể thấy, lợi ích của việc sử dụng Adapter Objects là:

* Giúp tạo ra 1 absstraction layer xung quanh việc sử dụng API bên ngoài. Bằng cách này, chúng ta sẽ tách sự phụ thuộc vào lib ra 1 chỗ riêng, sẽ dễ để maintain/ scale sau này.
* Tiện dụng cho việc testing (do nó đã được tách thành các object riêng biệt, có đầu vào - trong serializer và đầu ra - trong deserializer).
* Code hướng đối tượng hơn, tuân thủ các quy tắc của OOD

Hy vọng thông qua bài viết, các bạn có thể nắm được nguyên lý cơ bản của Adapter Pattern để áp dụng vào projects sau này.

Tham khảo:

[Rails handbook: Adapter Pattern](https://github.com/infinum/rails-handbook/blob/master/Design%20Patterns/Adapters.md)

[Arkency Ruby on Rails Adapters](https://blog.arkency.com/2014/08/ruby-rails-adapters/)

[Adapter Design Pattern usage in Rails App](http://rustamagasanov.com/blog/2014/11/16/adapter-design-pattern-usage-in-rails-application-on-examples/)


















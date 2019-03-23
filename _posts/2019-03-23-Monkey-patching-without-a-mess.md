---
layout: post
title: Monkey Patching without a mess
tags:
- Ruby
---

## 1. Tản mạn
`Monkey Patching` là 1 cách cho phép lập trình viên có thể mở rộng hoặc chỉnh sửa 1 hệ thống phần mềm 1 cách tạm thời và cục bộ. Ruby là 1 ngôn ngữ linh hoạt, điều này được thể hiện rõ qua Monkey Patching.

Tất cả các class trong ruby đều là `Open Class`, do đó chúng ta có thể thay đổi bất cứ class naof ở bất cứ nơi đâu: thêm mới, override các methods đã tồn tại. Nghe có vẻ rất hay, mềm dẻo, linh hoạt. Tuy nhiên chính điều này làm cho Monkey path trong Ruby trở thành 1 con dao 2 lưỡi. Chúng ta sẽ tự tay cắt mình nếu như không control được tầm ảnh hưởng của nó.

Bài viết dưới đây sẽ đưa ra 1 vài hướng dẫn để giúp bạn kiểm soát tốt hơn các monkey patching methods của mình.

## 2. Monkey patching without making a mess
### 2.1 Đặt các monkey patching methods vào trong module
Ví dụ bạn muốn check 1 ngày có phải là cuối tuần hay không, bạn có thể viết:

```ruby
class DateTime
  def weekday?
    !sunday? && !saturday?
  end
end
```

Tuy nhiên, lời khuyên là: Khi bạn monkey patch 1 class, không nên chỉ đặt methods vào trong tên class như vậy.

Lý do:

* Nếu 2 thư viện cùng monkey-patch cùng 1 method, bạn sẽ không thể nào sử dụng được 1 trong 2 methods: Ví dụ: Trong gem rubocop có monkey-patch một method [strip_indent](https://github.com/rubocop-hq/rubocop/blob/master/lib/rubocop/core_ext/string.rb#L41). Nếu bạn cũng có ý định monkey-patch 1 method cho `String` và tình cờ cũng đặt tên method đúng như vậy thì sao? Method được monkey-patch trước sẽ bị ovverwritten và biến mất :D
* Nếu như có lỗi xảy ra, log sẽ báo ra lỗi giống như lỗi đó xảy ra trong class `DateTime` -> Khó debug hơn.
* Sẽ khó để bỏ require cho các monkey patches: Nếu bạn viết code như trên, và muốn loại bỏ methods đã được monkey patches, bạn chỉ có cách comment toàn bộ file, hoặc skip việc require files đó.
* Nếu bạn quên thêm dòng `require 'date'` trước khi chạy monkey patch này, bạn sẽ vô tình định nghĩa lại class `DateTime` thay vì patching nó.

Thay vào đó, hãy đặt các monkey patches vào trong 1 module:

```ruby
module CoreExtensions
  module DateTime
    module BusinessDays
      def weekday?
        !sunday? && !saturday?
      end
    end
  end
end
```

Bằng cách này, bạn có thể nhóm các monkey patches lại với nhau. Nếu có lỗi xảy ra, nó sẽ chỉ chính xác xem vấn đề nằm ở đâu. Và bạn có thể include/ exclude chúng 1 cách đơn giản: comment or uncomment 1 dòng.

```ruby
# Actually monkey-patch DateTime
DateTime.include CoreExtensions::DateTime::BusinessDays
```

### 2.2 Đặt các monkey patching methods cạnh nhau
Khi bạn muốn monkey patch các core classes, thực tế là bạn đang add thêm core Ruby APIs. Do đó, khi bạn vừa vào 1 project, liệu có cách nào để biết được nó có monkey patch core API không? Thật may là phần lớn các ruby dev đều có conventions cho monkey patching.

Trong Rails app, các file monkey patch sẽ được đặt tại đường dẫn `lib/core_extensions/class_name/group.rb`. Với đoạn code bên trên:

```ruby
module CoreExtensions
  module DateTime
    module BusinessDays
      def weekday?
        !sunday? && !saturday?
      end
    end
  end
end
```

sẽ được đặt trong: `lib/core_extensions/date_time/business_days.rb`.

Do đó, bất cứ 1 new dev nào cũng có hể tìm kiếm Ruby files trong `lib/core_extensions` và học cách để thêm mới 1 ruby APIs.

### 2.3 Sử dụng Refinements để giới hạn tầm ảnh hưởng của các methods monkey patches
Bạn có thể đọc thêm về [refinements](https://docs.ruby-lang.org/en/2.4.0/syntax/refinements_rdoc.html).

Với refinements, bạn có thể monkey-patch 1 class mà không làm ảnh hưởng tới mọi chỗ sử dụng class đó trong project. Refinement sẽ chỉ active trong 1 module khi nào bạn muốn và require nó vào.

```ruby
module M
  refine String do
    def titleize
      # Implementation
    end
  end
end
```

Nếu muốn sử dụng, bạn cần dùng hàm `using` để gọi nó:

```ruby
# In a file
# not activated here
using M
# activated here
class Foo
  # activated here
  def foo
    # activated here
  end
  # activated here
end
# activated here
```

```ruby
In a class:
# not activated here
class Foo
  # not activated here
  def foo
    # not activated here
  end
  using M
  # activated here
  def bar
    # activated here
  end
  # activated here
end
# not activated here
```

Như vậy, chúng ta có thể giới hạn tầm ảnh hưởng của 1 monkey patch methods. Tránh việc dính bug mà k hiểu nguyên nhân tới từ đâu.

### 2.4 Hãy nghĩ về các ảnh hưởng của monkey patching methods bạn viết
Khi bạn monkey patch 1 class, bạn thường implement những method phù hợp với hoàn cảnh sử dụng lúc đó. Điều này có thể hữu ích ngay lúc đó, tuy nhiên điều này có thể khiến bạn hoặc người đọc code bối rối nếu đọc lại sau này.

Hãy suy nghĩ về những side effections. Dưới đây là 1 số gợi ý cho bạn:

* Handle các input vào: Hãy luôn cẩn thận với các input đầu vào, Nó có thể là `nil`, `string`, `array`, ... Cố gắng cover các trường hợp, raise exception khi cần thiết, hoặc đơn giản, `to_s` nếu cần trước khi execute code của mình.
* Handle error một cách rõ ràng: Chúng ta có thể dễ dàng throwing `ArgumentError` với message nếu thấy 1 input không đúng với expect. Cố gắng không để raise `NoMethodError`, nó sẽ khó cho người dùng khi debug.
* Lưu lại comment về method bạn viết: Nên có comment cho các methods của bạn, nó dùng để làm gì, nhận vào tham số như thế nào, ... để mọi nguời có thể dễ dàng sử dụng khi đã hiểu được bạn muốn làm gì với method đó.

## 3. Kết luận
Monkey patching core classes không phải lúc nào cũng là không tốt. Nếu bạn làm đúng cách, nó sẽ khiến code bạn trông sạch hơn, rõ ràng hơn khi đọc (và trông "ruby" hơn :D). Tuy nhiên, nó cũng có thể là con dao hai lưỡi khiến việc debug trở nên khó khăn.

Hãy đặt các patches cạnh nhau, nhóm trong 1 modules, handle các tác nhân không mong muốn để monkey patches giữ an toàn hơn cho code của bạn :D


References:

* https://www.justinweiss.com/articles/3-ways-to-monkey-patch-without-making-a-mess/
* https://spin.atomicobject.com/2017/12/29/monkey-patching-refinements/
* https://dzone.com/articles/how-not-to-cut-your-source-with-sharp-knife-as-mon



























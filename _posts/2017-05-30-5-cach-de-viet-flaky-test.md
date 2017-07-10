---
layout: post
comments: true
title: 5 ways to write a flaky test
tags:
- Become a Rubyist
---

## Mở đầu
Trong quá trình phát triển phần mềm, chúng ta muốn đảm bảo tính đúng đắn, chính xác của 1 tính năng đồng thời muốn thích ứng với công việc maintain phát triển mở rộng sau này, code base tốt chỉ là 1 phần, phần còn lại chính là viết Test. Những năm gần đây, các công ty hầu hết đều đưa việc viết Test vào trong quy trình làm việc, không thể phủ nhận là phần trăm coverage function tăng lên rất nhiều. Tuy nhiên, lập trình viên có thể gặp phải một vài vấn đề oái oăm trong quá trình viết test.

Đã bao giờ bạn viết test và chạy pass ở máy mình, nhưng khi push code lên, CI chạy test lại bị fail? Bạn chạy thử lại ở local thì vẫn pass, chạy lại lần nữa thì lại bị fail? (boiroi). Mình đã từng gặp trường hợp như thế mà không hiểu tại sao. Sau khi google thử thì thấy có khá nhiều người cũng gặp trường hợp tương tự khi viết test, và người ta gọi những  test như vậy là `flaky test` (theo cách nói dân dã là test bị failed một cách "hên xui"). Khi project của bạn có tới khoảng 10k tests thì cơ hội gặp phải loại test này sẽ tăng lên rất nhiều. Và chỉ 1 lần gặp phải thôi, việc CI phải build lại sẽ mất rất nhiều thời gian. Bài viết dưới đây sẽ hướng dẫn các bạn một vài cách để viết (hoặc tránh) flaky test.

## Các cách để tránh Flaky Test
### 1. Random factories

```Ruby
# Giả sử email của customer là duy nhất
10.times do
  Customer.create!(email: Faker::Internet.safe_email)
end
```
Bạn có thấy vấn đề gì ở đây không? Trong đại đa số trường hợp, test này sẽ pass. Nhưng đôi khi, [Faker](https://github.com/stympy/faker) có thể trả về 1 email random mà đã được sử dụng trước đó và tất nhiên, test của bạn sẽ bị crash với lỗi email không unique :D

Bạn nên thay đoạn code thành:
```Ruby
10.times do |n|
  Customer.create!(email: Faker::Internet.safe_email(n.to_s))
end
```
Tham số thêm vào sẽ yêu cầu Faker trả về email thứ n, thay vì random ra 1 cái email bất kì.

### 2. Chắc chắn đã order các records
```Ruby
assert_equal([1, 2, 3], @products.pluck(:quantity))
```
Ngay cả khi test này pass trong đại đa số trường hợp, câu query SELECT mà không có ORDER sẽ khôgn thể đảm bảo được thứ tự đầu ra của các records. Để tránh trường hợp fail test do random order, bạn nên chỉ chính xác thứ tự order:

```Ruby
assert_equal([1, 2, 3], @products.pluck(:quantity).sort) # or assert_equal([1, 2, 3], @products.order(:quantity).pluck(:quantity))
```

### Nói không với việc sử dụng Global Environment
``` Ruby
BulkEditor.register(User) do
  attributes(:email, :password)
end
assert_equal [:email, :password], BulkEditor.attributes_for(@user)
```
Trong trường hợp này, `BulkEditor` được sử dụng là một biến toàn cục để lưu trữ danh sách những models đã đăng ký. Việc viết test như bên trên sẽ làm ảnh hưởng tới các test case chạy phía sau test này. Hãy làm cho việc chạy test **không phụ thuộc vào thứ tự chạy test**.
Giải pháp:
```Ruby
setup to
  BulkEditor.register(User) do
    attributes(:email, :password)
  end
end

teardown do
  BulkEditor.unregister(User)
end
```

Một ví dụ nữa cho phần này:
```Ruby
test "something" do
  SomeGem::VERSION = '9999.99.11'
  assert_not @provider.supported?
end
```
Bất cứ test nào chạy sau test này sẽ gặp lỗi "broken value of `SomeGem::VERSION`". Nó sẽ dẫn tới 1 cảnh báo: `warning: already initialized constant SomeGem::VERSION`

Giải pháp:
```Ruby
test "something" do
  # only the block will get modified value of the constant
  stub_constant(SomeGem, :VERSION, '9999.99.99') do
    assert_not @provider.supported?
  end
end
```

### 4. Time-sensitive tests
```Ruby
post = publish_delayed_post
assert_equal 1.hour.from_now, post.published_at
```
Bình thường thì test kia sẽ pass. Nhưng trong 1 số trường hợp, post publishing sẽ mất nhiều thơi gian hơn khoảng 1 milli giây (Có thể do phần cứng xử lý chậm việc insert). và `pushlished_at` sẽ khác với `1.hour.from_now`.

Có một helper đặc biệt `assert_in_delta` có thể sử dụng trong trường hợp này:
```Ruby
post = publish_delayed_post
assert_in_delta 1.hour.from_now, post.published_at, 1.second
```

Một cách khác, bạn có thể "đóng băng" thời gian với thư việc như [Timecop](https://github.com/travisjeffery/timecop)

### 5. Require-dependent tests

Chúng ta có 2 loại test classes: một cho phép gọi remote HTTP và loại còn lại thì không. Ví dụ:
```Ruby
# test/unit/remote_api_test.rb
require 'remote_test_helper'

class RemoteServiceTest < ActiveSupport::TestCase
  test "something" do
    # ...
  end
end

# test/unit/simple_test.rb
require 'test_helper'

class SimpleTest < ActiveSupport::TestCase
  test "something" do
    # ...
  end
end
```

Một số test sử dụng `remote_test_helper` để cho phép test case tạo 1 external HTTP calls. Bạn có thể đoán, nó có thể chạy OK khi bạn chạy từng test một. Nhưng khi chạy trên CI, phụ thuộc vào thứ tự chạy test, nó có thể đến trường hợp **tất cả các test chạy sau remote case cũng cho phép thực hiện external calls**.

Bạn cần luôn nhớ rằng: `require` là toàn cục.

Một giản pháp tốt hơn đó là sử dụng một macro để chỉ modify context của các test đặc biệt:

```Ruby
# test/unit/remote_api_test.rb
require 'test_helper'

class RemoteServiceTest < ActiveSupport::TestCase
  allow_remote_calls!

  test "something" do
    # ...
  end
end

# test/unit/simple_test.rb
require 'test_helper'

class SimpleTest < ActiveSupport::TestCase
  test "something" do
    # ...
  end
end
```
## Kết luận
Khi dự án phát triển đến một giai đoạn nhất định, việc phải đối mặt với Flaky test sẽ là một trở ngại lớn, làm mất rất nhiều thời gian trong việc tìm hiểu, fix test, build CI. Bài viết trên hướng dẫn cho các bạn một vài cách để có thể deal với Flay Test. Hy vọng nó có thể giúp ích cho các bạn trong quá trình phát triển phần mềm sau này. ;)

Bài viết dịch từ: http://iempire.ru/2016/10/21/flaky-tests/
Nếu bạn muốn đọc thêm về Flaky test, bạn có thể đọc ở đây:
* https://thoughtbot.com/upcase/videos/rspec-bisect
* https://semaphoreci.com/community/tutorials/how-to-deal-with-and-eliminate-flaky-tests
* https://bugs.ruby-lang.org/issues/12776
* https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html


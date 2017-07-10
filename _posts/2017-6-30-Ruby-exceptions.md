---
layout: post
comments: true
title: Ruby exceptions
tags:
- Become a Rubyist
---


# Mở đầu

Trong quá trình lập trình với Ruby, chắc hẳn việc bắt gặp các exception là không thể tránh khỏi. Có thể bạn biết về `rescue`, `raise`, ... để làm việc với exception, nhưng bạn đã thự sự hiểu về cách hoạt động của chúng chưa?

Bài viết này sẽ thảo luận về 1 số định nghĩa, phương pháp để bạn có thể deal với exceptions trong Ruby.

# Ruby exception và cách xử lý

Nếu như bạn không biết exceptions trong Ruby hoạt động như thế nào, hãy thử check qua ví dụ này:

```Ruby
puts a # => undefined local variable or method `a' for main:Object (NameError)
```
`a` là một biến chưa được định nghĩa và đó là lý do tại sao chúng ta nhận được đoạn exception với 1 tin nhắn mô tả như trên: "Biến hoặc hàm 'a' chưa được định nghĩa.'". Bên cạnh đó, chúng ta cũng có thể nhìn thấy loại exception: `NameError`.

## 1. rescue

Nếu chúng ta biết code của chúng ta có thể có 1 exception nào đó, chúng ta có thể bắt chúng sử dụng `begin ... rescue`.

```Ruby
begin
  puts a
rescue
  puts 'Something bad happened'
end
```

Bên cạnh đó, chúng ta cũng có thể định nghĩa trong `rescue` chính xác loại lỗi mà chúng ta có thể bắt:

```Ruby
begin
  puts a
rescue NameError => e
  puts e.message
end
```

Để bắt những loại exceptions khác nhau bằng `rescue`, chúng ta có thể dùng cách:

```Ruby
def foo
  begin
    # logic
  rescue NoMemoryError, StandardError => e
    # process the error
  end
end
```

Có một điều rất quan trọng mà chúng ta cần phải biết. Đó là: **Mặc định, `rescue` bắt tất cả các errors được kế thừa từ `StandarError`**. Do đó, nó sẽ không bắt được `NotImplementedError` hay `NoMemoryError`,.. Để hiểu rõ hơn về các exceptions có thể được bắt bởi `rescue`, chúng ta có thể check cây thừa kế về exceptions trong Ruby:

![](http://rubyblog.pro/images/posts/exceptions_hierarchy.jpg)

Như chúng ta có thể thấy, `rescue` có thể định nghĩa 1 biến, mà từ đó, chúng ta có thể access vào exception object:

```Ruby
def foo
  begin
    raise 'here'
  rescue => e
    e.backtrace # ["test.rb:3:in `foo'", "test.rb:10:in `<main>'"]
    e.message # 'here'
  end
end
```
Bạn có thể xem thêm về những methods của object này ở [đây](http://ruby-doc.org/core-2.3.0/Exception.html)

## 2. ensure

Ruby cung cấp một từ khoá thú vị khác giúp thao tác với exception, đó là `ensure`. Ngay cả khi exception có xuất hiện hay không, Ruby cũng sẽ thực hiện đoạn code trong `ensure`. Thông thường, các lập trình viên sử  dụng nó để đóng connection tới DB hay remove các file tạm, ...

```Ruby
begin
  puts a
rescue NameError => e
  puts e.message
ensure
  # clean up the system, close db connection, remove tmp file, etc
end
```

Có một điều rất quan trọng cần phải biết về `ensure`. Nếu bạn khai báo `return` trong `ensure` nhưng không định nghĩa `rescue`, `ensure` sẽ raise lên 1 exception. Chúng ta hãy xem qua ví dụ:

```Ruby
def foo
  begin
    raise 'here'
  ensure
    puts 'processed'
  end
end

foo
# processed
# => `foo': here (RuntimeError)
```
Trong ví dụ trên, Ruby chạy đoạn code trong `ensure` trước, sau đó mới throw exception bởi vì chúng ta không bắt nó trong `rescure`. Bây giờ, chúng ta sẽ xem 1 ví dụ khác về việc return trong `ensure`:

```Ruby
def foo
  begin
    raise 'here'
  ensure
    return 'processed'
  end
end

puts foo # => processed
```
Không có lỗi `RuntimeError`! Chúng ta vẫn không bắt ngoại lệ, nhưng Ruby trả về 'processed' từ `ensure` và không throw error.

## 3. raise

Module `Kernel` có method `raise` cho phép chúng ta throw errors. Đây là 1 alias cho `raise` - `fail`, nhưng thông thường, chúng ta gặp `raise` nhiều hơn.

Nếu bạn chỉ gọi raise mà không có param gì, nó sẽ throw error:
```
raise # => `<main>': unhandled exception
```
Exception này không nói gì với developer nên thông thường, bạn nên thêm vào đó ít nhất 1 đoạn tin nhắn nào đó:
```
raise 'Could not read from database' # => Could not read from database (RuntimeError)
```
Bây giờ, bạn đã thấy `raise` trả về 1 đoạn tin nhắn và đó là lỗi `RuntimeError`. Mặc định, `raise` throw `RuntimeError`.

Chúng ta có thể định chính xác exception mà ta muốn raise:

```
raise NotImplementedError, 'Method not implemented yet'
# => Method not implemented yet (NotImplementedError)
```

Có một điều thú vị là `raise` gọi tới method `#exception` cho bất kì class nào bạn truyền tới nó. Trong trường hợp ở trên, nó gọi tới `NotImplementedError#exception`. Điều này cho phép chúng ta có thể dễ dàng thêm 1 exception support vào 1 class nào đó. Chỉ cần trong class đó đặt 1 hàm exception:

```
class Response
  # ...
  def exception(message = 'HTTP Error')
    RuntimeError.new(message)
  end
end

response = Response.new
raise response # => HTTP Error (RuntimeError)
```

## 4. $!

Còn 1 điều thú vị nữa về exception trong Ruby nữa. Đó là khi exception xuất hiện, Ruby lưu trữ nó trong global variable `$!`.

```
$! # => nil
begin
  raise 'Exception'
rescue
  $! # <RuntimeError: Exception>
end
$! # => nil
```

Như chúng ta có thể nhìn thấy ở đây, chúng ta có 1 exception được assigned vào `$!` trong `rescue`, nhưng ngay sau đó, khi ra ngoài rescue, biến này lại trở thành `nil`.

## 5. retry

Ruby cung cấp cho chúng ta 1 cách để chạy code trong `begin` part 1 hoặc nhiều lần. Thử tưởng tượng chúng ta có 1 service mà đôi khi không cần required data. Chúng ta có thể đóng gói các requests tới services này trong 1 vòng lặp, nhưng chúng ta có thể sử dụng `retry` để chạy đoạn code trong `begin` thêm 1 lần nữa:

```
tries = 0
begin
  tries += 1
  puts "Trying #{tries}..."
  raise 'Did not work'
rescue
  retry if tries < 3
  puts 'I give up'
end
# Trying 1...
# Trying 2...
# Trying 3...
# I give up
```

Đoạn code trên khá là đơn giản. Nếu code trong `begin` throws 1 exception, chúng ta sẽ thử chạy nó thêm 1 lần nữa. Ý tưởng này khá là thú vị. Nhưng bạn nên chú ý là có nhiều cách handling error của bên thứ ba khá hay như: [article](https://martinfowler.com/bliki/CircuitBreaker.html), [gem](https://github.com/wsargent/circuit_breaker)



# Kết luận

Trên đây mình đã trình bày 1 vài vấn đề về Ruby exception, tất cả những gì bạn cần nhớ là:

* `raise` mặc định sẽ throw `RuntimeError`
* `rescue` mặc định chỉ bắt `StandardError` và tất cả các exception kế thừa từ nó.
* Khai báo `return` trong `ensure` mà không có error handling sẽ chặn exception.
* Trong quá trình error handling, Ruby lưu trữ exception trong biến `$!`
* Ruby có `retry`

Hy vọng sau bài viết này bạn có thể hiểu hơn về Exception trong Ruby. Cảm ơn các bạn đã dành thời gian đọc bài :D

Nguồn: rubyblog.pro/2017/03/exceptions

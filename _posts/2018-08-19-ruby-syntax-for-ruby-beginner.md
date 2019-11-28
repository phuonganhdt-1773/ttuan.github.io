---
layout: post
title: Basic Ruby Syntax for Beginner
tags:
- Ruby
---

Những methods, data types, tips cần biết khi bắt đầu học Ruby.

## String
Một string là một chuỗi kí tự nằm bên trong dấu nháy kép (`""`). Sử dụng để hiển thị text và data.

Ví dụ:
```ruby
"I like chocolate"
```
Còn một cách viết khác đó là sử dụng dấu nháy đơn (`''`)
```ruby
'Ruby is awesome'
```
#### Important methods:
* `size`
* `empty?`
* `include?`
* `gsub`
* `split`

#### More methods:
[Ruby String Methods (Ultimate Guide) - RubyGuides](https://www.rubyguides.com/2018/01/ruby-string-methods/)

## Hashes
Một Hash (`{}`) là một cặp giá trị key-value (`a => b`). Sử dụng như 1 dictionary. Bạn có thể truy cập các phần tử của hash qua key của chúng. Keys cần phải là unique.

#### Example
```ruby
# With symbol keys
## Create
h = { a: 1, b: 2, c: 3 }

## Access
h[:a]

## Set
h[:test] = 10

# With string keys
h = { "a" => 1, "b" => 2, "c" => 3 }
```

#### Important methods
* `key?`
* `fetch`
* `new`
* `merge`

#### More methods
[Class: Hash (Ruby 2.6.4)](http://ruby-doc.org/core-2.6.4/Hash.html)

## Symbol
Symbol là một string tĩnh được sử dụng cho việc định danh (ví dụ như keys trong hash). Chúng luôn được bắt đầu bằng một dấu hai chấm. (`:bacon`).

Khi được sự dụng bên trong hash (`{}`), dấu `:` sẽ được revert vị trí.
#### Example
```ruby
{ abc: 1 }
```

Symbol trong case này là: `:abc`

#### More methods
[What Are Ruby Symbols & How Do They Work? - RubyGuides](https://www.rubyguides.com/2018/02/ruby-symbols/)

## Nil
Một singleton class (only one object allowed) đại diện cho loại giá trị "not found" :v
Có thể được hiểu như `false` trong một vài trường hợp.

#### Learn more
* [Everything You Need to Know About Nil - RubyGuides](https://www.rubyguides.com/2018/01/ruby-nil/)
* [Understanding Boolean Values in Ruby - RubyGuides](https://www.rubyguides.com/2019/02/ruby-booleans/)

## Array
Một object được sử dụng để lưu trữ 1 list của objects. Một array có thể contain nhiều loại dữ liệu (`a = [1, "abc", []])`, bao gồm cả những array khác.

Bạn có thể access array bằng cách sử dụng `index` của chúng. (`a[0]ư`) và nested array với `a[0][0]`

#### Example
```ruby
a = []

a << 10
a << 20
a << 30

a
# [10, 20, 30]
```

#### Important methods

* `size`
* `emtpy?`
* `push/pop`
* `join`
* `flatten`

#### More methods
[Class: Array (Ruby 2.6.4)](http://ruby-doc.org/core-2.6.4/Array.html)

## Enumerable
Một Module của Ruby, được sử dụng để tuongw tác với các phần tử của bất kì class nào, chỉ cần chúng implement `each` method, vd: Array, Range, Hash.

#### Important methods
* `map`
* `select`
* `inject`

#### More
[A Basic Guide to The Ruby Enumerable Module (+ my favorite method)](https://www.rubyguides.com/2016/03/enumerable-methods/)

## File

Một class giúp bạn làm việc với files trong Ruby: Đọc file, ghi file, lấy file size, ...

#### Example

```ruby
File.read("/tmp/test.txt")
```

#### Important methods

* `read`
* `write`

#### More

[How To Read & Write Files in Ruby (With Examples)](https://www.rubyguides.com/2015/05/working-with-files-ruby/)

## Regular Expression

Nếu bạn đang tìm kiếm từ 1 patterns, substrings hay một chuỗi đặc biệt nào đó bên trong một string, regular expression chính là thứ bạn cần :D

Chúng có thể được sử dụng cho việc validate email, phone numbers. Hoặc dùng để extract information từ text.

#### Example

```ruby
"aaaa1".match?(/[0-9]/)
# true

"".match?(/[0-9]/)
# false
```

#### More

[Ruby Regular Expressions (Complete Tutorial)](https://www.rubyguides.com/2015/06/ruby-regex/)

## Ruby Gems & Bundler
Ruby gems là các thư viện mà bạn có hể download để sử dụng trong code Ruby. Những thư viện này cung cấp một số functions mới mà bạn có thể dùng để giải quyết bài toán của mình.

Ví dụ, trong Rails application, bạn có thể dễ dàng add authentication với Devise gem, hoặc phân trang với gem Kaminari.

##### More
[Ruby Gems, Gemfile & Bundler (The Ultimate Guide) - RubyGuides](https://www.rubyguides.com/2018/09/ruby-gems-gemfiles-bundler/)

## Class & Object-Oriented Programming

Ruby là một ngôn ngữ hướng đối tượng. Mọi thứ trong cuộc sống đều là các object. Object bao gồm data và các methods.

#### Important methods
* `class`
* `include / extend`

#### More
* [How to Write Your Own Classes in Ruby (Tutorial + Examples)](https://www.rubyguides.com/2019/02/ruby-class/)
* [YouTube](https://www.youtube.com/watch?v=LuTTUNnSj6o&list=PL6Eq_d2HYExeKIi4d9rUEoD6qSiKS4vfe&index=2)

## Types of Variables
Một biến là 1 label cho một object, để chúng ta có thể access vào object đấy. Việc liên kết biến này với object đó được gọi là "variable assignment".

#### Example
```ruby
a = 1
```

Có khá nhiều loại biến trong Ruby:

* Local variables (`something`)
* Instance variables (`@something`)
* Constants (`Something` / `SOMETHING`)
* Global variables (`$something`)

Mỗi loại biến sẽ có thể được access từ những vị trí khác nhau (trong/ ngoài class, function, ... )

## %w, %i, %q, %r, %x
Chúng ta có thể khởi tạo objects bằng cách sử dụng dấu `%`

#### Example
```ruby
array_of_strings = %w(apple orange coconut)
array_of_symbols = %i(a b c)

string = %q(things)

regular_expression = %r([0-9])
```

## Use of Parenthesis
Dấu ngoặc `()` và dấu chấm phẩy `;` trong Ruby không bắt buộc, nhưng chúng ta vẫn có thể sử dụng bình thường :v

#### Some basic rules
* Không sử dụng dấu ngoặc khi định nghĩa một method mà không có arguments => `def foo`
* Nên sử dụng dấu ngoặc với method arguments => `def foo(a, b, c)`
* Nên sử dụng dấu ngoặc khi bạn muốn thay đổi độ ưu tiên thực hiện, hoặc priority hoặc một operation => `(a.size + b.size) * 2`


Trên đây là những kiến thức cơ bản mà bạn nên chú ý khi bắt đầu học Ruby. Hy vọng bài viết sẽ có ích cho mọi người :D

Nguồn: [Ruby Syntax Reference For Beginners - RubyGuides](https://www.rubyguides.com/2019/10/ruby-syntax/)

---
layout: post
title: Ruby Quick Tips
tags:
- Ruby
---

# Mở đầu
Gần đây mình có config rubocop cho 1 dự án. Khi chạy rubocop thì nó thông báo rất nhiều cảnh báo (điều này cũng khó tránh vì code đã viết được 1 thời gian dài rồi). Tuy nhiên có một vài điều mình không hiểu tại sao Rubocop lại cảnh báo như thế. Viết như cũ thì có gì sai? Sau khi google thử, mình có tìm được 1 [bài viết](https://www.toptal.com/ruby/tips-and-practices) tổng hợp 1 số tips khá hay, mình đã chọn ra vài tips để giới thiệu với mọi người :D

# Tips
### 1. Sử dụng `!!` (Double-bang) cho các giá trị boolean
```ruby
# BAD
def revealed?
    revealed
end
```
Trong phần lớn các ngôn ngữ lập trình, bao gồm cả Ruby, dấu `!` sẽ trả về giá trị đảo của toán hạng dưới dạng boolean. Nên khi bạn đặt 2 dấu `!` cạnh nhau, nó sẽ convert giá trị thành 1 gía trị boolean.
```ruby
# GOOD
def revealed?
    !!revealed
end
```

### 2. Tránh sử dụng `Array.new` với một giá trị mặc định
```ruby
# BAD
@masked_word = Array.new(word.size, '_').join

```
```ruby
# GOOD
@masked_word = "_" * word.size
```
Sử dụng `Array.new` với giá trị default khá là nguy hiểm vì mảng sẽ được fill với cùng 1 instance. Ví dụ:
```ruby
arr = Array.new(3, "foo")
#=> ["foo", "foo", "foo"]
arr[0].replace("bar")
#=> "bar"
arr
#=> ["bar", "bar", "bar"]
```

### 3. Tránh sử dụng `white !(not)`
```ruby
def play
    while !exceeded_max_guesses?
    	....
    end
end

```
Thay vì sử dụng `white not`, chúng ta có thể sử dụng `until`:
```ruby
until exceeded_max_guesses?
    ....
end
```
hoặc có thể sử dụng positive case:
```ruby
while has_remaining_guesses?
    ....
end
```

### 4. Sử dụng `loop do` thay vì `white(true)`
```ruby
# BAD
def play
    get_guess
    while true
        ...
        ...
    end
end
```
```ruby
# GOOD
def play
    get_guess
    loop do
        ...
        ...
    end
end
```
Vòng lặp `loop do` sẽ trông clean hơn khi làm việc với các external iterators. ví dụ:
```ruby
# No loop do
my_iterator = (1..9).each
begin
    while(true)
    	puts my_iterator.next
    end
rescue StopIteration => e
    puts e
end
```
```ruby
my_iterator = (1..9).each
loop do
    puts my_iterator.next
end
```
Khi dùng `loop do`, exception sẽ tự động được handle. Ngoài ra còn 1 lý do khác nữa đó là `loop do` cho phép chúng ta lặp giữa 2 collections cùng 1 lúc.
```ruby
iterator = (1..9).each
iterator_two = (1..5).each

loop do
    puts iterator.next
    puts iterator_two.next
end

#=> 1,1,2,2,3,3,4,4,5,5,6.
```

### 5. Tránh sử dụng `before_action` để set biến. Hãy dùng Memoized Finder
`before_action` filters được sử dụng để làm code trong controller DRY hơn. DRY là 1 concept rất đúng. nhưng đôi khi nó sẽ làm cho controller của bạn trở nên phức tạp hơn.

```ruby
# NOT GOOD
before_action :set_contest

def index
	@entries = @contest.entries
end

private

def set_contest
    @contest = Contest.find(params[:contest_id])
end
```
```ruby
# BETTER
def index
	@entries = contest.entries
end

private

def contest
    @_contest ||= Contest.find(params[:contest_id])
end
```

### 6. Symbol Garbage Collector là gì?
Ruby 2.2 giới thiệu 1 tính năng mới. Đó là Symbol garbage collector (GC). Tại sao nó lại là 1 tính năng lớn? Vì trước bản Ruby 2.2, symbol sẽ "sống" mãi :v

```ruby
# Ruby 2.1
before = Symbol.all_symbols.size
100_000.times do |i|
  "toptaller_#{i}".to_sym
end
GC.start
after = Symbol.all_symbols.size
puts after - before
# => 100001

# Ruby 2.2
before = Symbol.all_symbols.size
100_000.times do |i|
  "toptaller#{i}".to_sym
end
GC.start
after = Symbol.all_symbols.size
puts after - before
# => 1
```
Bạn nên chú ý điều này khi maintain những dự án vẫn dùng ruby bản cũ, < 2.2

### 7. Làm thế nào để sử dụng Boolean Operators trên Non Boolean Values?
Chúng ta đều biêts tới `||=` sẽ update biến với giá trị được trả của vế bên phải nếu biến chưa được define. Nhưng bạn có biết về `&&=`

Để hiểu rõ hơn, hãy xem ví dụ dưới đây:

```ruby
str &&= str + "Talent" #=> nil
str = "Top"
str &&= str + "Talent" #=> "TopTalent"
```

Như bạn thấy trong ví dụ trên: `str &&= str + "Talent"` = `str = str + "Talent" if str`.

### 8. Giữ tất cả các biến constants ở 1 chỗ
Những dữ liệu mà không thay đổi trong suốt quá trình chạy application được gọi là constants. Chúng ta nên lưu trữ nó vào trong 1 file để dễ quản lý.

```ruby
# File config/settings.rb

ADMIN_EMAIL  = 'eqbal@toptalcom'
ADMIN_PASS   = 'my_awesome@pass'

```

### 9. In ra Data sau "END" block
Trong Ruby có 1 keyword khá hay là `__END__`. Nó được sử dụng để nói với parser để dừng lại việc executing source file. Nó thường được sử dụng để lưu code snippets hoặc notes về script bên trên.

Nội dung của file bên dưới chữ `__END__` có thể được truy vấn với từ khoá `DATA` trong IO object.

```ruby
DATA.each_line.map(&:chomp).each do |url|
    `open "#{url}"`
end

__END__
https://www.toptal.com/
www.toptal.com/ruby/tips-and-practices
```

Nếu nội dung là CSV:

```ruby
require "csv"

CSV.parse(DATA, headers: true).each do |row|
  puts "#{row['Name']} => #{row['URL']}"
end

__END__
Id,Name,URL
1,Eqbal,https://www.toptal.com/resume/eqbal-quran
2,Diego,http://https://www.toptal.com/resume/diego-ballona
```

### 10. Tránh sử dụng `rescue Exception => e`
Đôi khi ta sẽ bắt gặp những đoạn code như:

```ruby
def do_something!
  # ... do something ...
  success
rescue Exception => e
  failed e
end
```

Đây là 1 việc rất tệ. `Exception` là root của cây phả hệ exception trong Ruby. Tất cả các exception sẽ raise lên 1 subclass của `Exception`. Khi bạn rescuing Exception, nó có thể bắt:

```ruby
SystemStackError
NoMemoryError
SecurityError
ScriptError
  NotImplementedError
  LoadError
    Gem::LoadError
  SyntaxError
SignalException
  Interrupt
SystemExit
  Gem::SystemExitException
```

Bạn có muốn rescue `Exception` trong những trường hợp như `NoMemoryError`? Nếu như bên trong rescue Exception là 1 thao tác gửi mail, bạn có muốn gửi mail tới người dùng trong trường hợp job fails?

Thay vào đó, hãy dùng `rescue => e` ~ `rescue StandardError => e` hoặc chỉ định rõ exception mà bạn muốn bắt.

```ruby
def do_something!
  # ... do something ...
  success
rescue => e
  failed e
end
```

### 11. Sử dụng Splat Expander
Toán tử Splat, `*`, trong Ruby được dùng để làm đơn giản code hơn. Ví dụ

```ruby
t,o,p,t,a,l = *(1..6) #=> [1, 2, 3, 4, 5, 6]
puts p #=> 3
```

hoặc có thể dụng cho 1 object:

```ruby
Toptaller = Struct.new(:name, :tech)
toptaller = Toptaller.new("Eqbal", "Ruby")
name, tech = *toptaller
puts tech #=> Ruby
```

### 12. Tránh sử dụng `nil`
`nil` là 1 instance của `NilClass` và là 1 loại `singleton` đặc biệt của Ruby.

```ruby
>> nil.class
=> NilClass
>> nil.singleton_class
=> NilClass
```
Vấn đề ở chỗ khi bạn sử dụng `nil`, nó có thể dẫn đến 1 số lỗi `undefined method .... for nil:NilClass`. Ví dụ:

```ruby
class Toptaller
  attr_reader :name

  def self.find(id)
    return nil if id == ''
    new(id)
  end

  def initialize(name)
    @name = name
  end
end

Toptaller.find('Eqbal') #=> #<Toptaller:0x007fb6b3635f58 @name="Eqbal">
Toptaller.find('')      #=> nil
```

```ruby
toptallers = ids.map {|id| Toptaller.find(id)}

toptallers.each {|top| puts top.name}
=> eqbal
=> NoMethodError: undefined method `name' for nil:NilClass
```

Trường hợp này chúng ta rất hay gặp :v Hãy dùng Null Object Pattern để giải quyết nó.

```ruby
class Toptaller
  attr_reader :name

  def self.find(id)
    return nil if id == ''
    new(id)
  end

  def initialize(name)
    @name = name
  end

  def self.get_toptaller(name)
    case name
    when nil
      NilToptaller.new
    else
      Toptaller.find(name)
    end
  end
end

names = ["Eqbal", nil, "Diego"]

class NilToptaller
  def name
    "No name."
  end
end

toptallers = names.map {|n| Toptaller.get_toptaller(n).name}
["Eqbal", "No name.", "Diego"]
```

### 13. Sử dụng `_` trong irb hoặc rails console
Đôi khi bạn gọi 1 câu truy vấn mà quên đặt tên biến cho nó, ví dụ:
```ruby
User.find_by name: "user_name"
```
Bình thường thì mình sẽ bấm mũi tên lên, sau đó gõ `Ctrl + A` để về đầu dòng, đặt tên biến `u = ` rồi enter. Tuy nhiên, trong ruby console và rails consle đã hỗ trợ sẵn dấu `_` sẽ lưu lại giá trị trả về cuối cùng. bạn chỉ cần dùng `_` để thao tác là được. (ở đây `_` sẽ tương đương với việc gọi `u`)

# Kết luận
Bài viết không mới, tuy nhiên mình vẫn hy vọng bạn sẽ tìm được 1 tip thú vị :v
Thank you for reading :D

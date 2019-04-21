---
layout: post
title: Ruby Hash Looking up, why is it so fast?
tags:
- Ruby
---

## 1. Mở đầu
Trong Ruby, Hash là 1 tập hợp các cặp key-value. Và nếu bạn test thử với việc truy vấn Hash, bạn sẽ thấy Ruby trả về kết quả rất nhanh. Nhưng tại sao nó lại làm được như vậy?

Để hiểu được logic đằng sau việc implement Hash trong Ruby, chúc ta hãy thử tưởng tượng Ruby vẫn chưa implement Hash, và nhiệm vụ của chúng ta là dùng chính ngôn ngữ Ruby để xây dựng kiểu lưu trữ key-value này. So, let's go ;)

## 2. Implement Hash
Đầu tiên, chúng ta sẽ cần tạo 1 `Struct` để lưu trữ cho `HashEntry`. Nó đơn giản chỉ là 1 object chứa key-value mà chúng ta sẽ add vào trong hashes tổng.

```ruby
HashEntry = Struct.new :key, :value
```

Như vậy, chúng ta đã có chỗ để lưu 1 object key-value. Bây giờ, chúng ta cần 1 class mà có thể lưu trữ Hashes, tức là lưu trữ các objects key-value vừa tạo được ở bên trên, với điều kiện là sẽ phải xử lý được conflicts giữa những object có cùng key.

```ruby
class HashTable
	attr_accessor :bins

	def initialize
		self.bins = []
	end
end
```

Chúng ta đã thêm `bins` attribute cho class này. `bins` là 1 mảng nơi ta sẽ lưu trữ các phần tử HashEntry.

Bây giờ, hãy cùng viết 1 method để add các HashEntry vào trong HashTable. Để làm giống với những gì mà Ruby vẫn cung cấp, ta sẽ implement method `<<` để thêm phần tử vào :D

```ruby
class HashTable
	attr_accessor :bins

	def initialize
		self.bins = []
	end

	def <<(entry)
		self.bins << entry
	end
end
```

Ngon, thế là từ giờ chúng ta có thể add các phần tử HashEntry vào trogn HashTable bằng cách:

```ruby
entry = HashEntry.new :foo, :bar
table = HashTable.new
table << entry
```

Bây giờ, để truy vấn 1 entry bằng key, ta sẽ tìm kiếm bằng cách:

```ruby
def [](key)
	self.bins.detect {|entry| entry.key == key}
end
```

Nghe có vẻ đơn giản đúng không :v Để lấy ra 1 cặp key value, chúng ta chỉ cần duyệt mảng `bins` để tìm entry có key giống với key chúng ta muốn tìm. Nhưng đơn giản là 1 chuyện, còn phương pháp này có cho hiệu quả tốt không? Hãy tử check qua performance bằng Ruby benchmarking tools thử xem nhé :v

```ruby
require 'benchmark'

#
# HashTable instance
#
table = HashTable.new

#
# CREATE 1,000,000 entries and add them to the table
#

(1..1000000).each do |i|
  entry = HashEntry.new i.to_s, "bar#{i}"

  table << entry
end

#
# Look for an element at the beginning, middle and end of the HashTable.
# Benchmark it
#
%w(100000 500000 900000).each do |key|
  time = Benchmark.realtime do
    table[key]
  end

  puts "Finding #{key} took #{time * 1000} ms"
end
```

Khi chạy thử đoạn code này, chúng ta nhận được kết quả như sau:

```
Finding 100000 took 33.641 ms
Finding 500000 took 192.678 ms
Finding 900000 took 345.329 ms
```

Chúng ta có thể thấy rõ rằng, khi add càng nhiều phần tử HashEntry vào trong HashTable, thời gian tìm kiếm sẽ càng ngày càng tăng lên. Và tất nhiên, kết quả như trên là không thể chấp nhận được khi áp dụng vào các case thực tế :D

Bây giờ, chúng ta thử xem Ruby đã implement theo 1 hướng khác để giải quyết vấn đề này ntn? :v Thay vì sử dụng single array để lưu trữ tất cả các entries, Ruby sẽ dùng mảng 2 chiều để lưu trữ :D

Đầu tiên, ta sẽ tính ra 1 unique integer cho mỗi entry. Sau đó, ta sẽ chia số integer vừa tìm được cho tổng số các phần tử mà `bins` có thể chứa => Lấy được 1 số dư. Số dư này sẽ được sử dụng làm `index` cho entry đó trong `bins`.

Khi ta tìm kiếm 1 entry trong hashes bằng key, ta sẽ tính toán ra `index` của entry bằng thuật toán bên trên, sau đó chỉ cần truy vấn các entries tại index đó trong `bins` là sẽ lấy được cặp key-value cần tìm.

Có vẻ hơi khó hiểu? :v hãy thử xem qua đoạn code dưới đây nhé:

```ruby
class HashTable
  attr_accessor :bin_count, :bins

  def initialize
    self.bin_count = 500
    self.bins = []
  end

  def bin_for(key)
    key.hash % self.bin_count
  end

  def <<(entry)
    index = bin_for(entry.key)
    self.bins[index] ||= []
    self.bins[index] << entry
  end

  def [](key)
	  index = bin_for(key)
	  self.bins[index].detect do |entry|
	    entry.key == key
	  end
	end
end
```

Method `bin_for` để tính để tính ra được `index` của entry mới sẽ được lưu ở trong `bins`. Còn khi lưu vào và lấy ra entry, chúng ta sẽ sử dụng hàm `bin_for` này để tính ra index, sau đó add/truy vấn dữ liệu như cách thông thường mình đã làm ở trên.

Chạy thử benchmark cho đoạn code trên, ta thấy thời gian tìm kiếm đã được cải thiện đáng kể:

```
Finding 100000 took 0.025 ms
Finding 500000 took 0.094 ms
Finding 900000 took 0.112 ms
```

Tuy nhiên, ta vẫn có thể cải thiện kết quả trên thêm nữa, bằng cách tăng số `bin_count` lên. Như thế mảng bins sẽ có thể lưu trữ nhiều hơn, các phần tử có trùng `index` trong bins sẽ ít đi => Khi lấy ra entry từ key, ta sẽ không phải duyệt nhiều hơn (`self.bins[index].detect {|entry| entry.key == key}`)

```ruby
class HashTable
  # ...

  def initialize
    self.bin_count = 300000
    self.bins = []
  end

  # ...
end
```

Khi tăng `bin_count` lên 300000, ta thử chạy lại benchmark, chúng ta thu đươcj kết quả:

```
Finding 100000 took 0.014 ms
Finding 500000 took 0.016 ms
Finding 900000 took 0.005 ms
```

`bin_count` càng lớn thì thời gian truy vấn càng được cải thiện, tuy nhiên, trên thực tế thì Ruby sử dụng `bin_count` là bao nhiêu?

Ruby quản lý động size của `bins`. Nó bắt đầu từ 11 và ngay khi mảng `bins` có nhiều hơn 5 phần tử, bin size sẽ tự động tăng lên và tất cả các hash elements sẽ được chuyển chỗ sang mảng `bins` có size lớn mới. Làm như thế này, Ruby sẽ hạn chế được thời gian tìm kiếm và hạn chế sự lãng phí memory.

## 3. Kết luận
Nếu như bạn muốn tìm hiểu thêm về các thuật toán mà Ruby đã sử dụng để implement các cấu trúc và hiểu sâu hơn về Ruby, hãy thử đọc qua cuốn sách [Ruby Under a Microscope](http://patshaughnessy.net/ruby-under-a-microscope). Cuốn sách giải thích các thuật toán ở 1 góc nhìn mới, đơn giản hơn, và ai cũng có thể đọc được.

Thuật toán bên trên là thế, tuy nhiên, trên thực thế, code của Hash Ruby được implement bằng C, tuy nhiên, thuật toán vẫn là như thế =]]

Bạn cũng có thể ngó qua source code của Ruby Hash tại [đây](https://github.com/rubinius/rubinius/blob/master/core/hash.rb)

Cám ơn các bạn đã đọc bài viết :D

References: [Hash lookup in Ruby, why is it so fast?](https://www.engineyard.com/blog/hash-lookup-in-ruby-why-is-it-so-fast)

---
layout: post
title: Guide to Rails Caching.
tags:
- Rails
- Caching
---

# Mở đầu
Từ khi mới bắt đầu vào học làm web, mình đã được các đàn anh đi trước nói về mấy vấn đề lớn mà bất cứ một web backend developer nào cũng sẽ có lúc gặp phải. Đó là: Search, Cache và Load Balancing. Trong 1 bài viết trước, mình có nói về 1 số biện pháp cache (HTTP cache), bạn có thể đọc ở [đây](https://ttuan.github.io/2017/10/15/Optimize-your-Rails-applicaiton/).

Hôm nay, mình sẽ giới thiệu 1 vài điều cơ bản về Application Cache, Cache Store và so sánh Benchmark của từng loại.

Bài viết phần lớn được dịch từ bài viết tại [speedshop.co](https://www.speedshop.co/2015/07/15/the-complete-guide-to-rails-caching.html)


# Rails Caching Overview
Nếu bạn chưa có chút khái niệm nào về Rails Caching, hãy thử đọc qua bài viết [Caching with Rails: An Overview](http://guides.rubyonrails.org/caching_with_rails.html) nhé. Đây là bài viết khá ngắn gọn nhưng sẽ cho bạn cái nhìn tổng quan về Rails Caching.

## Benefits of Caching
Tại sao chúng ta cần dùng cache? Câu trả lời rất đơn giản, đó là Tốc độ.

Ruby là một ngôn ngữ có tốc độ khá chậm (so với mặt bằng chung của các ngôn ngữ khác như C, Java, Go, ..). Giải pháp đặt ra đó là: Chúng ta cần *thực hiện ít nhất có thể các xử lý Ruby trong mỗi request* =)) Cách đơn giản nhất đó là Caching. Chỉ cần thực hiện công việc 1 lần, cache lại kết quả, trả về kết quả cache này cho các request lần sau.

Nhưng, nhanh như thế nào mới đủ?

Theo như bài viết ở [đây](http://theixdlibrary.com/pdf/Miller1968.pdf), phần lớn người dùng cảm thấy thoải mái với ngưỡng < 1s (Không phải response time 1s, mà là 1s kể từ lúc user click hoặc interacte đến lúc DOM được vẽ xong).

Hãy thử cùng phân tích xem trong 1s kia, chúng ta cần phải thực hiện những gì?

* **50ms** cho việc chạy ngầm của network.
* **150ms** cho loading JS and CSS resources. (dùng để vẽ khung của trang web)
* Ít nhất **250ms** cho việc thực hiện code JS mà chúng ta vừa download. (Khoảng thời gian này có thể lớn hơn nếu như code JS của bạn có nhiều function cần thực hiện ngay lúc load DOM).

Như vậy, chúng ta chỉ còn **~500ms**. Vậy để đạt được mốc 1s, thời gian phản hồi của server nên nằm trong khoảng **200 - 300ms**. Đây cũng là mức mà trang [Google Speed Insight](https://developers.google.com/speed/docs/insights/Server) gợi ý.

300ms cho mỗi request, chúng ta vẫn có thể thực hiện được mà không cần dùng Rails caching. Đặc biệt, nếu bạn chú ý tối ưu các câu queries SQL/ sử dụng ActiveRecord 1 cách thuần thục, có thể thời gian phản hồi của server còn nhỏ hơn. Tuy nhiên, dùng caching sẽ đơn giản, dễ dàng hơn rất nhiều.

OK. Bạn đã sẵn sàng để bắt đầu "Caching". Nhưng, chúng ta cần phải Cache cái gì? :3

## What need to be cached?
Thay vì việc lần mò xem trong app của mình có chỗ nào hiệu năng chưa tốt, chỗ nào cần query nhiều, .. chúng ta có thể dùng tool để detect phần nào của web đang bị chậm. Mình khuyến khích nên dùng gem [rack-mini-profiler](https://github.com/MiniProfiler/rack-mini-profiler). Gem này sẽ giúp bạn xác định thời gian server xử lý từng công việc một, làm thế nào để có thể in ra được 1 trang web.

Nếu bạn khônga muốn dùng gem, bạn cũng có thể theo dõi bằng log của Rails

![](https://i.imgur.com/wTHHYbr.png)

Như trong hình trên, chúng ta thấy Rails show ra

>Completed 200 OK in 110ms (Views: 65.6ms | ActiveRecord: 19.7ms)

Tổng thời gian in ra view + thời gian query là 110ms. Có 2 điểm đáng lưu ý là:

+ ActiveRecord::Relations lazily loads data. Giả sử trong controller, bạn để đoạn code `@users = User.all` và không xử lý gì với biến `@users` này. Trong views, khi bạn show ra thông tin của từng user: `@users.each do ...`, lúc này thì `@users` mới bắt đầu được query vào DB để lấy dữ liệu. và **thời gian query này sẽ được tính vào tổng thời gian của Views**.
+ Thời gian được tổng hợp ở ActiveRecord **không** phải là tổng thời gian execute code Ruby trong ActiveRecord (build queries, executing query, turning query results into ActiveRecord objects), nó **chỉ bao gồm thời gian query trong DB**

**NOTE:** Khi bạn test Rails app performance, hãy tạo 1 môi trường mà có `RAILS_ENV=production`. Việc chạy trên production mode sẽ giúp bạn kiểm chứng về mặt tốc độ dưới con mắt của end users, đồng thời disable việc reload code và compile assets. Cách tốt nhất là bạn hãy dùng Docker, build 1 môi trường giống như trên production để trải nghiệm.


# Caching techniques

## 1. Key-based cache expiration
Việc ghi và đọc từ cache khá đơn giản. Nếu bạn chưa biết những kiến thức cơ bản về nó, hãy đọc qua [guide](http://guides.rubyonrails.org/caching_with_rails.html) này. **Phần phức tạp nhất của cache là biết khi nào expire caches.**

Ý tưởng: Lưu trữ thông tin dưới dạng Hash, trong đó cache key chứa thông tin về giá trị được cached. Khi 1 object thay đổi, cache key cho object đó cũng sẽ bị thay đổi => cache cũ đã bị expired.

Trong ActiveRecord, bất cứ khi nào chúng ta thay đổi 1 attribute và save vào DB, `updated_at` attribute cũng thay đổi. Nên chúng ta có thể sử dụng `updated_at` trong cache keys khi chúng ta caching ActiveRecord object. Đó chính là cách mà Rails đã thực hiện:

Ví dụ:

```ruby
<% todo = Todo.first %>
<% cache(todo) do %>
  ... a whole lot of work here ...
<% end %>
```
Khi chúng ta viết như trên, Rails sẽ tạo ra 1 cache key có dạng:

>views/todos/123-20170806214154/7a1156131a6928cb0026877f8b749ac9

trong đó: `todos` là class của object được cache. `123` là id của object, `20170806214154` là `updated_at` attribute của object. Và phần còn lại được gọi là `template tree digest`. Đây là một đoạn mã hash md5, là tên của file chứa đoạn thông tin vừa được cache.

Khi ta thay đổi bất cứ thứ gì trong cache key thì sẽ expires the cache:

+ Class của object thay đổi.
+ Object id thay đổi.
+ Trường `updated_at` của object thay đổi.
+ Template thay đổi. (file cache bị thay đổi nội dung/ ...)

Bạn có thể nhận ra rằng, technique này không thực sự expire các cache keys, nó chỉ **không sử dụng** tới các cache cũ.

Ngoài ra, bạn cũng có thể truyền 1 mảng vào trong cache. Cache key sẽ dựa trên version của các phần tử trong array đó. Ví dụ như:

```ruby
<% todo = Todo.first %>
<% cache([current_user, todo]) do %>
  ... a whole lot of work here ...
<% end %>
```
Bất cứ khi nào `current_user` được update hoặc `todo` thay đổi, cache key sẽ bị expire và thay thế.

Vậy khi nào thì những đoạn cache kia sẽ bị xoá? Rails có cũng cấp thêm 1 vài option khi config cache_store. Bạn có thể xem ở [đây](http://guides.rubyonrails.org/v4.1/caching_with_rails.html#activesupport-cache-store). Trong đó có set `expires_in`. Sau thời gian `expires_in` được cung cấp, cache entries sẽ được tự động remove đi. ^^

## 2. Russian Doll Caching
Russian Doll - Búp bê Nga là loại búp bê mà 1 con búp bê sẽ chứa 1 con nhỏ hơn ở bên trong. Russian doll caching cũng giống như vậy. Chúng ta sẽ stack các đoạn cache fragments bên trong 1 đoạn cache khác. Ví dụ như chúng ta có đoạn in ra list các Todo như sau:

```ruby
<% cache('todo_list') do %>
  <ul>
    <% @todos.each do |todo| %>
      <% cache(todo) do %>
        <li class="todo"><%= todo.description %></li>
      <% end %>
    <% end %>
  </ul>
<% end %>
```
Nhìn đoạn code trên thì có vẻ khá ổn. Tuy nhiên, nó sẽ có vấn đề nếu như chúng ta thay đổi description của 1 todo có sẵn. Ví dụ từ "description 1" thành "description 2". Khi ta reload lại page, todo list vẫn hiển thị "description 1" vì: mặc dù đoạn cache bên trong đã bị thay đổi nhưng đoạn cache ngoài (đoạn cache todo list) thì không.

=> Nếu chúng ta muốn tái sử dụng đoạn cache fragment bên trong, chúng ta cũng sẽ phải renew đoạn code bên ngoài nếu đoạn code bên trong thay đổi.

Russian doll caching vẫn sử dụng key-based cache expiration để giải quyết vấn đề này. Mục đích cần làm là: Khi đoạn cache bên trong bị hết hạn, chúng ta sẽ expire đoạn code bên ngoài. Còn nếu như đoạn code bên ngoài bị expire, chúng ta KHÔNG muốn đoạn code bên trong bị expire.

```ruby
<% cache(["todo_list", @todos.map(&:id), @todos.maximum(:updated_at)]) do %>
  <ul>
    <% @todos.each do |todo| %>
      <% cache(todo) do %>
        <li class="todo"><%= todo.description %></li>
      <% end %>
    <% end %>
  </ul>
<% end %>
```

Bây giờ. Nếu bất cứ 1 @todo nào thay đổi, `@todos.maximum(:updated_all)` sẽ thay đổi. Hoặc nếu có 1 Todo bị xoá hoặc thêm vào `@todos`, đoạn `@todos.map(&:id)` cũng sẽ thay đổi => đoạn code bên ngoài bị expire. Tuy nhiên, bất cứ todo items nào bên trong không thay đổi, chúng ta vẫn có thể tái sử dụng.
Rails có cung cấp cho bạn 1 option khác, đó là sử dụng `touch` option trong ActiveRecord associations. Khi 1 object gọi hàm `touch()`, no sẽ update trường `updated_at` trong DB.

```ruby
class Corporation < ActiveRecord::Base
  has_many :cars
end

class Car < ActiveRecord::Base
  belongs_to :corporation, touch: true
end

class Brake < ActiveRecord::Base
  belongs_to :car, touch: true
end

@brake = Brake.first

# calls the touch method on @brake, @brake.car, and @brake.car.corporation.
# @brake.updated_at, @brake.car.updated_at and @brake.car.corporation.updated_at
# will all be equal.
@brake.touch

# changes updated_at on @brake and saves as usual.
# @brake.car and @brake.car.corporation get "touch"ed just like above.
@brake.save

@brake.car.touch # @brake is not touched. @brake.car.corporation is touched.
```
Chúng ta có thể sử dụng behavior này kết hợp với Russian Doll caches:

```ruby
<% cache @brake.car.corporation %>
  Corporation: <%= @brake.car.corporation.name %>
  <% cache @brake.car %>
    Car: <%= @brake.car.name %>
    <% cache @brake %>
      Brake system: <%= @brake.name %>
    <% end %>
  <% end %>
<% end %>
```
Với đoạn code này, kết hợp cùng `touch` relationships config bên trên, mỗi khi `@brake` thay đổi, đoạn cache bên ngoài cũng sẽ bị expired (vì trường `updated_at` của car và corporation đã bị thay đổi). Tuy nhiên, nếu `car` và `corporation` thay đổi, đoạn cache cho `brake` bên trong sẽ vẫn có thể tái sử dụng.

# Which cache backend should I use?
Rails developers có khá nhiều lựa chọn cho cache backend:

* **ActiveSupport::FileStore** Mặc định. Nếu bạn chọn cache backend này, các đoạn cache sẽ được lưu trữ trong filesystem.
* **ActiveSupport::MemoryStore** Config này cho phép bạn đặt các đoạn cache vào 1 thread-safe Hash và lưu trữ trên RAM.
* **Memcache and dalli** `dalli` là Memcache cache stores client khá phổ biến. Memcache được phát triển cho LiveJournal năm 2003 và được design cho web app.
* **Redis and redis-store** `redis-store` là một client phổ biến nếu bạn sử dụng Redis để cache.
* **LRURedux** là 1 memory-based cache store. Khá giống với `ActiveSupport::MemoryStore`, nhưng nó được thiết kế trên cơ chế cải thiện hiệu năng bởi Sam Saffron, co-founder của Discourse.

Bây giờ, chúng ta sẽ cùng phân tích về Ưu điểm và Nhược điểm của từng loại cache. Cuối bài, chúng ta sẽ so sánh performance benchmarks để bạn cso thể lựa chọn cache backend phù hợp nhất.

## 1. ActiveSupport::FileStore
FileStore là default cache implementation cho Rails app. Nếu như bạn không set `config.cache_store` trong file `production.rb`, bạn sẽ mặc định sử dụng FileStore.

FileStore thường lưu trữ caches trong folder `tmp/cache`.

### **a. Advantages**
* **FileStore works across processes** Ví dụ, nếu bạn có 1 Rails app trên Heroku, web server là Unicorn, đồng thời có 3 Unicorn workers thì mỗi worker đều có thể chia sẻ cache. Nếu worker 1 tính toán và lưu trữ `@todos` cache, worker 2 có thể sử dụng đoạn cache này. Tuy nhiên, điều này KHÔNG đúng với dạng `across hosts`.
* **Disk space is cheaper than RAM** Hosted Memcache servers không hề rẻ. Ví dụ, 30MB Memcache server sẽ không vấn đề gì. Nhưng nếu 5GB cache thì sao? nó sẽ có giá $290/tháng.  Nếu sử dụng disk space, nó sẽ tiết kiệm cho bạn rất nhiều.

### **b. Disadvantages**
* **Filesystems are slow** Truy cập vào disk thì luôn chậm hơn truy cập vào RAM. Tuy nhiên, nó có thể nhanh hơn việc truy cập cache ở 1 nơi khác qua network.
* **Cache can't be shared across hosts** Bạn sẽ không thể share cache với Rails server nếu server đó không được quyền truy cập vào filesystem. Điều này sẽ gây khó khăn cho việc large deployments.
* **Not an LRU cache** Đây là hạn chế lớn nhất của FileStore. FileStore sẽ expire các file cache dựa trên **thời gian chúng được tạo ra, không phải theo thời gian cuối cùng chúng được sử dụng**. Khi chúng ta sử dụng key-based expiration, dung lượng các file cache của ta sẽ tăng lên nhanh chóng (vì mỗi lần update, tạo mới `@todo` sẽ tạo ra 1 file cache mới). Đến khi dung lượng các file cache đạt tới độ giới hạn (giả sử chúng ta config là 1GB), nó sẽ bắt đầu expire các đoạn cache **dựa trên thời gian chúng được tạo**. Tức là nếu mình có 1 đoạn cache, được tạo ra đầu tiên, được truy cập 10 lần/giây, FileStore vẫn sẽ expired item này đầu tiên. ^^. Least-Recently-Used cache là 1 thuật toán cho phép expire những file cache ít sử dụng. => Sẽ tốt hơn khi kết hợp cùng Key-based cache expiration.
* **Crashes Heroku dynos** Đây cũng là 1 nhược điểm khá lớn của FileStore. Trên Heroku, việc truy cập cache từ filesystem sẽ khá chậm. Nếu có quá nhiều file cache, Rails app sẽ mất hàng giờ để load files. Thêm vào đó, Heroku restart các dynes 24h 1 lần -> filesystem bị reset -> các file cache sẽ bị xoá.

### **c. Khi nào thì nên sử dụng ActiveSupport::FileStore**
Hãy dùng FIleStore nếu bạn có ít request load (1 hoặc 2 servers) và vẫn cần *very large cache* (>100MB). Ít nhất, không dùng nó trên Heroku.

## 2. ActiveSupport::MemoryStore
MemoryStore cũng được Rails cung cấp sẵn. Thay vì lưu trữ cached value trong filesytem, MemoryStore lưu trữ chúng ở trên RAM, dưới dạng 1 big Hash.

ActiveSupport::MemoryStore, cũng giống như các cache stores khác, là thread-safe.

### **a. Advantages**
* **It's fast** Do được truy xuất từ RAM nên tốc độ rất nhanh.
* **It's easy to set up** Việc cài đặt rất đơn giản, bạn chỉ cần thay đổi `config.cache_store` thành `:memory_store`.

### **b. Disadvantages**
* **Cache can't be shared across processes or hosts** Giống như FileStore, cache không thể share giữa các hosts, nhưng có thể share giữa các processes.
* **Cache add to your total RAM usage** Vì cache được lưu trữ trên RAM nên việc lưu data sẽ làm tăng lượng RAM Usage.

### **c. Khi nào thì nên sử dụng ActiveSupport::MemoryStore**
Nếu bạn có 1 hoặc 2 servers, với ít workers, và bạn lưu trữ cache data với dung lượng nhỏ (<20MB), MemoryStore sẽ khá phù hợp.

## 3. Memcache and dailli
Memcache là một external cache store được sử dụng + recommended nhiều nhất cho Rails apps. Memcache được phát triển cho LiveJournal vào năm 2003 và được sử dụng trên production cho các site như Wordpress.org, Wikipedia và Youtube.

### **a. Advantages**
* **Distributed, so all processes and hosts can share** Không giống như FileStore and MemoryStore, tất cả các processes và hosts có thể chia sẻ/ truy cập cache. Chúng ta có thể tận dụng tối đa lợi ích của cache bởi vì cache key chỉ cần viết 1 lần mà có thể truy cập từ mọi entire trong hệ thống.

### **b. Disadvantages**
* **Distributed caches are susceptible to network issues and latency** Điều này khá là hiển nhiên. Truy cập file cache over network có thể chậm hơn việc truy cập dữ liệu từ RAM hoặc trong filesystem.
* **Expnesive** Nếu bạn sử dụng FileStore hay MemoryStore, việc này diễn ra trên chính server của bạn -> free. Nếu muốn sử dụng Memcache, bạn cần setup trên 1 Memcache instance trên AWS hoặc 1 service khác như Memcachier.
* **Cache value are limited to 1MB** Thêm vào đó, cache keys giới hạn 250bytes.

### **c. Khi nào thì nên sử dụng Memcache**
Nếu bạn sử dụng nhiều hơn 2 hosts, bạn nên sử dụng distributed cache store. Tuy nhiên, mình nghĩ Redis là 1 lựa chọn tốt hơn. =))

## 4. Redis and redis-store
Redis, giống như Memcache, là 1 dạng lưu trữ dữ liệu trên memory, dưới dạng key-value. Redis được tạo ra từ năm 2009 bởi Salvatore Sanfilippo - leader and maintainer today.

Thêm vào đó, [redis-store](https://github.com/redis-store/redis-store) là 1 Redis cache gem trong block: [readthis](https://github.com/sorentwo/readthis). Nó vẫn đang trong giai đoạn phát triển nhưng được đánh giá là khá triển vọng.

### **a. Advantages**
* **Distributed, so all process and hosts can share** (Giống với Memcache)
* **Allow different eviction policies beyond LRU** Redis cho phép bạn lựa chọn action khi cache store bị đầy. Có thể tham khảo thêm tại [excellent Redis documentation](http://redis.io/topics/lru-cache)
* **Can persist to disk, allowing host restarts** Redis có thể ghi vào disk, không giống như Memcache. Điều này cho phép Redis write DB vào disk, restart mà vẫn giữ được những thứ vừa cache. (awesome)

### **b. Disadvantages**
* **Distributed caches are susceptible to network issues and latency** (Giống với Memcache)
* **Expensive** Bạn sẽ phải trả phí, setup Redis instance on AWS hoặc 1 service như Redis.
* **While Redis supports several data types, redis-store only supports Strings** Đây là 1 điểm hạn chế rất lớn của `redis-store` gem so với Redis. Redis hỗ trợ một vài loại dữ liệu phổ biến như: Lists, Sét, Hashes. Memcache, cũng chỉ có thể lưu trữ Strings. Nếu chúng ta muốn sử dụng các kiểu dữ liệu khác giống như Redis, chúng ta sẽ phải thay đổi rất nhiều marshaling/ serialization trong gem.

### **c. Khi nào thì nên sử dụng Redis**
Nếu bạn đang chạy nhiều hơn 2 servers hoặc processes, Redis là 1 sự lựa chọn tốt.

## 5. LRURedux
Được phát triển bởi Sam Safron của Discourse, LRURedux là một bản nâng cấp, tối ưu của ActiveSupport::MemoryStore. Tuy nhiên, nó không còn cung cấp 1 ActiveSupport-compatible interface nữa nên khi có thể bạn sẽ bị tắc khi sử dụng ở mức low-level trong app, nó không còn là default Rails cache store nữa rồi.

### **a. Advantages**
* **Ridiculously fast** LRURedux có 1 tốc độ truy cập nhanh nhất trong các loại cache store được giới thiệu ở đây.

### **b. Disadvantages**
* **Caches can't be shared across processes or hosts** Nó không thể được share giữa các hosts cũng như các processes. (vd như Unicorn workers hay Puma clusterd workers).
* **Caches add to your total RAM usage** Việc lưu trữ data trên memory sẽ làm tăng lượng sử dụng RAM.
* **Can't use it as a Rails cache store**  Tức là bạn sẽ không thể viết trực tiếp: `config.cache_store :lru_redux` trong file `production.rb` được, do nó không cung cấp ActiveSupport-compatible interface. Bạn chỉ có thể dùng nó ở mức low-level.

### **c. Khi nào thì nên sử dụng LRURedux?**
Bạn nên nghĩ tới việc sử dụng LRURedux khi app yêu cầu tốc độ cao. App có thể mở rộng, dữ liệu lớn,...


# Cache Benchmarks
Dưới đây là 1 số thống kê, đo benchmarks của các loại Cache. Code test được lưu trên [GitHub](https://gist.github.com/nateberkopec/14d6a2fb7fe5da06a1f6).

Chúng ta sử dụng method `fetch` để lấy cặp key-value. Nếu data đã tồn tại trong cache, ta sẽ đọc ra dữ liệu, nếu không, chúng ta viết block để execute. Benchmarking method này để test cả hiệu năng đọc/ghi. `i/s` là "iterations/second".

```
LruRedux::ThreadSafeCache:   337353.5 i/s
ActiveSupport::Cache::MemoryStore:    52808.1 i/s - 6.39x slower
ActiveSupport::Cache::FileStore:    12341.5 i/s - 27.33x slower
ActiveSupport::Cache::DalliStore:     6629.1 i/s - 50.89x slower
ActiveSupport::Cache::RedisStore:     6304.6 i/s - 53.51x slower
ActiveSupport::Cache::DalliStore at pub-memcache-13640.us-east-1-1.2.ec2.garantiadata.com:13640:       26.9 i/s - 12545.27x slower
ActiveSupport::Cache::RedisStore at pub-redis-11469.us-east-1-4.2.ec2.garantiadata.com:       25.8 i/s - 13062.87x slower
```

Từ kết quả trên, ta có thể thấy:

* LRURedux, MemoryStore và FileStore có tốc độ khá cao.
* Memcache và Redis vẫn có tốc độ cao nếu cache được đặt ở cùng host.
* Khi sử dụng host ở nơi khác và truy cập qua network, Memcache và Redis cần thời gian để load (~50ms/cache read). Khi chọn Memcache hay Redis host, hãy chọn cái gần nhất với server của bạn và benchmark hiệu năng của nó.

**NOTE: Khi sử dụng remote, distributed cache, hãy chắc chắn về thời gian cần thiết để đọc từ cache**. Bạn có thể sử dụng Benchmarking hoặc đọc từ Rails logs. Nhưng hãy chắc chắn rằng không cache bất cứ thứ gì mà thời gian đọc cache nhiều hơn thời gian để ghi nó.

# Kết luận
Bài viết trên có lẽ sẽ đem cho bạn 1 cái nhìn tổng quan về Cache trong Rails, đồng thời có so sánh 1 số Cache Store, để bạn có thể cân nhắc ưu, nhược điểm của từng loại và sử dụng cho phù hợp với Rails app của mình.

Hy vọng bài viết có thể giúp ích cho bạn. ^^
































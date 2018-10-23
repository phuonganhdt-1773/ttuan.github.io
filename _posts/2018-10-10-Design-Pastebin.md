---
layout: post
title: Design Pastebin.com (or Bit.ly)
tags:
- System-design
---

## 1. Mở đầu
Cho dù bạn có phải là người làm trong lĩnh vực công nghệ hay không, chắc hẳn bạn cũng ít nhất 1 lần sử dụng các trang rút ngắn link như [goog.gl](https://goo.gl/), [pastebin.com](https://pastebin.com/), [bit.ly](https://bitly.com), ... Với ý tưởng rất đơn giản: Rút gọn các đường link dài thành 1 đường link ngắn, giúp bạn gửi, lưu trữ, ... một cách dễ nhìn hơn :D

Vậy, nếu bạn đi phỏng vấn và gặp một câu hỏi là: Hãy thiết kế hệ thống cho trang `pastebin.com`, bạn sẽ làm như thế nào?

Một câu hỏi (giống như ban đầu mình nghĩ) thì khá là đơn giản, nhưng làm sao để thiết kế một cách tối ưu, để có thể đáp ứng được một lượng lớn active-user? Hãy cùng theo dõi bài viết nhé :D

## 2. Design

### Step 1: Outline use cases and constraints
Bước đầu tiên để thiết kế 1 hệ thống đó là ta cần thu thập các yêu cầu và phạm vi ảnh hưởng của các vấn đề. Sau đó đặt ra câu hỏi để phân loại các trường hợp và các ràng buộc.

#### Use cases
Chúng ta sẽ liệt kê ra 1 vài use cases như sau:

* User sẽ nhập vào 1 đoạn text và sẽ nhận lại được 1 link ngẫu nhiên.
	* Thời gian tồn tại của link:
		+ Setting mặc định là không hết hạn.
		+ Có thể set thời gian tồn tại của 1 đường link.
* User nhập vào 1 url đã rút gọn của hệ thống và nhận lại được contents.
* User dùng ẩn danh.
* Service để tracks analytics của trang.
	+ Check hàng tháng xem có bao nhiêu lượt views, sử dụng, ...
* Service để xoá các đường link đã hết hạn.
* Service có độ khả dụng cao.


##### Out of scope

* User đăng ký 1 account mới
	* User xác thực email.
* User đăng nhập vào account
 	* User chỉnh sửa documents
* User có thể chỉnh xem link có hiển thị hay không
* User có thể set shortlink thành một từ nào đó.

#### Ràng buộc và giả định

#### Giả định
* Lưu lượng traffic được phân phối như nhau.
* Khi click vào short link thì có thể vào được link nhanh chóng.
* Nội dung paste vào chỉ là text.
* Page view analytics không cần phải realtime.
* 10 triệu người dùng.
* 10 triệu bản ghi paste được ghi hàng tháng.
* 100 triệu bản ghi paste được đọc hàng tháng.
* 10:1 là tỉ lệ read/write.

#### Tính toán :v

* Size per paste: Dung lượng của mỗi bản ghi paste
	+ 1KB content cho mỗi bản ghi paste.
	+ `shortlink` - 7 bytes
	+ `expiration_length_in_minutes` - 4 bytes
	+ `created_at` - 5 bytes
	+ `paste_path` - 255 bytes
	+ total = ~1,27KB
* 12,7 GB là dung lượng các bản ghi mới được thêm vào mỗi tháng.
	+ 1,27 KB per paste * 10 triệu paste mỗi tháng.
	+ ~ 450 GB trong 3 năm
	+ 360 triệu shortlinks trong 3 năm
	+ Giả sử phần lớn là các new pastes, thay vì người dùng update pastes cũ.

* Trung bình 4 pastes được thêm vào mỗi giây.
* Trung bình 40 pastes được đọc mỗi giây.

Bonus:
* 2.5 triệu giây mỗi tháng
* 1 request/s = 2.5 triệu requests/month
* 40 request/s = 100 triệu requests/month
* 400 reqeust/s = 1 tỷ request/month

### Step 2: Create a high level design
Tạo 1 sơ đồ với các components quan trọng :D
![](https://camo.githubusercontent.com/2e0371e591b8311e36f5f5fa6ae18711f252b1f8/687474703a2f2f692e696d6775722e636f6d2f424b73426e6d472e706e67)

### Step 3: Design core components
Thiết kế chi tiết cho các core components

#### Use case: User nhập vào 1 block text và nhận về 1 random link.

Chúng ta có thể tạo 1 DB như hash table lớn, map url được sinh ra với 1 file trên server.

Thay vì việc quản lý file trên server, chúng ta có thể sử dụng `Object Store` như Amazon S3 hoặc 1 [NoSQL document store](https://github.com/donnemartin/system-design-primer#document-store)

Hoặc 1 giải pháp khác với relation DB, chúngt a có thể sử dụng [NoSQL key-value store](https://github.com/donnemartin/system-design-primer#key-value-store). Để cân nhắc, ta có thể tham khảo qua bài viết [Tradeoffs between choosing SQL or NoSQL](https://github.com/donnemartin/system-design-primer#sql-or-nosql).

* Client gửi 1 paste request tới Web Server, chạy như 1 reverse proxy.
* Web Server chuyển tiếp request tới Write API server.
* Write API server sẽ làm các nhiệm vụ sau:
	+ Generate an unique url
		- Check xem url đó có unique không bằng việc search trong DB.
		- Nếu như url không unique thì sẽ sinh 1 url mới.
		- Nếu chúng ta support custom url, chúng ta sẽ sử dụng luôn url người dùng cung cấp (sau khi check duplicate)
	+ Lưu vào SQL database `pastes` table.
	+ Lưu nội dung của bản paste đó lên Object Store
	+ Trả về url.

Bảng `pastes` sẽ có cấu trúc dạng

```
shortlink char(7) NOT NULL
expiration_length_in_minutes int NOT NULL
created_at datetime NOT NULL
paste_path varchar(255) NOT NULL
PRIMARY KEY(shortlink)
```

Chúng ta sẽ đánh index cho trường `shortlink` và `created_at` để tăng tốc độ tìm kiếm. (log-time thay vì scan cả bảng) và giữ data trong memory. Đọc 1 MB từ memory mất khoảng 250 microseconds, trong khi đọc từ SSD mất 4x và từ ổ đĩa mất 80x thời gian so với từ memory.

Để sinh ra unique url, chúng ta có thể dùng:

* Dùng thuật toán [MD5](https://en.wikipedia.org/wiki/MD5) hash từ ip của user + timestamp
	+ MD5 là 1 hash function khá phổ biến, dùng để sinh ra 128-bit hash value.
	+ MD5 là uniformly distributed.
	+ Chúng ta có thể dùng MD5 hash cho 1 dữ liệu sinh ngẫu nhiên.

* [Base 62](https://www.kerstner.at/2012/07/shortening-strings-using-base-62-encoding/) để encode MD5 hash.
	+ Base 62 có thể encodes `[a-zA-Z0-9]` nên có thể encode được urls.
	+ Chỉ sinh ra 1 hash result với 1 đầu vào
	+ Base 64 cũng là 1 bộ encode khá phổ biến, nhưng nó gặp vấn đề với urls vì có thêm dấu `+` và dấu `/`

```python
def base_encode(num, base=62):
    digits = []
    while num > 0
      remainder = modulo(num, base)
      digits.push(remainder)
      num = divide(num, base)
    digits = digits.reverse
```

* Với 7 kí tự của output, chúng ta có thể sinh ra 62^7 values, nên có thể handle được 360 triệu shortlink trong 3 năm.

```python
url = base_encode(md5(ip_address+timestamp))[:URL_LENGTH]

```

Chúng ta có thể cung cấp REST API:

```bash
$ curl -X POST --data '{ "expiration_length_in_minutes": "60", \
    "paste_contents": "Hello World!" }' https://pastebin.com/api/v1/paste
```

Response:

```
{
    "shortlink": "foobar"
}
```

#### Use case: User nhập vào 1 paste url và xem nội dung của paste

* Client gửi 1 paste request tới Web Server
* Web Server chuyển request đó tới Read API server
* Read API server sẽ thực hiện:
	+ Check SQL Database để tìm url.
		- Nếu url có tồn tại, lấy nội dung paste từ Object Store
		- Nếu k tồn tại, trả về lỗi.

REST API:

```
$ curl https://pastebin.com/api/v1/paste?shortlink=foobar

```

Response:

```
{
    "paste_contents": "Hello World"
    "created_at": "YYYY-MM-DD HH:MM:SS"
    "expiration_length_in_minutes": "60"
}
```

#### Use case: Service track analytics of pages

Vì chúng ta k cần thống kê realtime, chúng ta có thể sử dụng MapReduce trên Web Server logs để đếm

```python
class HitCounts(MRJob):

    def extract_url(self, line):
        """Extract the generated url from the log line."""
        ...

    def extract_year_month(self, line):
        """Return the year and month portions of the timestamp."""
        ...

    def mapper(self, _, line):
        """Parse each log line, extract and transform relevant lines.

        Emit key value pairs of the form:

        (2016-01, url0), 1
        (2016-01, url0), 1
        (2016-01, url1), 1
        """
        url = self.extract_url(line)
        period = self.extract_year_month(line)
        yield (period, url), 1

    def reducer(self, key, values):
        """Sum values for each key.

        (2016-01, url0), 2
        (2016-01, url1), 1
        """
        yield key, sum(values)
```

#### Use case: Service để xoá các pastes đã bị expired

Để xoá các pastes đã bị expired, chúng ta có thể tìm toàn bộ DB để xem paste nào đã bị hết hạn và xoá nó khỏi bảng.

### Step 4: Scale the design
Khi hệ thống được mở rộng, chúng ta sẽ cần bổ sung những thiết kế đó để đáp ứng được nhu cầu.

[](https://camo.githubusercontent.com/4aee2d26ebedc20e7fa07a2c30780e332fa29f2c/687474703a2f2f692e696d6775722e636f6d2f346564584730542e706e67)

Analytics DB, chúng ta có thể sử dụng Amazon Redshift hoặc GG BigQuery.

OBject Store như Amazon S3 có thể handle được 12.7 Gb new content mỗi tháng.

Để đáp ứng 40 requests read mỗi giây, chúng ta nên dùng Memory Cache thay vì DB. Memory Cache khá hữ ihcs trong việc handle các phân bố request không đồng đều. SQL Read Replicas có thể handle được trường hợp k có cache.

4 paste writes mỗi giây, chúng ta có thể sử dụng 1 SQL Write Master-Slave. hoặc cũng có thể sử dụng các SQL scaling patterns khác như:

* [Federation](https://github.com/donnemartin/system-design-primer#federation)
* [Sharding](https://github.com/donnemartin/system-design-primer#sharding)
* [Denormalization](https://github.com/donnemartin/system-design-primer#denormalization)
* [SQL Tuning](https://github.com/donnemartin/system-design-primer#sql-tuning)


## 3. Kết luận
Lúc đầu, nếu nhận được requirement thiết kế 1 web site tương tự pastebin, mình sẽ chỉ nghĩ là sẽ random ra 1 chuỗi url rồi trả về cho client, khá đơn giản. Tuy nhiên, để thiết kế 1 hệ thống tối ưu (chuỗi cần sinh có độ dài là bao nhiêu để vừa đủ với nhu cầu lưu trữ, quản lý nội dung ntn, ...), đáp ứng được lượng truy cập lớn thì mình lại chưa hề nghĩ tới.

Hy vọng hướng design như trong bài viết này sẽ có ích khi bạn thiết kế 1 hệ thống khác. :D

## Reference
Bài viết được dịch từ bài: https://github.com/donnemartin/system-design-primer/blob/master/solutions/system_design/pastebin/README.md

Các bài viết/ thảo luận bạn có thể tham khảo:

#### NoSQL
* [Key-value store](https://github.com/donnemartin/system-design-primer#key-value-store)
* [Document store](https://github.com/donnemartin/system-design-primer#document-store)
* [Wide column store](https://github.com/donnemartin/system-design-primer#wide-column-store)
* [Graph database](https://github.com/donnemartin/system-design-primer#graph-database)
* [SQL vs NoSQL](https://github.com/donnemartin/system-design-primer#sql-or-nosql)

### Caching

* Where to cache
    * [Client caching](https://github.com/donnemartin/system-design-primer#client-caching)
    * [CDN caching](https://github.com/donnemartin/system-design-primer#cdn-caching)
    * [Web server caching](https://github.com/donnemartin/system-design-primer#web-server-caching)
    * [Database caching](https://github.com/donnemartin/system-design-primer#database-caching)
    * [Application caching](https://github.com/donnemartin/system-design-primer#application-caching)
* What to cache
    * [Caching at the database query level](https://github.com/donnemartin/system-design-primer#caching-at-the-database-query-level)
    * [Caching at the object level](https://github.com/donnemartin/system-design-primer#caching-at-the-object-level)
* When to update the cache
    * [Cache-aside](https://github.com/donnemartin/system-design-primer#cache-aside)
    * [Write-through](https://github.com/donnemartin/system-design-primer#write-through)
    * [Write-behind (write-back)](https://github.com/donnemartin/system-design-primer#write-behind-write-back)
    * [Refresh ahead](https://github.com/donnemartin/system-design-primer#refresh-ahead)

### Asynchronism and microservices

* [Message queues](https://github.com/donnemartin/system-design-primer#message-queues)
* [Task queues](https://github.com/donnemartin/system-design-primer#task-queues)
* [Back pressure](https://github.com/donnemartin/system-design-primer#back-pressure)
* [Microservices](https://github.com/donnemartin/system-design-primer#microservices)

### Communications

* Discuss tradeoffs:
    * External communication with clients - [HTTP APIs following REST](https://github.com/donnemartin/system-design-primer#representational-state-transfer-rest)
    * Internal communications - [RPC](https://github.com/donnemartin/system-design-primer#remote-procedure-call-rpc)
* [Service discovery](https://github.com/donnemartin/system-design-primer#service-discovery)

### Security

Refer to the [security section](https://github.com/donnemartin/system-design-primer#security).

### Latency numbers

See [Latency numbers every programmer should know](https://github.com/donnemartin/system-design-primer#latency-numbers-every-programmer-should-know).

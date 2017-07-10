---
layout: post
comments: true
title: Tracking SQL query in Rails
tags:
- Become a Rubyist
---

# Tracking SQL queries in Rails

## Mở đầu
Khi bạn phát triển hoặc tối ưu Rails applications, một trong những công việc quan trọng đó là hiểu và tối ưu được các SQL queries vì phần lớn tốc độ web chậm là do các logic xử lý/ truy vấn DB chưa hợp lý. Chúng ta sẽ hỏi những câu hỏi như:
* Bao nhiêu câu SQL queries được gọi sau mỗi lần request?
* Mất bao nhiêu thời gian để hoàn thành một câu SQL query?
* Câu query có bị gọi lặp lại ở các nơi khác nhau trong code không?
* Câu query nào được gọi nhiều lần hơn các câu query khác?
...

và nhiều câu hỏi khác nữa. Thông thường, chúng ta thường tìm ra chúng bằng cách check file log, nhưng nếu chúng ta muốn phân tích chi tiết một request hoặc nếu chúng ta muốn track nhiều requests cùng 1 lúc, sẽ rất khó để chúng ta tìm và lọc ra các thông tin liên quan trong log file.

Chúng ta có thể sử dụng các tools như [rack-mini-profiler](https://github.com/MiniProfiler/rack-mini-profiler), nhưng khó khá khó sử dụng khi không render ra html page (như json, ajax response), hoặc khi chúng ta muốn lấy một kết quả tổng hợp từ nhiều requests với nhau.

Để trả lời cho các câu hỏi đó, [Steven Yue](https://github.com/steventen) đã tạo ra một gem tên là [sql_tracker](https://github.com/steventen/sql_tracker). Trong bài này, tôi sẽ chỉ cho các bạn cách hoạt động của gem `sql_tracker` và các tính năng của nó để giúp bạn track và phân tích các SQL queries.

## Tracking
Về cơ bản, `sql_tracker` sử dụng [instrumentation API](http://guides.rubyonrails.org/active_support_instrumentation.html) để track các SQL queries được gọi ra từ `Active Record` thông qua việc theo dõi `sql.active_record` hook:

```
ActiveSupport::Notifications.subscribe('sql.active_record') do |_name, _start, _finish, _id, payload|
 sql_query = payload[:sql]
end
```

Thay vì việc sử dụng một `block`, bạn có thể cung cấp một instance object, được cài đặt bởi `call` method:

```
class Handler
  def call(_name, _start, _finish, _id, payload)
    sql_query = payload[:sql]
  end
end

ActiveSupport::Notifications.subscribe('sql.active_record', Handler.new)
```

`sql_tracker` được khai báo và track các câu queries khi một Rails process chạy, và dừng việc tracking khi Rails process kết thúc.

## Filtering

Bạn có thể không muốn lắng nghe tất cả các câu queries. Ví dụ như bạn chỉ muốn track những câu `SELECT` của các queries, hoặc bạn muốn track các queries được gọi với một thư mục nhất định.

Mặc định, `sql_tracker` tracks và ghi lại tất cả 4 commands (SELECT, INSERT, UPDATE và DELETE), nhưng bạn có thể thay đổi bằng việc thay đổi biến settings `tracked_sql_command`

```
SqlTracker::Config.tracked_sql_command = %w(SELECT)
```
(Trong setting bên trên mình để viết hoa, bạn có thể sử dụng `select`)

Bạn cũng có thể thay đổi track paths/folders bằng việc sử dụng `tracked_paths` setting:

```
SqlTracker::Config.tracked_paths = %w(app/controllers/api)
```
Mặc định, `sql_tracker` tracks thư mục `app` và `lib` folders để bỏ qua các queries không liên quan (ví dụ như queries được gọi từ thư mục `tests`).

`sql_tracker` sử dụng Ruby [caller](http://apidock.com/ruby/Kernel/caller) method và Rail's `backtrace_cleaner` để thực hiện filering theo paths. `Caller` method trả về một mảng của các lệnh thực thi theo 1 stack và `backtrace_cleaner` giúp cho việc dọn dẹp stack sử dụng regex.

```
Rails.backtrace_cleaner.add_silencer do |line|
  line !~ %r{^(#{tracked_paths.join('|')})\/}
end

cleaned_trace = Rails.backtrace_cleaner.clean(caller)

# if cleaned_trace is empty, then the caller of this query is not included in tracked_paths
return false if cleaned_trace.empty?
```

## Grouping
`sql_tracker` sẽ thử đơn giản hoá và gọp các câu queries bằng việc thay thế các giá trị đặc biệt với `xxx`. Ví dụ, nếu bạn thực hiện một câu SQL query sau:
```
SELECT users.* FROM users WHERE users.id = 1
```
và sau đó là một câu query khác:
```
SELECT users.* FROM users WHERE users.id = 2
```
`sql_tracker` sẽ gộp chúng thành 1 câu query:
```
SELECT users.* FROM users WHERE users.id = xxx
```
`sql_tracker` sử dụng regular expressions để viết lại các câu SQL queries. Ví dụ như ở đây, nó đã tìm và thay thế các giá trị sau các operators so sánh như `=`, `<>`, `>`, `<` và cả `BETWEEN ... AND ...`

Sau khi đã nhóm lại, nó sẽ trở nên dễ dàng hơn để phân tích các câu queries, và tính toán tổng số query, thời gian trung bình, ...

## Storing
`sql_tracker` giữ tất cả các data trong bộ nhớ, và exports data trong 1 file JSON khi Rails process kết thúc. Tất cả các data chứa trong 1 `Hash`, format sẽ có dạng:
```
{
  key1: {
    sql: 'SELECT users.* FROM users ...',
    count: 1,
    duration: 0.34,
    source: ['apps/model/user.rb:57:in ...', ...]
  },
  key2: { ... },
  ... ...
}
```

Khi các keys là `md5` hoặc các câu sql queries đã được đơn giản hoá, và values là full câu sql query + một số thông tin thống kê.

Mặc định, file sẽ được lưu trong thư mục `tmp` trong Rails roots folder, bạn cũng có thể thay đổi bằng config:

```
SqlTracker::Config.output_path = File.join(Rails.root.to_s, 'my_folder')
```
Nếu bạn đã sử dụng app server `puma`, và set nhiều hơn 1 worker, bạn sẽ thấy nhiền hơn 1 JSON output file trong output folder vì mỗi một worker sẽ track và lưu data riêng biệt.

## Reporting
Cuối cùng, chúng ta có thể sinh ra report bằng cách sử dụng 1 hoặc nhiều JSON dump files:
```
sql_tracker tmp/sql_tracker-*.json
```
Report sẽ có dạng:

```
==================================
Total Unique SQL Queries: 24
==================================
Count | Avg Time (ms)   | SQL Query                                                                                                 | Source
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
8     | 0.33            | SELECT `users`.* FROM `users` WHERE `users`.`id` = xxx LIMIT 1                                            | app/controllers/users_controller.rb:125:in `create'
      |                 |                                                                                                           | app/controllers/projects_controller.rb:9:in `block in update'
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
4     | 0.27            | SELECT `projects`.* FROM `projects` WHERE `projects`.`user_id` = xxx AND `projects`.`id` = xxx LIMIT 1    | app/controllers/projects_controller.rb:4:in `update'
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
2     | 0.27            | UPDATE `projects` SET `updated_at` = xxx WHERE `projects`.`id` = xxx                                      | app/controllers/projects_controller.rb:9:in `block in update'
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
2     | 1.76            | SELECT projects.* FROM projects WHERE projects.priority BETWEEN xxx AND xxx ORDER BY created_at DESC      | app/controllers/projects_controller.rb:35:in `index'
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
... ...
```
Nó sẽ show cho bạn tổng số lần gọi của từng câu truy vấn, thời gian trung bình của từng câu query, và dòng code nào gọi câu query đó. Từ report này, bạn có thể hiểu được từng câu query đã được gọi nhưu thế nào, nơi nào gọi nó, ... Nếu các câu query đơn giản được gọi quá nhiều lần, hãy nghĩ tới giải pháp cache nó lại. Và nếu cùng 1 câu query bị lặp lại ở nhiều nơi khác nhau trong source code, đó là lúc cần refactor lại code.

Toi thích sử dụng `sql_tracker` để test các app của mình vì nó khá đơn giản, dễ dùng nhưng vừa đủ thông tin. Thông thường, tôi chọn một vài controller tests hoặc integration tests để chạy, và sau khi tests đã chạy xong, mở `sql_tracker tmp/sql_tracker-*.json` lần nữa để dumped data và check lại các queries.


## Kết luận

Bạn có thể xem code details tại [https://github.com/steventen/sql_tracker](https://github.com/steventen/sql_tracker). Tất nhiên, sẽ có rất nhiều thứ cần phải cải thiện. Tôi hy vọng bạn có thể sử dụng và hài lòng với gem này, mọi câu hỏi, các report lỗi và pull requests luôn được chào đón.

Bài viết được dịch từ: [http://stevenyue.com/blogs/tracking-sql-queries-in-rails/](http://stevenyue.com/blogs/tracking-sql-queries-in-rails/). Hy vọng qua bài viết này, các bạn có thể có thêm một cách mới để debug hoặc cải thiện lại chất lượng của Rails application của mình. :)

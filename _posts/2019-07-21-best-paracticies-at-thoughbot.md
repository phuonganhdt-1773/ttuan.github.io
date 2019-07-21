---
layout: post
title: Best practices at ThoughBot
tags:
- Rails
---

## Mở đầu
Nếu bạn là 1 Ruby on Rails dev, chắc hẳn bạn đã từng dùng qua gem [FactoryBot](https://github.com/thoughtbot/factory_bot) để tạo dữ liệu seed hoặc sinh dữ liệu khi test. Gem này ban đầu được các thành viên của [thoughbot](https://github.com/thoughtbot) viết ra. ThoughBot là một công ty rất nổi tiếng về làm ứng dụng (app/web/), có trụ sở ở Mỹ và Anh. Mình biết đến công ty này vì một số founder của công ty thường xuyên xuất hiện trong các buổi [Ruby Conferences](https://rubyconferences.org/) hay RailsConf.

Trong một lần lang thang, mình có tìm được [PlayBook](https://thoughtbot.com/playbook) của ThoughBot. (like Barney Stinson's PlayBook 😁). Trong này có nói về quy trình làm việc, lên ý tưởng, design, thăm dò ý kiến, coding, setup môi trường, .... tất cả các bước, việc phải làm để tạo ra 1 sản phẩm hoàn chỉnh. Trong cuốn PlayBook này, họ có nói về 1 số tip, tricks khi lập trình. Bài viết này được dịch lại từ post [Best Practices](https://github.com/thoughtbot/guides/blob/master/best-practices/README.md) chia sẻ 1 số kinh nghiệm của các kỹ sư làm việc tại ThoughBot, với 1 số quan điểm, mình sẽ nói thêm 1 vài điều về ý kiến cá nhân =]]
## Best Practices
### General
* **Không nên tin hoàn toàn vào 1 điều gì đó. Hãy cố gắng để hiểu chúng và luôn đặt câu hỏi khi bạn nghi ngờ** 👏 👏👏 - Điều này mình rất hay nhắc nhở các members trong team. Khi gặp 1 vấn đề, ngoài cách tìm hiểu nguyên nhân, giải pháp, thì việc hiểu được tại sao lại có giải pháp đó cũng rất quan trọng. Chỗ nào chưa hiểu thì cần **đặt câu hỏi**, mỗi người học hỏi nhau 1 chút thì sẽ nhanh tiến bộ/ nhớ lâu hơn rất nhiều. Đó là **học 1 cách chủ động**.
* **Không nên cố tình lờ/ỉm đi những exceptions** - Đây là 1 vấn đề rất hay gặp. Khi member code, họ là người trực tiếp làm việc với task đó, nên đôi khi họ sẽ phát hiện ra các exceptions, case dị mà người review sẽ không tìm được. Nhưng do không google được giải pháp (hoặc sợ mất thời gian) nên những exceptions này thường được ỉm đi :v Nếu người review hoặc QA tìm ra được, thì họ sẽ fix, còn không thì ... (yaoming). Chúng ta không nên như thế, khi gặp case dị, cần hỏi những người có kinh nghiệm để tìm giải pháp, như thế thì trình độ mình mới tăng lên được. Còn nếu không, đến lúc tìm ra bug, mình vẫn phải là người fix, nhưng fix lúc đó sẽ tốn efforts hơn rất nhiều. Đó là lý do mình luôn phải hỏi member khi gửi PullRequest là "Pull này e còn lăn tăn về vấn đề gì không?" 😁
* **Không viết những đoạn code xử lý dạng đoán function, tương lai sẽ dùng đến**: Ban đầu hãy xây dựng code thật đơn giản. Nếu sau này có spec rõ ràng hơn, refactor cũng chưa muộn, không nên đoán nội dung function.
* **Không duplicate các hàm của 1 build-in library**: Khi bạn có ý định viết 1 hàm custom nào đó, hãy thử tìm hiểu xem trong thư viện/framework đã có chưa, nếu đã có rồi thì dùng lại, k nên viết lại (vì hàm trong thư viện đã được tối ưu performance, test kĩ càng hơn nhiều).
* **Keep the code simple**: Tất nhiên, simple is the best :D

### Object-Oriented Design
* Không nên dùng biến toàn cục - Global Variables.
* Tránh sử dụng quá nhiều parametter khi truyền vào hàm.
* Giới hạn dependences của 1 object.
* Giới hạn các dependences mà object này phụ thuộc vào.
* Ưu tiên dùng composition hơn là kế thừa.
* Ưu tiên các methods nhỏ. (mỗi method chỉ nên từ 1 đến 5 dòng :v như thế việc đặt tên hàm sẽ dễ hơn, đọc logic cũng sẽ dễ hiểu hơn).
* Nên tạo ra các class với Single Responsibility (trong SOLID). Khi một class dài quá 100 dòng, có thể nó đang làm quá nhiều việc, nên chia nhỏ hơn để dễ test, dễ tái sử dụng.

### Ruby
* Tránh sử dụng parameters `optional`. Khi bạn cần sử dụng param này thì có thể method đang làm quá nhiều thứ.
* Tránh sử dụng `monkey-patching`. Điều này cũng không hoàn toàn đúng. Bạn có thể tham khảo bài viết [Monkey pathching without a mess](https://ttuan.github.io/2019/03/23/Monkey-patching-without-a-mess/) của mình :D
* Generate ra các [Binstubs](https://github.com/rbenv/rbenv/wiki/Understanding-binstubs) cho từng project, như `rspec` và `rake`, sau đó add vào version control.
* Nếu có các functions được sử dụng bởi nhiều models, nên tách nó từ class ra ngoài các models, sau đó tái sử dụng.

### Ruby Gems
* Định nghĩa tất cả các dependencies của 1 gem trong <PROJECT_NAME>.gemspec file.
* Reference file gemspec này vào trong `Gemfile`.
* Sử dụng [Appraisal](https://github.com/thoughtbot/appraisal) để test các gems mà có nhiều versions, hoặc support nhiều versions của Rails chẳng hạn.
* Sử dụng [Bundler](http://bundler.io/) để quản lý các dependencies.
* Sử dụng CI để show các build status với code review và test lại với nhiều phiên bản Ruby.

### Rails
* [Add foreign key constaints](http://robots.thoughtbot.com/referential-integrity-with-foreign-keys) vào file migrations.
* Tránh việc bypassing validatios. Ví dụ như: `save(validate: false)`. Chỉ dùng trong trường hợp thực sự cần thiết.
* Tránh việc khởi tạo nhiều hơn 2 objects trong controllers.
* Không thay đổi file migrations sau khi nó đã được merge vào `master`.
* Không nên gọi trực tiếp model class ở trong view.
* Không nên return `false` trong `ActiveModel`, thay vào đó, hãy raise exceptions.
* Không nên sử dụng instance variables trong partials view. Truyền các biến này vào partials từ đoạn render ra template.
* Không sử dụng SQL hoặc SQL fragments (ví dụ: `where('inviter_id IS NOT NULL')`) ở bên ngoài models.
* Tạo [Spring binstubs](https://github.com/sstephenson/rbenv/wiki/Understanding-binstubs) cho từng project, như `rake` và `rspec`, sau đó add vào version control.
* Nếu như bạn set default value cho 1 trường nào đó, hãy set ở trong file migrations.
* Luôn giữ file `db/schema.rb` trong version control.
* Chỉ nên sử dụng 1, 2 instance variables trong mỗi view.
* Sử dụng SQL, không nên dùng `ActiveRecord` trong file migration.
* Sử dụng [.ruby_version](https://gist.github.com/fnichol/1912050) file để chỉ định ruby version cho mỗi project.
* Sử dụng hậu tố `_url` cho tên routes ở trong mailer và [redirect](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30). Sử dụng hậu tố `_path` trong các trường hợp còn lại.
* Validate quan hệ `belongs_to` object `user`, chứ k phải qua column `user_id`.
* Sử dụng dữ liệu trong file `db/seed.rb` cho những dữ liệu mà require ở tất cả các môi trường.
* Sử dụng `dev:prime` task để seed dữ liệu cho môi trường development.
* Nên sử dụng `cookies.signed` thay cho `cookies` để [ngăn chặn giả mạo](http://blog.bigbinary.com/2013/03/19/cookies-on-rails.html).
* Nên sử dụng `Time.current` thay cho `Time.now`, `Date.current` thay cho `Date.today`, `Time.zone.parse('date_string')` thay cho `Time.parse('date string')`
* Sử dụng ENV.fetch cho các biến môi trường, thay vì dùng `ENV[]` để các biến môi trường được detect khi deploy.
* [Sử dụng blocks](https://github.com/thoughtbot/guides/blob/master/best-practices/samples/ruby.rb#L10) để định nghĩa date/time attributes trong FactoryBot factories.
* Sử dụng `touch: true` khi định nghĩa `belongs_to` relationships.

### Testing
* Tránh sử dụng `any_instance` trong rspec-mock và mocha. Ưu tiên dùng [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection)
* Tránh sử dụng `its`, `specify`, và `before` trong RSpec.
* Tránh `let` (or `let!`) trong RSpec. Ưu tiên tách thành helper methods, nhưng không cần phải implement lại method đó. Xem [ví dụ](https://github.com/thoughtbot/guides/blob/master/style/testing/avoid_let_spec.rb)
* Tránh sử dụng `subject` 1 cách tường minh bên trong RSpec `it` block. [Example](https://github.com/thoughtbot/guides/blob/master/style/testing/unit_test_spec.rb)
* Tránh sử dụng instance variables trogn tests.
* Tắt real HTTP request để gọi services bên ngoài. (Khi test không cần gọi API thật ra ngoài). Sử dụng `WebMock.disable_net_connect!`.
* Không cần test private methods.
* Test background jobs sử dụng [Delayed::Job matcher](https://gist.github.com/3186463)
* Sử dụng [stubs và spies](http://robots.thoughtbot.com/post/159805295/spy-vs-spy) (not mocks) trong isolated tests.
* Sử dụng single level of abstraction với scenarios.
* Sử dụng `it` example hoặc test method cho mỗi 1 case của hàm.
* Sử dụng [assertions about state](https://speakerdeck.com/skmetz/magic-tricks-of-testing-railsconf?slide=51) cho các incomming messages.
* Sử dụng stubs và spies để so sánh khi gọi outgoing messages.
* Sử dụng [Fake](http://robots.thoughtbot.com/post/219216005/fake-it) để stub requests khi gọi tới services bên ngoài.
* Sử dụng integration tests để chạy cả app.
* Sử dụng non-[SUT](http://xunitpatterns.com/SUT.html) methods trong việc expectations nếu có thể.

### Bundler
* Định nghĩa rõ [Ruby version](http://bundler.io/v1.3/gemfile_ruby.html) được sử dụng trong project ở trong Gemfile.
* Sử dụng [pesimistic version](http://robots.thoughtbot.com/post/35717411108/a-healthy-bundle) trong `Gemfile` cho những gem follow theo semantic versioning (như `rspec`, `factory-bot`, `capybara`, ...)
* Sử dụng [versionless](http://robots.thoughtbot.com/post/35717411108/a-healthy-bundle) `Gemfile` để định nghĩa những gem mà an toàn để update thường xuyên, như gem `pg`, `thin`, `debugger`.
* Sử dụng [exact version](http://robots.thoughtbot.com/post/35717411108/a-healthy-bundle) trong `Gemfile` cho những gem quan trọng, như `rails`.

### Relation Databases
* [Index foreign keys](https://tomafro.net/2009/08/using-indexes-in-rails-index-your-associations)
* Constrain most columns as [NOT NULL](http://www.postgresql.org/docs/9.1/static/ddl-constraints.html#AEN2444)
* Trong một SQL view, chỉ select những columns bạn cần. Tránh việc `SELECT table.*`

### Email
* Sử dụng [SendGrid](https://devcenter.heroku.com/articles/sendgrid) hoặc [Amazon SES](http://robots.thoughtbot.com/post/3105121049/delivering-email-with-amazon-ses-in-a-rails-3-app) để gửi mail trên staging và production.
* Sử dụng tool như [ActionMailer Preview](http://api.rubyonrails.org/v4.1.0/classes/ActionMailer/Base.html#class-ActionMailer::Base-label-Previewing+emails) để xem mail khi tạo/update mailer view.

### Web
* Tránh việc delays rendering bởi các tiến trình load đồng bộ.
* Sử dụng HTTPS thay cho HTTP khi linking assets.

### JavaScript
* Sử dụng syntax JS mới nhất với 1 trình biên dịch, như [babel](https://babeljs.io/).
* Sử dụng `to_param` hoặc `href` attribute khi serializing ActiveRecord models. Và sử dụng khi xây dựng URLs ở client side, thay vì dùng ID.
* Ưu tiên `data-*` attributes hơn là `id` và `class` attributes khi target HTML elements.
* Tránh việc target HTML elements sử dụng classes để dự định cho mục đích sau sử dụng style.

### HTML
* Sử dụng `<button>` tags thay cho `<a>` tags cho các actions.

### Sass
* Khi sử dụng [sass-rails](https://github.com/rails/sass-rails), sử dụng [asset-helpers](https://github.com/rails/sass-rails#asset-helpers) (`image_url`, `front-url`, ...) để cho Rails Asset Pipeline sẽ re-write the connect paths tới acces.
* Ưu tiên mixins hơn `@extend`.

### Browser
* Tránh support version trước IE11.

### Ruby JSON APIs
* Review những recommend practices ở [HTTP API Design Guide](https://github.com/rails/sass-rails#asset-helpers) trước khi thiết kế 1 API mới.
* Viết intergration tests cho API endpoints. Cần cung cấp đầy đủ [feature specs](https://github.com/rails/sass-rails#asset-helpers) hoặc [request specs](https://github.com/rails/sass-rails#asset-helpers).

## Kết luận
Bài viết trên tổng hợp lại 1 số best practices của các kĩ sư ở ThoughBot trong quá trình code Rails.

Hy vọng bạn có thể áp dụng chúng trong quá trình làm dự án ;)

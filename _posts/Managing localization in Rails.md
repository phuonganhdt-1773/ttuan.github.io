# Managing localization in Rails

# Mở đầu
Đối với những application lớn, việc quản lý các file localization có thể trở thành 1 task rất lớn, đặc biệt nếu như các file localization (translations YAML files, XML, Ruby hash, ...) trong app bạn được 1 team khác cung cấp cho. Bài viết dưới đây sẽ đưa ra một số giải pháp giúp cho việc quản lý các file này của bạ trở nên dễ dàng hơn.


Phần lớn các Rails application đều có 1 lượng người dùng quốc tế, hoặc sẽ có xu hướng mở rộng sản phẩm cho các thị trường nước ngoài. Đó là lý do tại sao chúng ta thường sử dụng I18n khi viết code. Thay vì sử dụng những đoạn code như thế này trong view:

```
<h1>Hello world!</h1>
```

thì chúng ta thường sử dụng 
```
<h1><%= t('hello_world') %></h1>
``` 

để thay thế. `<%= t('key_name') > method sẽ tìm kiếm key matching trong file localization của bạn và hiển thị đoạn text đã được maping với key đó. Trong bài viết này, mình sẽ trình bày lại một số bài học mình đã học được trong việc quản lý localization trong app Rails.

# Managing localization files
## Set team wide formatting rules
Điều đầu tiên và cũng là điều quan trọng nhất khi làm việc trong một team đó là các thành viên cần phải thống nhất về formatting rules khi thêm mới localization texts. Đây là một vài lý do:

* Cần phải có tính nhất quán giữa các key trong file localization. (name convention)
* Giảm thiểu việc bị trùng lặp key.
* Hạn chế việc app Rails không khởi động được do có lỗi formatting trong file yml.
* Có các quy tắc rõ ràng khi làm việc với các vendors của bên thứ 3.

Chúng ta sẽ đi lần lượt từng ý một nhé.

### 1. Quy tắc đặt tên key
Có một điều khá phổ biến trong những app Rails lớn, đó là trong file localization có tới hàng nghìn keys. Chúng ta có thể dễ dàng tìm ra các key bị trùng nhau khi app còn nhỏ, tuy nhiên khi file localization chứa tới hàng nghìn strings thì điều đó là khá khó khăn. Khi đó, nếu không thống nhất quy tắc đặt tên ngay từ đầu, bạn sẽ gặp nhiều khó khăn trong việc tìm bugs trong view.

Chúng ta cùng xem qua 1 ví dụ. Bạn có 2 key trong YAML file giống như thế này:

```
en:
  welcome_back: "Welcome back"
  text_welcome_back: "Welcome back!"
  label_welcome_back: "Welcome back"
```

Cả ba key này đều hiển thị đoạn text là "Welcome back" nhưng có khác biệt nhỏ. Key "welcome_back" và "label_welcome_back" đều hiển thị text "Welcome back" trong khi key "text_welcome_back" thì value của nó lại có thêm 1 dấu chấm than.

Nếu bạn có 1 file localization như trên và bạn làm việc trong 1 team, các developer khác có ý định dùng lại 1 key nào đó, họ sẽ search bằng 1 từ khoá có liên quan(ví dụ "Welcome") và khả năng họ sử dụng 1 key sai là rất lớn. (có thể họ sẽ thu được đoạn "Welcome back!" thay vì "Welcome back").

Vấn đề càng bị ảnh hưởng nhiều hơn khi bạn bắt đầu thay đổi nội dung của các đoạn text trong file view. Bạn sẽ lục tìm trong file YAML key mà có text cần thay đổi. Điều này rất có thể sẽ gây ảnh hưởng xấu, tạo bug tới các view khác cuũng đang sử dụng nó. Do đó, việc đặt ra 1 rule cho cả team trong việc đặt tên key là khá cần thiết. Khi đã có rule chung rồi, mọi người chỉ đơn giản là làm theo nó.

Mình đã từng thấy khá nhiều team làm theo kiểu: Chấp nhận duplicate text nhưng phải có key khác nhau, dựa trên nơi chúng sẽ được gọi. Ví dụ: `text_welcome_back` sẽ chỉ được dùng như 1 simple text, trong khi `label_welcome_back` sẽ được dùng cho label trong 1 form. 

Còn một rule khá quan trọng cần thảo luận với team, đó là: Khi những text khá giống nhau thì việc đặt tên key sẽ như thế nào? Ví dụ như "Welcome back!" và "Welcome back", rõ ràng là chúng chỉ khác nhau 1 dấu chấm than. Bnaj sẽ đặt nó thành 1 key-value khác hay sẽ fix cứng dấu "!" vào trong file HTML? Theo quan điểm cá nhân, mình nghĩ là nên đặt trong file localization bởi vì đối với một số ngôn ngữ như tiếng Tây Ban Nha, dấu "!" sẽ k được đặt ở cuối câu như các ngôn ngữ khác. Trong trường hợp sử dụng 1 key-value khác, chúng ta sẽ dễ dàng hơn trong việc dịch/ thay đổi text ở view.

### 2. Giảm thiểu việc trùng lặp strings
Như mình đã nhắc đến ở trên, việc khôgn có 1 quy tắc đặt tên key thống nhất có thể dẫn đến khá nhiều vấn đề. 1 trong số đó là bị trùng lặp các đoạn string. 

Ví dụ, khi mình search 1 từ nào đó để sử dụng trong file localization, mình thường sử dụng tính năng search của editor để tìm. Đôi lúc, mình tìm thấy 2 cặp key/value với cùng 1 text nhưng lại khác nhau về key. => Mình phải tìm lại các chỗ sử dụng 2 keys này để cân nhắc xem nên chọn key nào cho phù hợp, tránh khi thay đổi value sẽ bị ảnh hưởng, ... Việc có 1 file localization clean sẽ gỉảm thiểu được những vấn đề như thế.

### 3. Giảm khả năng app Rails không thể khởi động do gặp lỗi format
Nếu như localization file của bạn bị lỗi format, app Rails sẽ không thể khởi động được. Logs cũng sẽ không nói chính xác cho bạn biết là file localization nào gây nên vấn đề này, cũng như dòng nào là dòng đang bị lỗi. Việc này khiến các dev mất khá nhiều thời gian để debug. 

Để hạn chế những trường hợp có thể gây lỗi, mình thường áp dụng 1 vài rule:

* Luôn luôn bọc đoạn value trong nháy kép: Nếu bạn không bọc những value đó trong nháy kép, có thể app vẫn hoạt động bình thường. Tuy nhiên, khi trong value của bạn chứa dấu nháy kép, bạn sẽ gặp phải lỗi parsing. Thêm vào đó, việc đặt dấu nháy kép bên ngoài sẽ giúp bạn có thể đặt biến vào trong, hoặc sử dụng nháy đơn, ...
* Nếu như đoạn text của bạn yêu cầu hiển thị nháy kép trong đó, hãy sử dụng dấu `\`. Ví dụ: bạn có thể hiển thị đoạn: Hi "Ted" bằng cách dùng cặp key-value `hi_ted: "Hi \"Ted\""
* Format cho các dấu space, tab phải được thống nhất, không được chỗ dùng tab, chỗ dùng space. như thế sẽ dễ gây ra lỗi khi parsing.


### 4. Quy tắc khi làm việc với vendors của bên thứ 3
Nếu bạn đang làm việc với các file translation của bên thứ 3, việc bạn có 1 bộ các formatting rules sẽ giúp bạn dễ dàng hơn rất nhiều. Việc có rule làm việc chung giữa 2 bên sẽ giúp ta thống nhất được format sẽ sử dụng trong app, từ đó giảm thiểu được khả năng crash app hơn. 

## Autogenerated translations
Bạn cũng có thể dịch app của mình sang một ngôn ngữ khác sử dụng các service của Google Translate và Yandex. Tuy nhiên, bên cạnh các ưu điểm của cách này thì cũng tồn tại khá nhiều nhược điểm:

* Ưu điểm: 
+ Nhanh và dễ dàng để dịch app sang nhiều thứ tiếng.
+ Rẻ hơn là thuê người dịch.
+ Bạn có thể viết rake task để tự động dịch bất cứ string nào có trong app của bạn.

* Nhược điểm:
+ Các từ dịch thì thường không sát nghĩa.
Do hạn chế trong các translate engine, bạn có thể dịch được một số mẫu câu đơn giản khá chính xác. Tuy nhiên với các câu phức tạp, các bản dịch thường không sát nghĩa. Ví dụ, các tool dịch từ tiếng Anh sang tiếng Tây Ban Nha có thể khá ổn, nhưng từ tiếng Anh sang tiếng Trung thì thường không sát nghĩa. 

## Writing test
Khi làm việc với các file localization của bên thứ 3, có thể bạn sẽ bị gặp lỗi typos. Một lỗi typos khá hay gặp đó là ngôn ngữ ở top của file YAML bị sai. 

Ví dụ, đây là file YAML khi dịch sang tiếng Hàn:
```
---
ko:
  hello: "안녕햐세요"
```

Bạn có thấy `ko` được định nghĩa ở đầu file không. Đôi khi, chúng ta sẽ phải thay đổi language key (do language key không giống với các chữ cái trong tên tiếng Anh của nước đó). Ví dụ như "es" là key cho "Spanish".

Đây là 1 lỗi typo dễ bị miss. bạn có thể viết test cho case này như đoạn code dưới:

```
require 'test_helper'

class YamlLocaleKeyTest < ActiveSupport::TestCase
  test 'English key is correct' do
    I18n.locale = 'en'
    file_dir = Rails.root.join('config', 'locales', 'en.yml').to_s
    file = YAML.load_file(file_dir)
    assert file.keys.include? 'en'
  end

  test 'German key is correct' do
    I18n.locale = 'de'
    file_dir = Rails.root.join('config', 'locales', 'custom', 'de.yml').to_s
    file = YAML.load_file(file_dir)
    assert file.keys.include? 'de'
  end

  test 'Spanish key is correct' do
    I18n.locale = 'es'
    file_dir = Rails.root.join('config', 'locales', 'custom', 'es.yml').to_s
    file = YAML.load_file(file_dir)
    assert file.keys.include? 'es'
  end

  test 'French key is correct' do
    I18n.locale = 'fr'
    file_dir = Rails.root.join('config', 'locales', 'custom', 'fr.yml').to_s
    file = YAML.load_file(file_dir)
    assert file.keys.include? 'fr'
  end

  # Add more tests here if you have more languages
end
```
Ngoài ra, bạn có thể viết các test khác để check các lỗi bạn hay gặp để đảm bảo việc load file YAML sẽ không gây ra vấn đề gì. 

# Kết luận
Khi Rails app của bạn lớn hơn, việc quản lý localization file là 1 việc khá quan trọng và cần đươc chú ý. Bài viết có chia sẻ với bạn 1 số cách để quản lý localization file trong Rails app. Hy vọng có thể giúp ích được cho bạn và dự án.

Nguồn tham khảo: https://thisbitofcode.com/managing-localization-rails/



















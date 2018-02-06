# Rack - Người hùng thầm lặng

## Tản mạn
Vào một ngày đẹp trời, trong lúc đang làm 1 tính năng của Rails web app, mình cần thêm 1 vài config vào trong file `config/application.rb`. Khi bấm `Ctrl + p` để search file này, vừa mới gõ chữ `config` thì [Vim](https://ttuan.github.io/tags/#Vim) nó bỗng gợi ý 1 file tên là `config.ru`. Code Rails cũng được gần 2 năm rồi, mình cũng đã từng nhiều lần để ý tới cái file này, nhưng mọi chuyện vẫn chỉ dừng lại ở "để ý" (yaoming). Hôm nay, nhân lúc rảnh rỗi mà trí tò mò lại nổi lên, mình quyết định sẽ tìm hiểu thử xem thằng này là thằng nào :v

## Rack
Mở file `config.ru` lên thì code rất ngắn gọn, chỉ có thế này:
```
# This file is used by Rack-based servers to start the application.

require_relative 'config/environment'

run Rails.application
```
Có một dòng comment rất rõ ràng ở đầu file đó là file này được sử dụng bởi Rack-based server, dùng để khởi động app của mình. GG thử thì định dạng file `.ru` - đại diện cho từ `rackup`. Rack? có lẽ bạn đã từng gặp/ nhìn thấy từ này rất nhiều lần trong khi làm việc với Rails :v Vậy cụ thể thì Rack là gì và tại sao Rails app lại cần tới nó?

### 1. Rack là gì?
Theo định nghĩa tại [repo](https://github.com/rack/rack) của Rack:
>Rack provides a minimal, modular, and adaptable interface for developing web applications in Ruby.
> By wrapping HTTP requests and responses in the simplest way possible, it unifies and distills the API for web servers, web frameworks, and software in between (the so-called middleware) into a single method call.

Trước khi bắt đầu tìm hiểu về Rack, mình muốn nói qua về khái niệm Web Server. Theo như định nghĩa trên [Wiki](https://en.wikipedia.org/wiki/Web_server), web server là 1 phần mềm dùng để nhận và phản hồi lại HTTP requests. Ta có 1 follow ntn: ![](https://viblo.asia/uploads/29d61fda-6b01-4db8-9b7f-510ef5ffb5f9.png)

Khi request được gửi lên, nó sẽ được handle bởi web server (có thể là Puma, Thin, Webrick, Passenger, ..). Trước khi đến được với webserver, http request có định dạng rất phức tạp:
![](https://viblo.asia/uploads/e2cee2f6-5be6-4915-a507-e9118cad1f35.png)

nhưng webserver sẽ xử lý nó để đưa ra một định dạng để bên app app có thể dễ dàng hiểu được.
![](https://viblo.asia/uploads/e40580c5-5e14-4ca7-bb1a-e64bda6fb5da.png)

Sau khi xử lý xong, webserver gửi kết quả về cho bên Application, cụ thể ở đây là Rails. Rails sẽ bắt đầu thực hiện logic mà ta đã implement, sau đó trả về kết quả cho WebServer, webserver sẽ convert thông tin vừa nhận được thành dạng HTTP response, sau đó gửi về cho browser.

Mọi thứ có vẻ khá rõ ràng, rằng Web Server có thể *nói chuyện* với web framework. Tuy nhiên, có 1 vấn đề khá nghiêm trọng dẫn đến việc ra đời của Rack.

+ Có rất nhiều web server support cho Ruby: Puma, Thin, Passenger, Unicorn, ...
+ Có rất nhiều web frameworks cho Ruby: Rails, Cuba, Roda, Sinatra, Hanami, ...

=> Nếu Puma muốn *nói chuyên* trực tiếp với Rails, cả Rails và Puma cần phải có 1 chuẩn chung: Puma chuyển cho Rails 1 đầu vào mà Rails có thể hiểu được, xử lý xong, Rails trả lai cho Puma 1 đầu ra mà Puma có thể hiểu được.

Tuy nhiên, nếu Puma muốn support cho Sinatra, nó lại phải có 1 chuẩn khác để giao tiếp với Sinatra. Tương tự như vậy ở phía Web framework.

==> Cần phải có 1 common standard interface nằm giữa web servers và web frameworks. Hiểu như là 1 ngừời phiên dịch giữa web server và web framework.

Bây giờ flow sẽ là:
![](https://viblo.asia/uploads/220cef84-7670-403e-a437-b711ec7b92d2.png)

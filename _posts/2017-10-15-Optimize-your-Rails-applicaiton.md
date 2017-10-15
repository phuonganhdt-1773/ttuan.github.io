---
layout: post
title: Làm thế nào để tăng tốc độ load một trang web?
tags:
- Rails
- Nginx
---

## Mở đầu
Dạo gần đây mình có làm 1 trang web bán hàng, chức năng đơn giản chỉ là list sản phẩm, show trang chi tiết, tìm kiếm và đặt hàng :easy:. Trong quá trình phát triển tới lúc deploy lên production, mình thấy tốc độ của nó khá ổn -> khá là hài lòng. 

Mọi chuyện sẽ chẳng có gì cho tới khi mình gửi trang web đó cho 1 ông anh, nhờ ông ý check thử xem có góp ý gì không. Lúc anh ý mở lên thì "Ôi, sao web của m chậm thế? t load cả phút đồng hồ vẫn không xong?". WTH? Mình load thấy nhanh mà? K tin nên mình load lại, vẫn thấy OK (khoảng 3 -> 5s). Lúc rep lại bảo anh check thử mạng máy anh xem, ông ý bảo "t mở bằng cửa sổ ẩn danh". 

Ah. Mình test lại và đúng là thế thật. Do mình dev + test, browser thì nó sẽ cache lại -> Tốc độ nhanh hơn nhiều. Hầy, noob quá :3 :3 :3 Thế là mình google để tìm cách cải thiện tốc độ cho eny này.

**NOTE**: Trong bài viết này, mình sẽ không đề cập tới việc tối ưu hoá code, thuật toán, hay sử dụng application cache. Có thể sẽ là trong 1 bài viết khác :D

## Tìm nguyên nhân

Trước tiên, để có thể Optimizing được trang web, bạn cần phải biết nó đang chạy chậm ở đâu. Biết nguyên nhân rồi thì mới có thể fix được chứ đúng không =)))

Rút kinh nghiệm từ vụ lúc dev không để ý về performance, giờ mình sẽ kiểm tra trên con production :3 Sau khi lần mò 1 lúc thì mình tìm được mấy tools để phân tích web page: [Google PageSpeed](https://developers.google.com/speed/pagespeed/insights/), [GT Metrix](https://gtmetrix.com/) và [Web Page Test](https://www.webpagetest.org/).

Khi nhập url trang web của mình vào trang Google PageSpeed, mình đã check được ra 1 số nguyên nhân làm chậm web:
* Ảnh quá nặng, chưa qua nén, resize. (Thực ra mình có để quản lý ảnh theo version nhưng version lấy quality vẫn cao quá :v)
* Chưa gzip các file css, js.
* Chưa tận dụng tối ưu vụ http cache.

## Giải pháp

### 1. Xử lý phần ảnh
#### a. **Tạo version ảnh với chất lượng, dung lượng phù hợp**

Việc đầu tiên mình nghĩ đến là tạo ra các version ảnh với chất lượng, resize nhỏ đi. Vì nhiều vị trí đặt ảnh khá nhỏ, k cần nhất thiết phải lấy ảnh dung lượng cao. 

Cách quản lý ảnh theo version thì cũng đã được hướng dẫn khá nhiều trên mạng. Ở đây mình dùng `carierwave`.

```
# config/initializers/carrierwave.rb
module CarrierWave
  module RMagick
    def quality percentage
      manipulate! do |image|
        image.write(current_path){self.quality = percentage} unless image.quality == percentage
        image = yield(image) if block_given?
        image
      end
    end
  end
end

```

```
# app/uploaders/image_uploader.rb
class ImageUploader < CarrierWave::Uploader::Base
  include CarrierWave::RMagick

  version :preview do
    process quality: 40
    process resize_to_limit: [400, 400]
  end

  version :small do
    process quality: 40
    process resize_to_limit: [200, 200]
  end

  .....
end
```
Okie, việc của bạn giờ chỉ là đặt version phù hợp vào từng chỗ thôi!

Ngoài ra bạn có thể tham khảo thêm 1 số cách khác ở [đây](https://cloudinary.com/blog/image_optimization_in_ruby)

#### b. **Sử dụng biện pháp Progressive Image Loading**
Đây là một biện pháp rất thú vị mà mình đọc được từ bài viết trên [Medium](https://jmperezperez.com/medium-image-progressive-loading-placeholder/). Nguyên tắc của phương pháp này đó là:

* Đặt 1 placeholder trống.
* Thay thế nó bằng 1 ảnh có chất lượng thấp (blurring image).
* Sau đó thay thế nó bằng 1 ảnh chất lượng cao.

Cách này khá thú vị. Do không phải load ảnh chất lượng cao ngay từ đầu nên **thời gian load trang sẽ giảm đi đáng kể**. Sau khi html của trang đã được load xong thì dùng js để load dần các ảnh. Bạn có thể thêm kiểu khi người dùng lăn chuột tới đâu thì mình mới bắt đầu load ảnh (Lazy load). Bạn có thể tham khảo cách làm trong bài viết [này](https://blog.botreetechnologies.com/page-load-optimization-by-progressive-image-loading-like-medium-1d0f94744a4d)

Giờ thì bắt đầu thôi ;) . Code dưới đây mình tham khảo từ repo [này](https://github.com/AnkurVyas-BTC/test_page_performance)

```
// app/assets/stylesheets/product.css.scss

.placeholder {
  background-color: #f6f6f6;
  background-size: cover;
  background-repeat: no-repeat;
  position: relative;
  overflow: hidden;
}

.placeholder img {
  position: absolute;
  opacity: 0;
  top: 0;
  left: 0;
  width: 100%;
  transition: opacity 1s linear;
}

.placeholder img.loaded {
  opacity: 1;
}

.img-small {
  filter: blur(50px);
  /* this is needed so Safari keeps sharp edges */
  transform: scale(1);
}
```

```
// progressive_image_loading.js
window.onload = function() {

  var placeholder = $('.placeholder');

  placeholder.each( function(index) {
    // 1: load small image and show it
    var smallImgElement = $(this).find('.img-small');
    var img = new Image();
    img.src = smallImgElement.attr('src');
    img.onload = function () {
      smallImgElement.addClass('loaded');
    };

    // 2: load large image
    var imgLarge = new Image();
    imgLarge.src = $(this).data('large');
    imgLarge.onload = function () {
      imgLarge.classList.add('loaded');
    };
    $(this).append(imgLarge);

  })

}
```
```
# app/views/products/show.html.erb
...
<div class="placeholder" data-large="<%= product.picture_url(:preview) %>">
  <img src="https://cdn-images-1.medium.com/freeze/max/27/1*sg-uLNm73whmdOgKlrQdZA.jpeg?q=20" class="img-small">
  <div style="padding-bottom: 66.6%;"></div>
</div>
...
```
Sau khi làm xong, mình mở tab Network trên Google Chrome để test lại. Yeah, Page Load Time(Cache disable) đã giảm đi đáng kể. Giờ thời gian load trang còn khoảng 8s (T.T)

#### c. **Sử dụng ảnh WebP**
`WebP` là một loại format ảnh được phát triển bởi Google, dùng cả kiểu nén lossy và lossless. Hiện nay, chỉ có Google Chrome(+ Android) và Opera là support cho loại ảnh này. (Đọc thêm [Why Firefox still not supporting webp](https://www.reddit.com/r/firefox/comments/46wxye/why_is_firefox_still_not_supporting_webp/)). Nhưng k sao, chúng ta có thể config để show ảnh webp trên google chrome, còn trên firefox ta show ảnh khác (png, jpg or gif).

Tại sao lại là chuẩn WebP? Đơn giản vì nó rất nhẹ, chất lượng ảnh thì đảm bảo rất OK mà dung lượng thì cực ổn :v Cách áp dụng với Rails application thì bạn có thể tham khảo tại bài viết [này](http://leopard.in.ua/2013/11/23/rails-and-webp#.WeLt84YxVE4). Trong bài này mình xin tóm lược như sau:

```
gem "sprockets-webp"
```
Đặt ảnh png và jpg vào trong "app/assets/images" và bạn chạy lệnh sau để convert:

```
bundle exec rake assets:precompile RAILS_ENV=production
```
Bây giờ thì trong thư mục `public/assets` đã có ảnh webp rồi đó.

Giờ mình config trên Ngnix để trả về ảnh webp tới các trình duyệt hỗ trợ. Do Chrome và Opera sẽ gắn thêm `images/webp` trong Accept header của tất cả các requests ảnh, nên chúng ta có thể dùng Nginx để chọn ảnh đúng.

```
location ~ ^/(assets)/  {
   # check Accept header for webp, check if .webp is on disk
   if ($http_accept ~* "webp") { set $webp "true"; }
   if (-f $request_filename.webp) { set $webp "${webp}-local"; }
   if ($webp = "true-local") {
    add_header Vary Accept;
    access_log   off;
    expires      30d;
    rewrite (.*) $1.webp break;
   }
}
```
Done. bây giờ bạn có thể mở Chrome và Firefox ra, load thử và cảm nhận ;)

### 2. Enable gzip to Nginx
Tốc độ load của 1 trang web phụ thuộc vào size của các files chưa trong 1 trang. Các file (ảnh, css, js, html, ...) sẽ được download về browser. Nếu ta giảm được dung lượng các files này rồi mới truyền qua http request, có thể k làm trang web nhanh hơn nhiều, nhưng chắc chắn nó sẽ giảm được bandwidth sử dụng.

Ý tưởng là ta sẽ nén các files ở trên server trước khi trả về cho browser dưới dạng `gzip`. Sau đó browser sẽ giải nén nó. Tuỳ vào loại file mà độ nén sẽ khác nhau. Vd: text files nén được nhiều nhất, trong khi đó các files ảnh thì nén được ít hơn. Okie, giờ chúng ta đi config cho Nginx thôi ;)

```
# /etc/nginx/conf
gzip on;
gzip_disable "msie6";

gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
# gzip_min_length 256; #enable dòng này thì gzip sẽ k nén các file dưới 256 bytes
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;
```

Bây giờ, bạn có thể test lại bằng cách mở browser, click tab network và chọn 1 request (vd request lấy ảnh) và xem thử Header nó. bạn sẽ thấy có thêm `Content-Encoding: gzip` ;)

Bạn có thể thêm đoạn config sau vào trong Rails app. Đoạn này mình xem được ở trang [này](https://robots.thoughtbot.com/content-compression-with-rack-deflater) nhưng về ảnh hưởng của nso thì chưa được kiểm tra =)))

```
module YourApp
  class Application < Rails::Application
    config.middleware.use Rack::Deflater
  end
end
```


### 3. Enable cache
Đối với nhiều trang, chúng ta rất ít khi thay đổi resouces, nên là nếu chúng ta cache resources lại browser, lần sau khi người dùng truy cập vào, resource đã có sẵn và k cần load nữa ;) Thế là vừa cải thiện tốc độ vừa tiết kiệm được băng thông :v 

Config đơn giản, bạn chỉ cần đặt đoạn sau vào trong file `site-avaiable/config file`

```
location ~* \.html$ {
  expires -1;
}

location ~* \.(css|js|gif|jpe?g|png)$ {
  expires max;
  add_header Pragma public;
  add_header Cache-Control "public, must-revalidate, proxy-revalidate";
}
```

Bây giờ thì các file của bạn đã được trình duyệt cache lại rồi đó. Ở đây mình disable cache với các file `html` vì mình thấy các file này hay thay đổi, k nhất thiết phải cache lại. :3 

## Kết luận
Sau khi mình sử dụng 1 số biện pháp trên thì tốc độ load trang của mình đã giảm đáng kể. Bây giờ reload trên cửa sổ ẩn danh, no cache thì tốc độ load đã vào khoảng 4s ;) (ngon). Nhưng quan trọng hơn, bây giờ mình đã đỡ gà đi được 1 tí =))))


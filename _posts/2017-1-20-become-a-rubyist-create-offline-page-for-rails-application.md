---
layout: post
comments: true
title: Become a Rubyist: Create offline page for Rails application
tags:
- Become a Rubyist
---


*Sử dụng Service Worker để kết nối với người dùng ngay cả khi không có mạng*

![](https://viblo.asia/uploads/199cc148-edfe-4ba4-8f89-9ce3cc18b7bd.jpg)
## Giới thiệu
Khi bạn truy cập vào một website nào đó bằng Chrome mà chưa kết nối mạng, bạn sẽ trông thấy hình ảnh chú khủng long như hình trên. :3  Trước đây, khi mình là một normal user, điều này cũng k gây nhiều khó chịu (đang duyệt web mà tự nhiên mất mạng thì chịu thôi). Tuy nhiên, khi trở thành một web developer, mình muốn có một thông báo gì đó cho người dùng, hay nói cách khác là custom lại cái trang "Khủng long" offline kia.

Khi thử google về cách tạo một trang offline, mình khá ngạc nhiên khi có rất nhiều cách để thực hiện. Chúng ta có thể sử dụng [App Cache](http://diveintohtml5.info/offline.html) và Cache Manifest để tạo 1 trang offline. Khá là đơn giản. Tuy nhiên, theo một số dev, cách sử dụng App Cache này còn tồn tại khá nhiều [vấn đề](http://alistapart.com/article/application-cache-is-a-douchebag).

Thật may là còn có một cách làm sử dụng 1 web standard mới, [Service Worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API). Đây là một sự thay thế có tiềm năng cho App Cache, bằng cách sử dụng JavaScript, hơn là việc lưu trữ mainifest files.

Bây giờ, chúng ta sẽ cùng nhau sử dụng Service worker để render ra một trang thông báo lỗi đơn giản, để thông báo với người dùng của chúng ta khi họ truy cập vào site mà không có kết nối internet.
Để làm được điều này, chúng ta sẽ sử dụng service worker để precache, tạo 1 offline assets trong lần đầu tiên người dùng truy cập vào site. Sau đó, trong những lần truy cập sau, nếu không có mạng, chúng ta có thể sử dụng service worker để render ra trang offline đó.

Điều này là hoàn toàn có thể vì Service Worker hoạt động như một người liên kết giữa browser của người dùng và servers outside khi vượt ra vòng đời của 1 page.

Có một lưu ý đó là Service Worker [không hoạt động trên tất cả các trình duyệt.](https://jakearchibald.github.io/isserviceworkerready/). Do đó, cách làm dưới đây sẽ không phải cho tất cả người dùng, tuy nhiên, đó sẽ mang lại những trải nghiệm khác cho đại đa số người dùng.

## Tạo offline page
### Produce the assets
Đầu tiên, chúng ta sẽ phải tạo một trang offline,. Chúng ta có thể dễ dàng tạo ra một trang HTML tĩnh nằm trong thư mục public. Bạn có thể tạo page với style giống như những page tĩnh đã được generated sẵn như page 404 và 500.
Bạn có thể tham khảo source code: [/offline.html](https://gist.github.com/rossta/c4f6de214a138a355a9993c7cdadbdc0).
Bạn cũng có thể sử dụng cách khác, cài đặt 1 route và thêm controller action theo hướng dẫn ở [đây](https://mattbrictson.com/dynamic-rails-error-pages).

### Thêm một service worker file
Chúng ta sẽ cache trang HTML offline này trên client side trong lần truy cập đầu tiên của người dùng để có thể sử dụng sau này. Bạn cũng có thể thêm các file CSS, JavaScript và ảnh cho trang offline này, chỉ cần bạn nhớ cache cho tất cả những resources này là được.
File script service worker cần được đặt ở ngoài application.js hoặc các bundled assets khác. Nó có thể đặt ở bất kì chỗ nào mà Sprockets có thể load assets, nhưng bây giờ, chúng ta sẽ đặt một file JS trong `app/assets/javascripts/serviceworker.js`. Do file này không được bundled cùng với application.js, chúng ta sẽ phải config để Rails có thể precompile serviceworker file:

```
# config/initializers/assets.rb

Rails.application.config.assets.precompile += %w[serviceworker.js]
```

### Tạo một 'install' event
Vì service workers thuộc dạng hướng sự kiện nên chúng ta cần cung cấp các callbacks, tạo ra 3 key events trong vòng đời một service worker: `install`, `active` và `fetch`.
Sự kiện `install` sẽ được gọi trong lần đầu tiên service worker được gọi hoặc bất cứ khi nào nó được update hoặc có yêu cầu active lại. Bây giờ, chúng ta sẽ precache offline asets bằng đoạn code:

```
var version = 'v1::';

self.addEventListener('install', function onInstall(event) {
  event.waitUntil(
    caches.open(version + 'offline').then(function prefill(cache) {
      return cache.addAll([
        '/offline.html',
        // etc
      ]);
    })
  );
});
```

`event.waitUntil` cho phép một [promise](https://developers.google.com/web/fundamentals/getting-started/primers/promises) đảm bảo sự thành công của `install` event để cài đặt service worker. CHúng ta sử dụng `caches.open` để trả về một promise và thêm offline asets tĩnh để đặt tên đoạn cache liên quan giữa site của chúng ta và browser của người dùng. [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache) cho phép chúng ta lưu trữ theo từng cặp request/ response, khá gioogns với việc xây dựng HTTP cache.
Bạn cũng có thể cache precompiled assets bằng cách đổi tên `serviceworker.js` thành `serviceworker.js.erb` và nhúng vào helper methods như sau:
```
return cache.addAll([
  '/offline.html',
  '<%= asset_path "application.css" %>',
]);
```

### 'fetch' or fallback
Service worker của chúng ta có thể chặn bất kỳ yêu cầu mạng ra bên ngoài từ trình duyệt - kể cả các cross-origin host - với 'fetch' event. Chúng ta có khá nhiều cách để tạo 1 offline page để phản hồi lại các request:
```
self.addEventListener('fetch', function onFetch(event) {
  var request = event.request;

  if (!request.url.match(/^https?:\/\/example.com/) ) { return; }
  if (request.method !== 'GET') { return; }

  event.respondWith(
    fetch(request).                                      // first, the network
      .catch(function fallback() {
        caches.match(request).then(function(response) {  // then, the cache
          response || caches.match("/offline.html");     // then, /offline cache
        })
      })
  );
});
```

Đoạn code này sẽ filter các GET request từ host của chúng ta. Nếu có mạng, mọi thứ vẫn diễn ra bình thường. Khi chúng ta muốn cung cấp một offline fallback, chúng ta yêu cầu mạng phải `fetch` request đó. Nếu không được giải quyết, `catch` handler sẽ được gọi và trang offline sẽ được trả về.
### Clean up trong khi 'active'
Sự kiện `active` sẽ rất hữu ích để chúng ta dọn dẹp lại các caches cũ, đặc biệt là trong trường hợp page offline hoặc bất kì một static resources liên kết với offline page thay đổi.

```
// var version = "v2::";

self.addEventListener('activate', function onActivate(event) {
  event.waitUntil(
    caches.keys().then(function deleteOldCache(cacheNames) {
      return Promise.all(
        cacheNames.filter(function(cacheName) {
          return key.indexOf(version) !== 0;
        }).map(function(cacheName) {
          return caches.delete(cacheName);
        })
      );
    })
  );
});
```
Nếu chúng ta deploy một service workder với một version mới, sự kiện `install` sẽ được gọi lần nữa và cache lại static resources cho trang offline. Trong suốt khoảng thời gian `active`, tất cả các cache mà tên của nó không match với version mới sẽ bị xóa bỏ.

### Register that worker
Sau khi đã tạo xong sự kiện `active` và `install`, chúng ta cần đăng ký đoạn script này từ main page.
```
// app/assets/application.js

if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/serviceworker.js');
}
```
### Sprinkle in some middleware
Để tương tác dễ hơn với Rails aset pipeline, chúng ta có thể sử dụng gem `serviceworker-rails`.
```
# Gemfile

gem "serviceworker-rails"
```
`ServiceWorker::Rails` sẽ chèn middleware vào Rails stack để chúng ta có thể config route request và bundle asset
```
# config/initializers/serviceworker.rb

Rails.application.configure do
  config.serviceworker.routes.draw do
    match "/serviceworker.js"
  end
end
```
Bây giờ, bất cứ request nào tới path `/serviceworker.js` sẽ match với asset file. Nếu bạn đặt service worker trong 1 thư mục, bạn có thể sử dụng đoạn code sau:

```
match "/serviceworker.js" => "nested/directory/serviceworker.js"
```
Bạn có thể đọc thêm tại file [README](https://github.com/rossta/serviceworker-rails/blob/master/README.md) để có thể config middleware tốt hơn.

### Moment of truth
Vậy là chúng ta đã xong phần setup. Trang offline bây giờ đã sẵn sàng để sử dụng. Bây giờ, hãy thử tắt mạng của bạn đi và test thử. Chúng ta có thể sử dụng `Network` tab trong Chrome để sử dụng browser ở chế độ offline. Nếu bạn dùng Firefox có thể sử dụng Work Offline mode.
![](https://viblo.asia/uploads/efe4170e-c3ab-4d7d-a807-aec2defd1b61.jpg)
Để có thể xem đoạn demo, check thử trang [Service Worker Rails Sandbox](https://serviceworker-rails.herokuapp.com/offline-fallback/), sau đó ấn F12, mở tab Network và click vào box Offline rồi thử refresh lại trang, bạn sẽ nhận được thông báo:
![](https://viblo.asia/uploads/c81828c0-a335-4344-bb39-37cacde5df75.png)

Bạn có thể xem code ở [đây](https://github.com/rossta/serviceworker-rails-sandbox)

## Kết luận
Mình tin rằng đối với 1 web dev, chúng ta sẽ luôn nghĩ tới việc làm sao để app của mình trở nên tốt hơn, thân thiện hơn trong con mắt của khách hàng. Bài viết này hướng dẫn các bạn render một trang offline page khi người dùng đột nhiên mất mạng, không thể kết nối internet bằng cách sử dụng Service Worker API. Hy vọng bài viết có ích cho các bạn :D
Happy Coding. ;)

Nguồn: https://rossta.net/blog/offline-page-for-your-rails-application.html

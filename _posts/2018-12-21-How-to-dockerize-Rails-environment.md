---
layout: post
title: Docker series - Part 1 - How to dockerize Rails environment
tags:
- Docker
---

## Tản mạn
Trước đây mình cũng đã có thời gian tìm hiểu về Docker, và cũng đã viết 1 bài [Dockerizing Rails application](https://ttuan.github.io/2017/10/15/Dockerizing-Rails-application/). Tuy nhiên nếu chỉ dừng lại ở mức độ tìm hiểu mà không thực hành thì mình sẽ rất dễ quên. Và đúng là mình cũng quên luôn thật =))

Đợt gần đây, dự án mình làm cần phải dùng đến Elasticsearch, và dĩ nhiên, 1 loạt vấn đề phát sinh. Đầu tiên là do phiên bản ES ở máy mình. Do cài từ apt list của Ubuntu chưa được update, máy mình được cài bản ES 1.7, trong khi hiện tại ES đã lên tới version 6.5 :( Thế là phát sinh lỗi.

Sau khi fix được vấn đề này rồi, mình cần cài thêm ELK Stack (Elasticsearch, Kibana, Logstash), và lại 1 loạt vấn đề bị phát sinh. Rồi sau đó, khi muốn cài 1 plugin vào ES, mình lại phải note lại để báo với "sếp" để khi deploy lên staging nhớ mà cài.

Và 1 loạt các vấn đề khác nữa, chung quy lại cũng chỉ là tại thằng "môi trường code" -__-. Ngay lúc đó, mình lại nhớ tới thằng bạn "docker" =)) Và đúng là nó đã giải quyết được, cài ELK stack chỉ với 1 dòng lệnh :3 (Thanks for [docker-elk](https://github.com/deviantony/docker-elk))

Rồi mình dự định viết 1 serie các bài viết về Docker, áp dụng cho dự án Rails. Không phải kiểu tìm hiểu hết rồi mới viết bài, mà sẽ theo format: Áp dụng từ mức cơ bản nhất, sau đó gặp vấn đề gì thì sẽ tìm cách giải quyết. Nếu có thể sẽ thêm 1 phần nữa để giải thích tại sao giải pháp đó giải quyết được vấn đề của mình (theo kiểu dive vào bên trong solution xem họ làm ra sao). Hy vọng mình có đủ động lực để viết hết cái serie này =))

Ok. Let's go

## Vấn đề
1. Về môi trường làm việc:
	+ Mỗi khi chúng ta bắt đầu join 1 dự án nào đó, việc đầu tiên là cài đặt môi trường. Ví dụ: ruby 2.5.1, rails 5.0.2, mysql 5.7, .... Có khi đã mất hẳn 1 buổi chỉ để ngồi cài môi trường. Nếu chỉ code trên máy của công ty, bạn chỉ mất 1 buổi ngồi cài, tuy nhiên tối vừa về nhà thì sếp lại gửi cho cái bug, bạn lại lóc cóc mở lap cá nhân lên rồi tìm cách fix, tuy nhiên môi trường trên máy laptop cá nhân lại không đáp ứng được, lại ngồi cài lại môi trường?
	+ 1 ngày bạn đọc được 1 bài viết về ngôn ngữ Go chẳng hạn, và bỗng nổi hứng muốn thử tìm hiểu xem nó như thế nào, tuy nhiên lại k muốn cài nguyên cả môi trường code Go vào máy, nhỡ thấy nó chẳng ra gì thì lại ngồi lọ mọ xóa đi? nhưng liệu bạn có nhớ đã cài những gì hay không? liệu nó có thực sự xóa hết các dependence ra khỏi máy?
	+ Bạn được QA thông báo 1 lỗi trên production, nhưng khi bạn test lại trên local ở môi trường dev thì lại không bị, vậy ta sẽ debug thế nào?
2. Về việc đưa app lên server staging và production
	+ Với mỗi dự án, sau khi code xong trên môi trường dev, chúng ta sẽ đưa lên môi trường staging để test, nếu ok thì mới đưa lên môi trường production. Mỗi khi cần thêm 1 plugin hay 1 config nào đó, chúng ta thường sẽ phải ssh vào server và chỉnh sửa bằng tay. Mọi việc khá là thủ công và mất thời gian.

Còn khá nhiều use case liên quan tới bảo mật, CI, microservices, ... mà docker có thể giúp chúng ta giải quyết, tuy nhiên, như đã nói ở đầu bài, mình sẽ đi từ những vấn đề mình gặp trong thực tế, sau đó nếu có vấn đề phát sinh, mình sẽ tìm cách giải quyết nó, nên tạm thời chúng ta sẽ không nhắc nhiều tới những use cases này.

## Solution - Dockerizing Rails app

Trong serie này, mình sẽ để code tại repo: [rails-docker-demo](https://github.com/ttuan/rails-docker-demo). Code cho từng post sẽ được để vào trong từng pull request.

Việc đầu tiên chúng ta cần làm là config cho môi trường code ruby bằng cách tạo 1 [Dockerfile](https://docs.docker.com/glossary/?term=Dockerfile) ở thư mục root của project:

```
# Choose ruby version, you can choose another version
FROM ruby:2.5.1

# Install apt based dependencies required to run Rails as well as RubyGems.
RUN apt-get update && apt-get install -y build-essential nodejs

# Configure the main working directory.
ENV APP_DIR /app
RUN mkdir -p $APP_DIR
WORKDIR $APP_DIR

# Copy the main application.
COPY . $APP_DIR
RUN bundle install

CMD rails server -b 0.0.0.0 -p 3000
```
Mình sẽ giải thích qua 1 chút về nội dung trong Dockerfile, tại sao lại viết như vậy.

Đầu tiên, mình viết `FROM ruby:2.5.1` tức là chọn version của ruby. Khi chạy docker build, nó sẽ pull 1 image của ruby:2.5.1 từ [Dockerhub](https://hub.docker.com/) (có thể hiểu nó là Github cho Docker Images) về (image này có thể hiểu như là 1 "máy ảo" linux, được cài sẵn ruby version 2.5.1 và đóng gói lại. Thực ra thì nó không phải là "máy ảo", nhưng ở đây mình sẽ gọi thế cho mọi người dễ hình dung). Sau khi pull image đó về, chúng ta cần cài thêm package `build-essential` và `nodejs` để phục vụ cho việc code Rails.

Ý tưởng của chúng ta là:

**Mount toàn bộ source code của project vào bên trong cái "máy ảo" linux kia. sau đó chạy `rails server` bên trong "máy ảo" đó, và map cổng 3000 của "máy ảo" đó với cổng 3000 của "máy thật" bên ngoài của mình.**

Tuy nhiên, lúc này, trong "máy ảo" đó, nếu chạy lệnh `ls` để list ra các thư mục, nó sẽ có dạng:

```
root@bd46feb61e91:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
Nếu copy nguyên source code của project vào đây thì sẽ khó quản lý, nên chúng ta sẽ đặt source code ở 1 thư mục riêng, đó là lý do có lệnh `mkdir -p $APP_DIR` rôif copy code `COPY . $APP_DIR`, sau đó chạy bundle bằng lệnh `RUN bundle install` để download các gems vào bên trong "máy ảo". Do đã set `WORKDIR $APP_DIR` ở bên trên nên chúng ta không cần chạy lệnh `cd` vào trong thư mục `/app` trước khi chạy lệnh bundle nữa.

Và cuối cùng, chúng ta chỉ cần chạy lệnh `rails server -b 0.0.0.0 -p 3000` để khởi động rails server bên TRONG "máy ảo" thôi. =]]

Bây giờ chúng ta sẽ build image của project, đặt tên nó là `docker-demo` bằng lệnh:

```
docker build -t docker-demo .
```
Và sau đó run container, map cổng 3000 từ bên trong "máy ảo" ra bên ngoài "máy thật" của mình bằng option `-p 3000:3000`, để có thể vào được từ browser ở "máy thật"

```
docker run -p 3000:3000 -it docker-demo
```
Và bây giờ, mở chrome lên, vào trang: [http://localhost:3000](http://localhost:3000), chúng ta sẽ thấy được trang chủ của rails-docker-demo.

Như vậy ta đã xong bước 1, đóng gói source code + môi trường code ruby vào 1 image.

Vấn đề được nêu lên ở đầu mới được giải quyết 1 phần. Vì hiện tại, chúng ta vẫn đang dùng db là mysql ở trên "máy thật". Trong phần tiếp theo, chúng ta sẽ tìm cách để tách khỏi sự phụ thuộc vào db này.

[PullRequest#1](https://github.com/ttuan/rails-docker-demo/pull/1/files)

# Kết luận

Trên đây mình đã trình bày vấn đề của việc setup môi trường theo cách thông thường, và giải pháp đóng gói môi trường code Rails với Docker. Mọi vấn đề đều được giải quyết từ những thứ đơn giản nhất. Nếu có vấn đề, hãy tìm giải pháp, nếu giải pháp lại sinh ra 1 vấn đề khác, hãy tìm 1 giải pháp tốt hơn. Quan trọng nhất là chúng ta nhìn ra được vấn đều của mình.

Phần 2 mình sẽ trình bày về việc đóng gói DB environment, cách sử dụng docker-compose để làm "mượt" flow hơn.

Cám ơn các bạn đã đọc bài viết ;)

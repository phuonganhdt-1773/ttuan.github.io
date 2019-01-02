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

## Tips and Tricks
### 1. Một số commands hữu ích
* [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)

List ra các containers đang chạy. Nếu bạn muốn list ra tất cả các cotnainers, hãy thêm flag `-a` / `-all` và `-q` / `--quite` nếu muốn chỉ list ra các ids của containers thôi.

* [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)

Phần lớn các images của chúng ta được tạo trên các base image trên DockerHub. DockerHub chứa rất nhiều images được build sẵn mà chúng ta có thể `pull` về (giống như pull code từ Github về vậy :v)

* [docker build](https://docs.docker.com/engine/reference/commandline/build/)

Command này có lẽ là command được dùng nhiều nhất, cho phép build 1 docker images từ Dockerfile và 1 "context" (bao gồm các files, được đặt ở PATH hoặc URL). Như bên trên, khi chúng ta build bằng lệnh `docker build -t docker-demo .` thì `.` ở đây chính là context, `-t` flag là để đánh label cho image đó (kiểu đặt tên cho image)

* [docker run](https://docs.docker.com/engine/reference/run/)

Command để chạy docker container từ 1 image. Các tham số hay được dùng như `-it` 

* [docker logs](https://docs.docker.com/engine/reference/commandline/logs/)

Hiển thị logs của 1 container, ta có thể sử dụng flag `-f` hoặc `--follow` để xem log output. Ví dụ: `docker logs --follow container_id`

* [docker volume ls](https://docs.docker.com/engine/reference/commandline/volume_ls/)

List ra các volumes được dùng để lưu trữ data, sinh ra và sử dụng bởi Docker containers.

* [docker rm](https://docs.docker.com/engine/reference/commandline/rm/)

Xoá 1 hoặc nhiều containers: `docker rm container_id`

* [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)

Xoá 1 hoặc nhiều images: `docker rmi image_id`

* [docker stop](https://docs.docker.com/engine/reference/commandline/stop/)

Stop 1 hoặc nhiều containers: `docker stop container_id`. Nếu muốn stop tất cả các containers đang chạy: `docker stop $(docker ps -a -q)`. Hoặc có thể dùng lệnh `kill`: `docker kill container_id`.

* Clean up all docker images and containers

```
# Kill all running containers
docker kill $(docker ps -q)

# Delete all stopped container
docker rm $(docker ps -a -q)

# Delete all images 
docker rmi $(docker images -q)

# Clean up all resources (images, containers, volumes, networks), that are dangling (not associated with any containers)
docker system prune
```

Bạn có thể tham khảo thêm về [Dockerfile best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#workdir) - Tổng hợp 1 số tricks để làm Dockerfile thêm gọn gàng, sạch sẽ ;) 

### 2. Q&A
Trong quá trình tìm hiểu về docker, mình có rất nhiều câu hỏi. Nên mình nghĩ nên có 1 mục để nói về phần Q&A, biết đâu cũng có nhiều người có chung câu hỏi như mình :v 

**Q**: Khi lần đầu build image bằng Dockerfile, mình thấy thời gian chạy khá lâu, nhưng những lần build sau lại thấy nhanh hơn (mặc dù vẫn có thay đổi trong Dockerfile nhưng thời gian build lại ngắn hơn rất nhiều). Vậy Docker "cache" lại các bước build đó như thế nào?

**A**: Câu trả lời là nhờ vào docker layers. Còn lý do tại sao docker layers có thể giúp chúng ta, nó hoạt động ntn thì mình xin dành phần này cho 1 bài viết khác =]]

**Q**: Trong khi dung docker, mình thấy có khá nhiều lệnh giống nhau: RUN vs CMD, ADD với COPY, hay docker run và docker exec, vậy chúng khác nhau ntn?

**A**:

* RUN vs CMD: Lệnh RUN sẽ được thực thi trong quá trình build image. Nó tạo ra 1 layer mới và chồng lên trên image hiện tại. (Ví dụ Dockerfile của chúng ta có dòng: FROM ubuntu; RUN apt-get update. Thì khi chạy lệnh RUN, docker tạo ra 1 layer mới, thực thi lệnh apt-get update, layer này sẽ nằm bên trên layer được tạo ra bởi lệnh FROM ubuntu). Không giống với RUN, CMD dùng để set command mặc định. Command này sẽ được thực thi bởi container nếu chúng ta khởi động container mà không add custom command vào. Nếu trong Dockerfile có nhiều CMD mà khi chạy container, ta vẫn đặt custom command thì command cuối cùng trong Dockerfile sẽ không được thực thi. Bạn có thể tham khảo thêm ở [đây](http://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/).
* ADD vs COPY: 2 lệnh này đều cho phép bạn copy file từ local vào bên trong container. Tuy nhiên, COPY thì chỉ đơn giản lày copy files :v còn ADD cho phép copy và giản nén 1 số định dạng file.
* docker run vs docker exec: Lệnh `docker run` thường được sử dụng cho 1 container mới, tức là trong trường hợp bạn chưa có container và muốn tạo 1 container mới: Ví dụ: `docker run ubuntu bash` sẽ build mới container ubuntu, và vào đó chạy ct bash. Còn `docker exec` được sử dụng trong trường hợp bạn đã chạy container rồi, và muốn thao tác trong container đang chạy đó. Ví dụ bạn đã chạy container ubuntu rồi thì có thể thao tác trong container đó bằng `docker exec ubuntu bash`.

**Q**: Khi chạy docker images, mình thấy trong list đó có nhiều image dạng repo/tag: "\<none>\<none>", vậy những image này là thế nào?

**A**: Khi chúng ta gọi lệnh `docker pull ubuntu` chẳng hạn, thực ra không phải chúng ta pull 1 phát đươc luôn, mà ta sẽ phải pull từng layer của image ubuntu về. Đó là vì 1 image được tạo nên bởi nhiều layer, tuy nhiên chỉ có image cha trên cùng là được đánh tag, ví dụ: ubuntu:lastest. còn những layer còn lại (hay còn được gọi là image trung gian), sẽ được đánh dấu là <none>. Giả sử từ image ubuntu có mã hash "abc123123" vừa được pull trên kia, chúng ta dùng nó để build 1 image B khác dạng (FROM ubuntu; COPY . /tmp) chẳng hạn. Nhưng sau 1 tháng, ubuntu có update, chúng ta update ubuntu image thì image mới sẽ có mã hash khác đi (vd: "cdef123"). Tuy nhiên image B vẫn đang dùng bản ubuntu cũ (mã hash "abc123123"). Nếu chúng ta update bản ubuntu mới cho image B này, thì layer( hay image trung gian) được tạo ra từ dòng `FROM ubuntu` sẽ bị dangling vì nó đang là bản cũ -> 1 image <none> nữa lại sinh ra. Image dạng này thường được sinh ra do lệnh `docker build` hoặc `docker pull`. Để dọn dẹp nó thì bạn có thể tham khảo 1 vài câu lệnh ở phần Tips and tricks bên trên :v 


## Kết luận

Trên đây mình đã trình bày vấn đề của việc setup môi trường theo cách thông thường, và giải pháp đóng gói môi trường code Rails với Docker. Mọi vấn đề đều được giải quyết từ những thứ đơn giản nhất. Nếu có vấn đề, hãy tìm giải pháp, nếu giải pháp lại sinh ra 1 vấn đề khác, hãy tìm 1 giải pháp tốt hơn. Quan trọng nhất là chúng ta nhìn ra được vấn đều của mình.

Phần 2 mình sẽ trình bày về việc đóng gói DB environment, cách sử dụng docker-compose để làm "mượt" flow hơn.

Cám ơn các bạn đã đọc bài viết ;)

## References
* [Top 10 Docker CLI commands you can't live without](https://medium.com/the-code-review/top-10-docker-commands-you-cant-live-without-54fb6377f481)
* [What are docker none-none image](https://www.projectatomic.io/blog/2015/07/what-are-docker-none-none-images/)

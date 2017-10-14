---
layout: post
title: Dockerizing Rails application
tags:
- Docker
- Rails
---

![docker.png](https://d2l3jyjp24noqc.cloudfront.net/uploads/image/img/486/Dockerizing_a_Ruby_on_Rails_Application.png)

## Mở đầu
Thời gian gần đây, nghe mọi người nói nhiều tới [Docker](https://www.docker.com/) quá. Mình cũng được nghe về thằng này nhiều, cũng thỉnh thoảng bỏ ra nghịch, cài thử và trải nghiệm nhưng chỉ được lúc đó, xong rồi là mình lại vứt xó và k ngó ngàng gì tới. (Cơ bản là vì mình không thực sự thấy được tác dụng của nó cho lắm =]])

Tối hqua ngồi đọc được blog của một anh, thấy anh ý viết:
> I love `Linux`, `Docker` and `php`.

Mình thấy hơi tò mò, không hiểu sao ông ý lại thích Docker như thế :v . Nhân ngày cuối tuần rảnh rỗi, mình mới bỏ ra tìm hiểu, nhân tiện áp dụng luôn vào dự án đang làm xem nó có gì hay ho không :D

## Docker
### 1. Docker là gì?
`Docker` là một nền tảng mở, được xây dựng với mục đích là để **xây dựng, vận chuyển và chạy** các ứng dụng phân tán. Nó được mọi người biết đến như 1 giải pháp thay thế cho vấn đề ảo hoá (thay vì tạo máy ảo - tạo phần cứng ảo và cài hệ điều hành lên). `Docker` đóng gói các ứng dụng thành các Container riêng lẻ, sẽ được chạy chung trên nhân của hệ điều hành -> việc khởi động hay tắt bỏ + tốc độ sẽ nhanh hơn nhiều.

Sau khi dùng `Docker Engine` để đóng gói, bạn có thể đưa lên `DockerHub` để lưu trữ và chia sẻ.

Bạn có thể xem hướng dẫn cài đặt Docker tại [đây](https://docs.docker.com/engine/installation/)

### 2. Một số khái niệm
* Docker images: là 1 `read-only template` - đóng gói ứng dụng và các thành phần phụ thuộc của ứng dụng.
* Docker containers: Mỗi 1 docker container được tạo ra từ 1 docker image. Đây là một môi trường chưa mọi thứ mà ứng dụng có thể dùng để chạy được. Nó độc lập, k ảnh hưởng và bị ảnh hưởng bởi phần còn lại của hệ thống.
* Dockerfile: File chứa tập lệnh để Docker có thể đọc và thực hiện đóng gói một images.
* Docker server: Quản lý các containers và images.
* Docker client: Gồm các tools GUI hoặc command để ta có thể thao tác với docker.
* DockerHub: Site lưu trữ và chia sẻ images Docker.

## Dockerizing a Rails Application
OK. Bây giờ chúng ta sẽ tìm cách để đóng gói 1 rails application lại nhé ;)

### 1. Settings Docker
Đầu tiên, từ Terminal, bạn hãy gõ:

```sh
cd path/to/your_project
touch Dockerfile
```
Bạn vừa tạo 1 `Dockerfile` trong thư mục của dự án, bây giờ hãy mở file và điền nội dung như dưới đây nhé:

```
# Choose ruby version, you can choose another version
FROM ruby:2.3.1
MAINTAINER tuantv.uet@gmail.com

# Language (Optional)
ENV LANG C.UTF-8

# Configure the main working directory.
ENV APP_DIR /app
RUN mkdir -p $APP_DIR
WORKDIR $APP_DIR

# Install apt based dependencies required to run Rails as well as RubyGems.
RUN apt-get update && apt-get install -y \
  build-essential curl nodejs gnupg imagemagick
RUN apt-get clean

# Copy the Gemfile as well as the Gemfile.lock and install the RubyGems.
COPY Gemfile /cache/Gemfile
COPY Gemfile.lock /cache/Gemfile.lock
RUN echo "gem: --no-rdoc --no-ri" > ~/.gemrc
RUN cd /cache && bundle install -j4

# Copy the main application.
COPY . $APP_DIR
RUN chmod +x $APP_DIR/bin/*

# Expose port 3000 to the Docker host, so we can access it from the outside.
# EXPOSE 3000

CMD ["rails server -b 0.0.0.0 -p 3000"]
```
Nhìn vào file trên có lẽ bạn sẽ không thấy khó hiểu lắm :v . Mình xin giải thích qua như sau:

* `FROM` dùng để chỉ định image mình sẽ sử dụng.
* `ENV` set biến môi trường, `WORKDIR` để chỉ định thư mục làm việc cho các câu lệnh tiếp theo.
* `RUN` để chạy 1 lệnh của OS.

Đến đây thì bạn đã có thể tạo ra 1 container từ Dockerfile bên trên.

```
docker build -t demo .
docker run -itP demo
```
Nếu bạn config port trong file Dockerfile thì giờ bạn đã có thể truy cập bình thường từ browser. Tuy nhiên, bạn còn cần phải có `db` để có thể sử dụng các thao tác khác với ứng dụng.

Tất nhiên, bạn có thể tạo 1 container khác để chứa db mà bạn muốn thao tác. Nhưng điều này khá là bất tiện, vì bạn sẽ phải thao tác với nhiều container, gõ đi gõ lại nhiều lệnh. Đó là lý do Docker Compose ra đời.

### 2. Docker Compose
Docker Compose là một công cụ hỗ trợ định nghĩa và chạy ứng dụng nhiều container. Các container bạn dùng sẽ được định nghĩa trong 1 file YAML tên dạng `docker-compose.yml`.

Bạn có thể tham khảo cách cài đặt tại [đây](https://docs.docker.com/compose/install/).

Bây giờ là lúc chúng ta sẽ config Docker Compose.

```
# .docker-compose.yml
version: "3"
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    command: rails server -b 0.0.0.0
    volumes:
      - .:/app
      - bundle:/bundle
    ports:
      - 3000:3000
    # Connect with container db
    links:
      - db
    env_file: .env
    # Puts 2 lines to debug :D
    stdin_open: true
    tty: true
  db:
    image: mysql:5.6
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_bin --innodb-file-format=Barracuda --innodb-file-per-table=true --innodb-large-prefix=true
    volumes:
      - db-data:/var/lib/mysql
    ports:
      - 3306:3306
    environment:
      - TZ=Japan
      - MYSQL_ALLOW_EMPTY_PASSWORD=yes
volumes:
  db-data:
    driver: local
  bundle:
    driver: local
```
Trong đoạn trên, mình có sử dụng file `.env` lưu biến môi trường để có thể sử dụng trong code -> kết nối với container `db`.

```
# .env example
DATABASE_HOSTNAME=db
DATABASE_USERNAME=root
DATABASE_PASSWORD=""
```

```
# database.yml in rails code
default: &default
  adapter: mysql2
  encoding: utf8
  host: <%= ENV["DATABASE_HOSTNAME"] %>
  username: <%= ENV["DATABASE_USERNAME"] %>
  password: <%= ENV["DATABASE_PASSWORD"] %>
```

Setting done. Giờ bạn hãy chạy lệnh `docker-compose build` để build 2 container `app` và `db`. Sau khi xong việc, hãy chạy lệnh để tạo db_development bằng cách:

```
docker-compose run app rake db:create
docker-compose run app rake db:migrate
```

Trong trường hợp của mình, mình đã có sẵn db trên máy local, bây giờ mình muốn import db đó vào trong container `db` để dùng luôn. Sau khi gg thì mình có tìm được trick khá hay như sau:

```
# Backup
docker exec CONTAINER /usr/bin/mysqldump -u root --password=root DATABASE > backup.sql

# Restore
cat backup.sql | docker exec -i CONTAINER /usr/bin/mysql -u root --password=root DATABASE

```
Bây giờ bạn đã có 2 container, 1 container chứa code và 1 container chứa db. Chỉ cần bạn chạy lệnh:

```
docker-compose up
```
là có thể bắt đầu trải nghiệm qua browser được rồi. Hãy vào thử `localhost:3000` để test thành quả nhé.

## TIP, Tricks
---- Trick này tham khảo từ bài viết: http://bit.ly/2giHNjh ---

1. Nên chạy `bundle` hoặc `rails g model` tại thư mục chính của dự án thay vì chạy trong container để tránh các vấn đề về khả nưng truy cập file được tạo.
2. Khi rebuild thường sẽ gặp lỗi k thể start lại được rails server. Lý do có thể là do file temp tạo với quyền root của container trước đó nên ta k access được. Hãy remove nó đi.
```
sudo chmod -R 777 tmp log
rm -rf tmp log
```
3. Để debug, ta dùng lệnh:
```
docker attach containter_name/container_id
```
4. Nếu console có log tương tự:
```
Started GET "/assets/home-fcec5b5a277ac7c20cc9f45a209a3bcd.js?body=1" for 10.0.2.2 at 2015-04-02 15:48:31 +0000
Cannot render console from 10.0.2.2! Allowed networks: 127.0.0.1, ::1, 127.0.0.0/127.255.255.255
```
bạn cần thêm vào config:
```
config.web_console.whitelisted_ips = ENV["WHITELISTED_IPS"]
```
và thêm ip 10.0.2.2 vào biến env.

---- Commands hay dùng ----
1. `docker images`: Liệt kê các images hiện tại.
2. `docker ps`: Các container đang chạy.
3. `docker ps -a`: Tất cả các container

## Kết luận
Bài viết trên mình thấy khá là cơ bản, chỉ là tổng hợp các bước làm + 1 vài thủ thuật mình "chôm" được từ 1 số nguồn trên mạng =))))

Hy vọng bạn có thể dùng nó để dockerizing Rails application của mình.

---
layout: post
title: Deploy Rails app with Capistrano, Nginx and Puma
tags:
- Rails
---

Đây chỉ là note về cách deploy 1 Rails app, step by step, để lần sau mình làm đỡ quên =)) 

Có rất nhiều vấn đề phát sinh khi bạn deploy 1 application, có thể đến từ code bạn viết, hoặc từ hệ điều hành bạn cài, cũng có thể đến từ 1 bug của gem bạn sử dụng :v Notes dưới đây chỉ nêu ra 1 số bước cơ bản để bạn có thể nhìn thấy được TODO list cần làm khi deploy. 

Sau khi làm xong các bước này, thử deploy thì cứ lỗi đâu đọc fix đấy thôi =)) 

### Pre-requirements
* Server Ubuntu 
* You can ssh to this server
* Basic knowledge of unix

# Config in code

1. Add config in Gemfile

```ruby
group :staging, :production do
  gem "capistrano", "3.9.1"
  gem "capistrano-bundler"
  gem "capistrano-rails", "~> 1.2"
  gem "capistrano-rvm"
  gem "capistrano3-puma"
end
```

And run `bundle install`

2. Generate capistrano files
If you are config for `staging` environment, just run `cap install STAGES=staging`

https://gist.github.com/ttuan/035247d0d6282be3f2af0bebd1229ff6

Ensure you write `.env.sample` file, with format: `export DATABASE_USERNAME=xxx` (với `export` ở đằng trước)

3. Copy thư mục `deploy_bin`

[https://github.com/framgia/rails5_skeleton/tree/master/deploy_bin](https://github.com/framgia/rails5_skeleton/tree/master/deploy_bin]


Push code to `test-deploy` branch :D Để mình có thể deploy branch này để test trước, sau khi test xong sẽ merge nó vào develop.


# Config server

## 1. Create user deploy
W: Bạn hoàn toàn có thể deploy bằng một user khác, không nhất thiết phải tạo user `deploy`. Tuy nhiên, mình nên follow theo nguyên tắc đầu tiên trong thiết kế hướng đối tượng: Single Responsibility Principle. Tạo 1 user `deploy`, để nó có thể:

> Do one thing, and do it well

H: Just gg :v

```bash
# Create user deploy
sudo useradd --create-home -s /bin/bash deploy
sudo adduser deploy sudo
sudo passwd deploy

# Login to user deploy
sudo su - deploy
```

Add user deploy to group sudo :v 
Sau khi đã tạo user deploy, bạn dùng lệnh `sudo su - deploy` để login vào user `deploy`. **Các bước tiếp theo sẽ được thực hiện từ console của user deploy**.

## 2. Add ssh key of user deploy to Github project
W: Tại sao phải add ssh key lên mục deploy key của Github repo? T hoàn toàn có thể add ssh key của user deploy này vào mục ssh key của tài khoản cái nhân, thì trên server mình vẫn pull code về được bình thường. Nhưng chúng ta k nên làm thế, vì 2 lý do: 

* Trên server có thể pull mọi private repos của mình về :v (vì nó được gắn ssh key vào `account` chứ không phải deploy key của 1 `repo`) 
* Khi bạn chuyển dự án, đồng nghĩa với việc bạn bị remove ra khỏi Github dự án, thế là server sẽ không kéo được code về nữa :v 

H:

```bash
# Generate ssh key
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
```

Sau khi đã gengerate ssh key, bạn cần [add public key của mình vào deploy key trên repo Github](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys)

Sau khi add xong, bạn hãy thử fetch project của mình về **thư mục gốc**:

```bash
cd 
git clone git@github.com/user_name/private_repo.git
```

## 3. Install RVM, Ruby, MySQL
W: (yaoming) Tất nhiên, Rails project cần rvm và Ruby là đúng rồi. :v 

H: Ở đây mình không viết cụ thể từng bước, bạn có thể follow theo guide ở [Install Ruby On Rails on Ubuntu 16.04 Xenial Xerus \| GoRails](https://gorails.com/setup/ubuntu/16.04)

Install `rvm`, `ruby`, `mysql`, còn Rails thì khỏi :v vì lát mình sẽ bundle ở trong project rồi.

## 4. Install Nginx
```bash
sudo apt-get install nginx
sudo service nginx start
```

Simple nginx config

```
upstream puma{
  server unix:/usr/local/rails_apps/repo_name/current/tmp/sockets/puma.sock;
}

server {
  listen 80;
  listen [::]:80 ipv6only=on;
  server_name subdomain.domain.com;
  root /usr/local/rails_apps/repo_name/current/public;
  client_max_body_size 4G;
  location / {
    try_files $uri @webapp;
  }
  location @webapp {
    proxy_redirect     off;
    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_set_header  X-Forwarded-Ssl on; # Optional
    proxy_set_header  X-Forwarded-Port $server_port;
    proxy_set_header  X-Forwarded-Host $host;
    proxy_read_timeout 10000;
    proxy_pass http://puma;

  }

  #passenger_enabled on;
  #custom error page
  error_page 404 /404.html;
  location = /404.html {
      root /usr/local/rails_apps/repo_name/current/public;
  }
  error_page 500 /500.html;
  location = /500.html {
      root /usr/local/rails_apps/repo_name/current/public;
  }
  error_page 422 /422.html;
  location = /422.html {
      root /usr/local/rails_apps/repo_name/current/public;
  }
}
```

## Deploy
1. Create folder to store source code

```bash
sudo mkdir /usr/local/rails_apps
sudo chown deploy:deploy /usr/local/rails_apps
```

2. Fetch code in branch `test-deploy` (first time)

3. Add Environment and deploy script

```bash
cd
cp -r repo_name/deploy_bin ~/
cp repo_name/.env.example ~/.env
# After you edit .env file
source ~/.env
```

Add deploy_bin folder to file `~/.profile` to run command deploy everywhere :v 

4. Deploy with command: `localdeploy branch test-deploy`

### Other notes
+ Rails assets compile force *.jpeg to .jpg: https://blazarblogs.wordpress.com/2016/04/06/rails-force-to-precompile-jpeg-to-jpg/ So ensure you use jpg image in your code. 
+ An issue of puma: Puma can not start after I deploy. I need to deploy 2 times to start app server. https://github.com/puma/puma/issues/1957 . Follow comment in this issue to fix that :D (remove 1 line in `shared/puma.rb` file)

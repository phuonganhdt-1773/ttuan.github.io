---
layout: post
title: How to use a private repo as a dependency with Yarn or NPM
tags:
- nodejs
- npm
---



![](https://dotlayer.com/wp-content/uploads/2018/11/dotlayer.com-how-to-use-a-private-github-repo-as-npm-dependency-in-package-json-1500x750.jpg)
## I. Mở đầu
Gần đây mình có join vào 1 dự án. Code cũ của dự án này dùng Nodejs. Một điều đặc biệt là dự án được chia nhỏ ra thành khá nhiều repo. Mỗi repo đảm nhận 1 việc.

+ `project-name-api`: Backend API code - Using Nodejs
+ `project-name-fe`: Frontend code - Using React
+ `project-name-model`: Database models
+ `project-name-api-client`: Repo được gen bằng swagger-codegen, là js client của repo `project-name-api`
+ `project-name-description`: support việc tạo description cho các project liên quan

Và còn nhiều repo khác nữa.

Trong các file `package.json` của các repo sẽ được include dependency dưới dạng khá `lạ`. Ví dụ: `"project-name-client": "git+https://token@github.com/username/project-name-client.git#version"`. Do thấy lạ nên mình có google thử và tìm được bài viết ở này - [How to Use a Private Github Repo as a Dependency with Yarn & NPM \| Dotlayer](https://dotlayer.com/how-to-use-a-private-github-repo-as-a-dependency-with-yarn-npm/). Bài viết hướng dẫn cách import 1 repo private vào 1 project khác, sử dụng yarn/ npm. Chúng ta hãy thử cùng nhau tìm hiểu nhé :D

## II. How to
### 1. Using a public repo as a dependency
Theo syntax thông thường, để install 1 package trực tiếp từ Github, ta chỉ cần sử dụng cú pháp:

```bash
npm install git+ssh://git@github.com:/]
npm install git+ssh://git@github.com:/[#semver:^x.x]
npm install git+https://git@github.com//
npm install git://github.com//
npm install github:/[#]
```

Chúng ta cũng có thể chạy trực tiếp dựa vào commanline.

Để sử dụng github repo như 1 dependency, ta có thể import trực tiếp vào trong `package.json` file bằng syntax:

```json
"dependencies": {
  "bar": "git://github.com/foo/bar.git"
}
```

Đối với public repo, syntax trên  chạy ngon với tất cả `git` based version control system như GitLab, BitBucket, ..

### 2. Using a private repo as a dependency
Nếu 1 repo dependency của chúng ta là private repo và chúng ta muốn import nó vào trong `package.json` thì sao?

Syntax và idea đều giống với public version, tuy nhiên, chúng ta sẽ phải thêm phần authentication. Bạn có thể follow theo 2 cách: Dùng HTTPS hoặc SSH

#### a. Dùng HTTPS
Ý tưởng rất đơn giản, ta sẽ dùng 1 user với quyền access vào repo, generate 1 access token cho user này, và sử dụng qua basic authen khi gọi HTTPS.

![dotlayer.com-how-to-use-a-private-github-repo-as-a-dependency-with-yarn-npm.png](https://dotlayer.com/wp-content/uploads/2018/12/dotlayer.com-how-to-use-a-private-github-repo-as-a-dependency-with-yarn-npm.png)

Bạn vào mục **Settings > Developer Settings** trên Github. Sau đó chọn **Personal access tokens** và **Generate new token**. Chọn scopes cho token bạn sẽ dùng. Để hạn chế risk thì bạn nên chỉ cấp cho token này scope `read`.

![dotlayer.com-how-to-use-a-private-github-repo-as-a-dependency-with-yarn-npm-1.png](https://dotlayer.com/wp-content/uploads/2018/12/dotlayer.com-how-to-use-a-private-github-repo-as-a-dependency-with-yarn-npm-1.png)

Sau khi đã lấy được token, bạn sửa lại file `package.json` về dạng:

```json
"dependencies": {
  "bar": "git+https://[INSERT PERSONAL TOKEN HERE]:x-oauth-basic@github.com/foo/bar.git"
}
```

Bây giờ thì bạn có thể chạy lệnh `npm install`, ta đã có thể lấy source từ private repo.

Nhược điểm của cách này đó là chúng ta sẽ để token ở trong commit. Tuy nhiên, thường thì các repo này của chúng ta sẽ để private => It's should be OK :v

Cách này thường được dùng trong trường hợp bạn chạy NPM install với Docker container hoặc trong môi trường không sử dụng được SSH keys.

#### b. Sử dụng SSH
Cách 2 này khá giống cách 1, tuy nhiên sử dụng SSH cho authentication. Trong trường hợp này, URL sẽ không bao gồm token.

```json
"dependencies": {
  "bar": "git+https://github.com/foo/bar.git"
}
```

Khi dùng cách này ta cần chắc chắn lại là tài khoản của ssh key có quyền truy cập vào repo. SSH key được gen bằng cách vào **Settings > SSH and GPG keys**. Bạn có thể follow theo [guide này](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

Sử dụng cách 2 này sẽ an toàn, bảo mật và được suggest nên dùng hơn là cách HTTPS.

## III. Summary
Bài viết trên chia sẻ 2 cách để bạn import private repository vào trong `package.json`, sau đó dùng npm/yarn để install package.

HTTPS hay SSH, mỗi cách đều có ưu, nhược điểm riêng. Hãy thận trọng khi sử dụng trong từng trường hợp.

Hy vọng bài viết giúp ích được cho bạn ;) Thank you for reading :D


Referency: [How to Use a Private Github Repo as a Dependency with Yarn & NPM \| Dotlayer](https://dotlayer.com/how-to-use-a-private-github-repo-as-a-dependency-with-yarn-npm/)
